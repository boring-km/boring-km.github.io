---
title: 7. 경계 조건: CORRECT 기억법
category: 자바와 JUnit을 활용한 실용주의 단위테스트
order: 7
---

- https://github.com/boring-km/JunitPractice

- Conformance(준수)
- Ordering(순서)
- Range(범위)
- Reference(참조)
- Existence(존재)
- Cardinality(기수)
- Time(시간)
- 마무리

### 7.1 [C]onformance 준수
- *값이 기대한 양식을 준수하고 있는가?*
- 예를 들어 이메일 같은 경우면 '@' 기호에 따라 양식을 준수하는지 판단한다.
- 문자열 데이터를 검증할 때는 많은 규칙이 필요하다. 다행히 대부분 단순하다.
- 하지만 구조적 데이터의 경우 테스트 케이스가 엄청 많을 수 있다.
    - 처음에 데이터가 입력될 때 테스트해서 다른 시스템으로 넘어갈 때 또 테스트하지 않도록 주의하면 되겠다.

### 7.2 [O]rdering 순서
- *값의 집합이 적절하게 정렬되거나 정렬되지 않았나?*
- 교제 예시 코드 [전체 코드 예시](https://github.com/gilbutITbook/006814/tree/master/iloveyouboss_15)

```kotlin
@Test
fun answersResultsInScoredOrder() {
  smeltInc.add(Answer(doTheyReimburseTuition, Bool.FALSE))
  pool.add(smeltInc)
  langrsoft.add(Answer(doTheyReimburseTuition, Bool.TRUE))
  pool.add(langrsoft)
  
  pool.score(soleNeed(doTheyReimburseTuition, Bool.TRUE, Weight.Important))
  val ranked: List<Profile> = pool.ranked() // 여기서 정렬
  assertThat(ranked.toArray(), equalTo(arrayOf<Profile>(langrsoft, smeltInc)))
}

```

### 7.3 [R]ange 범위
- *이성적인 최솟값과 최댓값 안에 있는가?*
- 자바 기본형으로 변수를 만들 때 대부분은 필요한 것보다 훨씬 많은 용량을 가진다.
- 기본형의 과도한 사용에 대한 코드 냄새를 **기본형 중독**이라 한다.

- 360도인 원의 이동 방향을 자바 기본형으로 저장하기보다 Bearing 클래스로 범위를 제약하는 로직을 캡슐화할 수 있다.
    - chapter07 BearingTest.kt 확인

#### 좀 더 복잡한 경우라면?
- 점 2개를 x, y라는 정수형 tuple로 유지하는 클래스가 있다.
- 두 점이 이루는 각 변은 100 이하여야 한다.
    - x, y 좌표 쌍으로 허용되는 범위는 상호 의존적이다.
- 좌표에 영향을 줄 수 있는 어떤 동작에 관하여 범위를 assert 해보자
    - 그래서 x, y 좌표 쌍의 범위를 타당하게 유지해보자
- @After를 추가해 테스트가 완료되었을 때마다 확인할 수 있다.
- [예시 코드](https://github.com/gilbutITbook/006814/blob/master/iloveyouboss_16/test/scratch/RectangleTest.java)
    - 각 변을 100으로 제약함

#### 7.3.1 불변성을 검사하는 사용자 정의 Matcher 생성
- 사용자 정의 hamcrest matcher를 구현하려면 TypeSafeMatcher 클래스를 상속해 매칭하고자 하는 타입을 지정해야 한다.
- matchesSafely()를 오버라이드 해서 제약 사항을 재정의
    - 각 변이 범위 안에 있으면 true
- 사용자 정의 matcher 클래스는 단언이 실패할 때
    - 의미 있는 메시지를 describeTo() 메서드에 기재해야 한다.
    - Matcher 인스턴스를 반환하는 정적 팩토리 메서드를 제공해야 한다.

#### 7.3.2 불변 메서드를 내장하여 범위 테스트
- 테스트할 대부분 범위는 application-domain 제약이라기보다는 자료 구조에 관한 제약에 의존하게 될 것이다.
- 희소 배열(sparse array)에 관한 [예시](https://github.com/gilbutITbook/006814/blob/master/iloveyouboss_16/src/util/SparseArray.java)
- (자세히 볼 필요까지야 있겠나...)
- checkInvariants() 메서드로 null이 아닌 배열 내 값들의 갯수가 배열의 길이와 동일하지 않으면 예외 던지기
    - 최소한 어디서 예외가 발생하는지 그나마 추적이 쉬워진다.

```java
import org.junit.*;
import scratch.*;
import static org.hamcrest.CoreMatchers.*;
import static org.junit.Assert.*;

public class SparseArrayTest {
  private SparseArray<Object> array;

  @Before
  public void create() {
    array = new SparseArray<>();
  }
  
  @ExpectToFail
  @Ignore
  @Test
  public void handlesInsertionInDescendingOrder() {
    array.put(7, "seven");
    array.checkInvariants();
    array.put(6, "six");
    array.checkInvariants();
    assertThat(array.get(6), equalTo("six"));
    assertThat(array.get(7), equalTo("seven"));
  }
}
```

> 인덱싱은 수많은 잠재적인 오류를 포함하고 있다.

### 7.4 [R]eference 참조
- *코드 자체에서 통제할 수 없는 어떤 외부 참조를 포함하고 있는가?*
- **고려해야 할 점들**
    - 범위를 넘어서는 것을 참조하고 있지 않은지
    - 외부 의존성은 무엇인지
    - 특정 상태에 있는 객체를 의존하고 있는지 여부
    - 반드시 존재해야 하는 그 외 다른 조건들
- 어떤 상태에 대해 가정할 때 **그 가정이 맞지 않으면 코드가 합리적으로 잘 동작하는지 검사**
    1. 가속 이후에 변속기를 주행으로 유지하는가?
    2. 주행 중에 주차로 바꾸는 파괴적인 요청을 무시하는가?
    3. 차량이 움직이지 않으면 주차로 변속기 변경을 허용하는가?

- [예제](https://github.com/gilbutITbook/006814/blob/master/iloveyouboss_16/test/transmission/TransmissionTest.java)

```kotlin
import org.hamcrest.CoreMatchers
import org.hamcrest.MatcherAssert.assertThat
import org.junit.Before
import org.junit.Test

class TransmissionTest {

  private lateinit var transmission: Transmission
  private lateinit var car: Car

  @Before
  fun 초기화() {
    car = Car()
    transmission = Transmission(car)
  }

  @Test
  fun 주행으로_기어를_바꾸고_35mph로_가속해도_기어는_주행기어다() {
    transmission.shift(Gear.DRIVE)
    car.accelerateTo(35)
    assertThat(transmission.gear, CoreMatchers.equalTo(Gear.DRIVE))
  }

  @Test
  fun 주행으로_기어를_바꾸고_30mph로_가속하면_기어를_주차로_바꿔도_기어는_주행기어다() {
    transmission.shift(Gear.DRIVE)
    car.accelerateTo(30)
    transmission.shift(Gear.PARK)
    assertThat(transmission.gear, CoreMatchers.equalTo(Gear.DRIVE))
  }

  @Test
  fun 기어를_주행으로_바꾸고_30mph로_가속할_때_브레이크_정지_후_기어를_주차로_바꾸면_기어는_주차기어다() {
    transmission.shift(Gear.DRIVE)
    car.accelerateTo(30)
    car.brakeToStop()
    transmission.shift(Gear.PARK)
    assertThat(transmission.gear, CoreMatchers.equalTo(Gear.PARK))
  }
}
```

### 7.5 [E]xistence 존재
- *주어진 값이 존재하는가?*
    - null, 0, 혹은 비어 있는 경우
- **우리는 프로그래머로서 보통 행복 경로를 만드는 데 무엇보다 주력한다**
    - 예상하는 데이터가 없을 때 발생하는 불행 경로는 그 다음에 생각하고는 한다.
- **호출된 메서드가 null을 반환하거나, 기대하는 파일이 없거나, 네트워크가 다운되었을 때 어떤 일이 일어나는지 확인하는 테스트를 작성해라!**

> 작성한 메서드가 홀로 설 수 있도록!

### 7.6 [C]ardinality 기수
- *정확히 충분한 값들이 있는가?*
- 울타리 기둥 오류는 한 끗 차이로 발생하는 수많은 경우 중 한 가지를 의미한다.
    - 개수를 어떻게 잘 세어 테스트할지 고민해 보고, 얼마나 많은지 확인해 보라

#### 팬케이크 가게의 작업 목록 도출
- 상위 열 개의 음식 목록을 유지해야 함
- 주문이 나올 때마다 이 상위 목록을 갱신하여 실시간으로 팬케이크 보스 아이폰 앱에 결과를 표시한다.

> 목록에 항목이 하나도 없을 때 보고서 출력
> 목록에 항목이 한 개만 있을 때 보고서 출력
> 목록에 항목이 없을 때 한 항목 추가
> 목록에 항목이 하나만 있을 때 한 항목 추가하기
> 목록에 항목이 아직 열 개 미만일 때 한 항목 추가하기
> 목록에 항목이 이미 열 개가 있을 때 한 항목 추가하기

- 만약 상위 20개의 목록으로 바뀐다면?
    - 아니면 5개? -> 같은 상수를 사용하고 있으므로 **0, 1, n**이라는 **경계 조건에만 집중하고** n은 비즈니스 요구 사항에 따라 바뀔 수 있다.

### 7.7 [T]ime 시간
- *모든 것이 순서대로 일어나는가? 정확한 시간에? 정시에?*
    - 상대적 시간(시간 순서)
    - 절대적 시간(측정된 시간)
    - 동시성 문제들
- login()이 먼저 logout()이 나중
- open() 이후에 read() 호출
- 수명이 짧은 자원에 대해 코드가 얼마나 기다릴 지 -> 타임아웃도 상대적인 시간 문제
    - 발생하지 않을 일을 기다리느라 코드가 무한 대기에 빠지지는 않았는지

> True Or False? 한 해의 모든 날은 항상 24시간?(윤초는 세지 않음)

- 정답은 상황에 따라 **다르다**
    - UTC(국제표준시)에서는 긍정이다.
    - DST(일광시간절약제)가 관찰되는 지역에서는 **False**
        - 3월 하루는 23시간이고 11월 하루는 25시간이다...
- 때가 되면 여기저기 깨진 코드가 많아진다.
- 시스템 시계에 의존하는 테스트를 작성하는 해결책도 있다.
    - 대신 테스트 코드에 통제할 수 있는 곳에서 얻어 오는 시간을 사용하도록 애플리케이션을 변경한다. (*챕터 5.4 참고*)

#### 동시성과 동기화된 접근 Context(문맥?)에 관하여...
- 동시에 같은 객체를 다수의 스레드가 접근한다면?
- 어떤 전역 혹은 인스턴스 수준의 데이터나 메서드에 동기화를 해야 할까?
- 파일 혹은 하드웨어에 외적인 접근은 어떻게 처리?
- **클라이언트에 동시성 요구 사항이 있다면 다수의 클라이언트 스레드를 보여주는 테스트를 작성할 필요가 있다.**

### 7.8 마무리
- 모든 경계를 알 필요가 있다. 테스트에서는 더욱...
- 경계 조건들은 자주 고약하고 작은 결함들을 만들어 내는 곳이다.
- **CORRECT** 약어를 통해 단위 테스트 작성 시 고려해야 하는 경계들을 기억하는데 도움을 받자.
