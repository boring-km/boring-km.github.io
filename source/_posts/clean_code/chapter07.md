---
layout: post
title: "Clean Code 7장 - 오류 처리"
categories: Clean_Code
tags: [java, dev]
toc: true
---

- 오류 코드보다 예외를 사용하라
- Try-Catch-Finally 문부터 작성하기
- 미확인 예외를 사용하기
- 예외에 의미를 제공하라
- 호출자를 고려해 예외 클래스를 정의하기
- 정상 흐름을 정의하기
- null을 반환/전달하지 말라
- 결론

### 오류 코드보다 예외를 사용하라
- 논리와 오류 처리 코드가 뒤섞이지 않게 만든다. (오류 코드에 따라 if/else로 분기처리 => try/catch)

### Try-Catch-Finally 문부터 작성하라
- 예외가 발생할 코드를 짤 때는 try-catch-finally로 시작하자
- 무슨 일이 생기든지 호출자가 기대하는 상태를 정의하기 쉬워진다.

#### 파일이 없으면 예외를 던지는 단위 테스트

```java
@Test(expected = StorageException.class)
public void retrieveSectionShouldThrowOnInvalidFileName() {
    sectionStore.retrieveSection("invalid - file");    
}
```

- 위의 테스트를 **먼저** 작성하고 테스트에 맞춰 코드를 구현한다.

```java
public List<RecordedGrip> retrieveSection(String sectionName) {
    // 실제로 구현할 때까지 비어 있는 더미를 반환한다.
    return new ArrayList<RecordedGrip>();
}
```

- 지금으로서는 테스트를 정상적으로 진행하기 어렵다.
- 테스트가 빨리 정상적으로 녹색이 들어오도록 수정하자. (TDD에서 봤던 순서)

```java
public List<RecordedGrip> retrieveSection(String sectionName) {
    try {
        FileInputStream stream = new FileInputStream(sectionName);
        stream.close();
    } catch (FileNotFoundException e) {
        throw new StorageException("retrieval error", e);
    }
    return new ArrayList<RecordedGrip>();
}
```

- 코드가 예외를 던지기 때문에 테스트가 성공할 수 있다.
- 나머지 세부 로직을 TDD를 사용해 추가한다.
- 나머지 로직은 FileInputStream을 생성하는 코드와 close 호출문 사이에 넣으며 오류나 예외가 전혀 발생하지 않는다고 가정한다.
- 먼저 강제로 예외를 일으키는 테스트 케이스를 작성한 후 테스트를 통과하게 코드를 작성하는 방법으로 시작하자

### 미확인 예외를 사용하라
- 확인된 예외는 OCP를 위반한다.
- 메서드에서 확인된 예외를 던졌는데 catch 블록이 3단계 위에 있다면 그 사이 메서드 모두가 선언부에 해당 예외를 정의해야 한다. (자바에서 throws 예외클래스명)
- 모듈과 관련된 코드가 전혀 바뀌지 않았더라도 (선언부가 바뀌었기 때문에) 모듈을 다시 빌드한 다음 배포해야 한다.
- **연쇄적인 수정으로 인해 캡슐화가 깨진다.**
- 때로는 정확한 예외를 위해 확인된 예외가 쓰일 수도 있지만, 일반적인 애플리케이션은 의존성이라는 비용이 그에 따른 이익보다 더 크다.

### 예외에 의미를 제공하라
- 예외를 던질 때는 전후 상황을 충분히 덧붙인다.
- 자바는 모든 예외에 호출 스택을 제공하지만, 실패한 코드의 의도를 파악하려면 호출 스택만으로는 부족하다.
- 오류 메시지에 정보를 담아 예외와 함께 던진다.
- 실패한 연산 이름과 실패 유형도 언급한다.
- 애플리케이션이 로깅 기능을 사용한다면 catch 블록에서 오류를 기록하도록 충분한 정보를 넘겨준다.

### 호출자를 고려해 예외 클래스를 정의하라
- 애플리케이션에서 오류를 정의할 때 가장 중요한 관심사는 **오류를 잡아내는 방법**이 되어야 한다.
- 감싸는(wrapper) 클래스를 사용해서 예외를 잡아내자

```java
public class LocalPort {
    private ACMEPort innerPort;
    
    public LocalPort(int portNumber) {
        innerPort = new ACMEPort(portNumber);
    }
    
    public void open() {
        try {
            innerPort.open();
        } catch (DeviceResponseException e) {
            throw new PortDeviceFailure(e);
        } catch (ATM1212UInlockedException e) {
            throw new PortDeviceFailure(e);
        } catch (GMXError e) {
            throw new PortDeviceFailure(e);
        }
    }
}
```  

- wrapper 클래스는 외부 API를 감싸면서 외부 라이브러리와 프로그램 사이에서 의존성이 크게 줄어든다.
- 나중에 다른 라이브러리로 갈아타도 비용이 적다.
- 테스트 하기도 쉬워진다.
- 특정 업체가 API를 설계한 방식에 발목 잡히지 않는다.

### 정상 흐름을 정의하라
- 비즈니스 논리와 오류 처리를 잘 분리한 코드가 되어가다가도, 오류 감지가 프로그램 언저리로 밀려나게 된다.

***특수 사례 패턴(Special Case Pattern)***

```java
// 식비를 비용으로 청구하지 않았다면 일일 기본 식비를 더한다.
try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getTotal();
} catch (MealExpensesNotFound e) {
    m_total += getMealPerDiem();
}

// 특수 상황을 처리할 필요가 없도록 바꾼다.
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();

public class PerDiemMealExpenses implements MealExpenses {
    public int getTotal() {
        // 기본값으로 일일 기본 식비를 반환한다.
    }
} 
```

### null을 반환하지 마라
- null을 확인하는 수많은 코드들은 여전히 있다.
- null을 반환하는 코드는 일거리를 늘릴 뿐만 아니라 호출자에게 문제를 떠넘긴다.
- 누구 하나라도 null 확인을 빼먹으면 애플리케이션이 통제 불능에 빠질지도 모른다.
- 위에서도 사용했던 **특수 사례 패턴**을 통해서 null이 나오지 않도록 빈 값, 빈 리스트 등을 반환하도록 해서 null 체크를 없애자

### null을 전달하지 마라
- 정상적인 인수로 null을 기대하는 API가 아니라면 메서드로 null을 전달하는 코드는 최대한 피한다.
- 대다수 프로그래밍 언어는 호출자가 실수로 넘기는 null을 적절히 처리하는 방법이 없다.
- 그렇다면 애초에 **null을 넘기지 못하도록 금지하는 정책**이 합리적이다.
- 인수에 null이 넘어오지 않도록 하여 부주의한 실수를 줄이자.

### 결론
- 깨끗한 코드는 읽기도 좋아야 하지만 안정성도 높아야 한다.
- 오류 처리를 프로그램 논리와 분리해 독자적인 사안으로 고려하면 튼튼하고 깨끗한 코드를 작성할 수 있다.
- 오류 처리를 프로그램 논리와 분리하면 독립적인 추론이 가능해지며 코드 유지보수성도 크게 높아진다.
