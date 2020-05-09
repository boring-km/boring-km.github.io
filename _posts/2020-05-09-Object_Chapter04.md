## 데이터 중심의 설계
- 객체 내부에 저장되는 데이터를 기반으로 시스템을 분할하는 방법
- 객체가 포함해야 하는 데이터에 집중함
- 객체의 종류를 저장하는 인스턴스 변수와 인스턴스의 종류에 따라 배타적으로 사용될 인스턴스 변수를 하나의 클래스 안에 함께 포함시킨 구조
- 캡슐화의 원칙에 따라 내부의 데이터를 반환하는 접근자와 데이터를 변경하는 수정자가 존재(getter, setter)

### 데이터 중심 설계의 문제점
- 높은 결합도, 낮은 응집도 => **변경에 취약**하다.
- 데이터 중심의 설계는 너무 이른 시기에 데이터에 관해 결정하도록 강요한다.
- 데이터 중심의 설계에서는 협력이라는 문맥을 고려하지 않고 **객체를 고립**시킨 채 오퍼레이션을 결정한다.
  - 객체의 구현이 **이미 결정된 상태에서** 다른 객체와의 협력 방법을 고민함
  - 객체의 **인터페이스가 구현에 노출되어** 협력이 구현 세부사항에 종속될 수 있다. => 내부 변경 시 외부에 영향을 준다.

```java
public class Movie {

  private void apply(DiscountPolicy discountPolicy) {

    Money discountAmount = Money.ZERO;

    // DiscountPolicy의 종류가 늘어날수록 case문도 계속 늘어난다.
    // 내부 DiscountPolicy에 대한 구현이 외부에 드러나 있음
    switch (discountPolicy) {
        case AMOUNT_DISCOUNT:
            discountAmount = this.discountAmount;
            break;
        case PERCENT_DISCOUNT:
            discountAmount = fee.multiply(discountPercent);
            break;
        case NONE_DISCOUNT:
            break;
    }
    fee = fee.minus(discountAmount);
  }
}
public class DiscountCondition {

  public boolean check(Screening screening) {

    LocalDateTime whenScreened = screening.getWhenScreened();
    int sequence = screening.getSequence();

    // DiscountConditionType의 종류가 많아질수록 if문도 계속 늘어난다.
    // 내부 DiscountConditionType에 대한 구현이 외부에 드러나 있음
    if (type == DiscountConditionType.PERIOD) {
        return whenScreened.getDayOfWeek().equals(startTime.getDayOfWeek()) &&
                whenScreened.isAfter(startTime) &&
                whenScreened.isBefore(endTime);
    } else {
        return this.sequence == sequence;
    }
  }
}
```
### 데이터 중심 설계에서의 캡슐화
  - 직접 객체의 내부에 접근할 수 없도록 구현은 하고 있지만 **객체 내부에 어떤 이름의 인스턴스 변수가 있는지를 인터페이스에 그대로 드러낸다.**
  - 설계에서 변하는 것이 무엇인지 고려하고 변하는 개념을 캡슐화해야 한다.


### 데이터 중심 설계에서의 높은 결합도
- 객체 내부의 구현이 객체의 인터페이스에 드러난다는 것은 클라이언트가 구현에 강하게 결합된다는 것
  - **구현은 변경될 가능성이 높다!**
- 여러 데이터 객체들을 사용하는 제어 로직이 특정 객체 안에 집중된다. (교재의 ReservationAgency 클래스)
  - 하나의 제어 객체가 다수의 데이터 객체에 강하게 결합된다.
  - 그래서 어떤 데이터 객체를 변경하더라도 제어 객체를 함께 변경할 수밖에 없다.

```java
public class ReservationAgency {
    public Reservation reserve(Screening screening, Customer customer, int audienceCount) {
        Movie movie = screening.getMovie();

        boolean discountable = false;
        for (DiscountCondition condition : movie.getDiscountConditions()) {
            if (condition.getType() == DiscountConditionType.PERIOD) {
                discountable = screening.getWhenScreened().getDayOfWeek().equals(condition.getDayOfWeek()) &&
                        condition.getStartTime().compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
                        condition.getEndTime().compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
            } else {
                discountable = condition.getSequence() == screening.getSequence();
            }
            if (discountable) {
                break;
            }
        }
        Money fee;
        if (discountable) {
            Money discountAmount = Money.ZERO;
            switch (movie.getMovieType()) {
                case AMOUNT_DISCOUNT:
                    discountAmount = movie.getDiscountAmount();
                    break;
                case PERCENT_DISCOUNT:
                    discountAmount = movie.getFee().times(movie.getDiscountPercent());
                    break;
                case NONE_DISCOUNT:
                    discountAmount = Money.ZERO;
                    break;
            }
            fee = movie.getFee().minus(discountAmount);
        } else {
            fee = movie.getFee();
        }
        return new Reservation(customer, screening, fee, audienceCount);
    }
}
```
- Reservation, Screening, Movie, DiscountCondition에 의존하고 있는 ReservationAgency

### ReservationAgency에서 낮은 응집도
- MovieType이 새로 추가되면 case문도 추가된다.
- MovieType별로 할인 계산 방식이 달라지면 case문 내부의 구현을 변경해야한다.
- DiscountCondition이 추가되면 그에 따라 if문이 증가한다.
- DiscountCondition별로 할인 여부 판단 방식이 달라지면 if문 내부 구현을 변경해야한다.
- fee를 계산하는 방식이 달라지면 할인 정책을 반영하는 부분의 구현을 변경할 수 있다.

1. 변경과 관련없는 코드들이 같은 모듈에 들어있어 문제를 발생시킬 수 있다.
2. 변경사항에 대해 하나의 모듈만 변경하는 것이 아니라 다른 여러 모듈을 함께 수정하는 추가작업이 발생하게 된다.

## 캡슐화
- 변하는 어떤 것이든 구현과 관련된 것이라면 다 감추자
- 변경될 수 있는 어떤 것이라도 캡슐화(객체 내부 인스턴스 변수의 값 뿐만 아니라 타입, 그 이름까지도)

### 캡슐화가 중요한 이유
- 불안정한 부분과 안정적인 부분을 분리해서 변경의 영향을 통제한다.
- 캡슐화를 지키면 모듈 안의 응집도는 높아지고 모듈 사이의 결합도는 낮아진다.

### 높은 응집도
- 이유: 설계를 변경하기 쉽게 만든다.
- 변경의 대상과 범위가 명확해진다.
- 단일 책임 원칙(Single Responsibility Principle, SRP)
  - 클래스는 단 **한 가지의** 변경 이유만 가져야 한다.
  - 모듈 내의 요소들이 **하나의 목적을** 위해 긴밀하게 협력함

### 낮은 결합도
- **하나의 변경에 대해 최소한의 모듈만** 변경되도록 설계
- 내부 구현을 변경했을 때 **다른 모듈에 미치는 영향이 최소가 되도록** 설계
- 변경될 가능성이 거의 없는 경우(표준 라이브러리, 성숙 단계의 프레임워크)에는 결합도가 높아도 고민하지 않아도 된다.

### 질문
- 데이터 중심 설계를 책임 주도적 설계로 변경하는 것 vs 재설계
