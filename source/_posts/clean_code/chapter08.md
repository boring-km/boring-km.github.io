---
layout: post
title: "Clean Code 8장 - 경계"
categories: Clean_Code
tags: [java, dev]
toc: true
---


- 외부 코드 사용하기
- 경계 살피고 익히기
- Log4j 익히기
- 학습 테스트는 공짜 이상이다
- 아직도 존재하지 않는 코드 사용하기
- 깨끗한 경계



### 외부 코드 사용하기

- 패키지 제공자나 프레임워크 제공자는 적용성을 최대한 넓히려 애를 쓰고,
- 사용자는 자신의 요구에 집중하는 인터페이스를 바란다.
- 이 시스템 경계에서 문제가 생길 소지가 많다.



#### java.util.Map

- Map이 제공하는 기능성과 유연성은 확실히 유용하지만 그만큼 위험도 크다.

- **위험**

  - Map을 만들어 여기저기 넘긴다고 할 때, Map 사용자는 누구나 clear()를 통해 Map의 내용을 지울 권한이 있다.
  - Map에 특정 객체 유형만 저장하기로 결정했어도, Map은 객체 유형을 제한하지 않기 때문에 마음만 먹으면 사용자는 어떤 객체 유형도 추가할 수 있다.

  

```java
// Case1: Sensor라는 객체를 담는 Map
Map sensors = new HashMap();
Sensor s = (Sensor)sensors.get(sensorId);
```

- Map이 반환하는 Object를 올바른 유형으로 변환할 책임은 Map을 사용하는 클라이언트에 있다.
- 하지만 의도가 분명하게 드러나지 않고 깨끗하지가 않다.



```java
// Case2: Generics를 사용한 Map
Map<String, Sensor> sensors = new HashMap<Sensor>();
Sensor s = sensors.get(sensorId);
```

- Generics를 사용하여 코드 가독성이 높아졌지만, Map<String, Sensor>가 사용자에게 필요하지 않은 기능까지 제공한다.
- 프로그램에서 Map<String, Sensor> 인스턴스를 여기저기 넘긴다면, Map 인터페이스가 변할 경우에 수정할 코드가 상당히 많아진다. (현재도 많이 바뀌는 지 모르겠지만, 단순히 Map만 두고 하는 얘기는 아닌듯 하다.)



```java
// Case3: 더 깔끔해진 Map!
public class Sensors {
  private Map sensors = new HashMap();
  
  public Sensor getById(String id) {
    return (Sensor) sensors.get(id);
  }
}
```

- Sensors 사용자는 Generics가 사용되었는지 여부에 신경 쓸 필요가 없다.
- Generics의 사용 여부는 Sensors 안에서 결정한다.
- 경계 인터페이스인 Map을 **Sensors 안으로** 숨긴다.
  - Map 인터페이스가 변하더라도 Sensors 이외의 코드에 영향이 미치지 않도록 할 수 있다.
- Sensors 클래스는 프로그램에 필요한 인터페이스만 제공한다.
  - 그래서 **코드는 쉽지만 오용하기는 어렵다.**
  - Sensors 클래스를 사용하는 프로그램에게 설계 규칙과 비즈니스 규칙을 따르도록 강제할 수 있다.
- *Map 클래스를 사용할 때마다 캡슐화하라는 얘기가 아니다.*
- Map을 여기저기 넘기지 말라는 말이다.

- **Map 인스턴스를 공개 API의 인수로 넘기거나 반환값으로 사용하지 않도록 하자**



### 경계 살피고 익히기

- 외부 패키지 테스트가 우리의 책임은 아니지만, 우리 자신을 위해 우리가 사용할 코드를 테스트하는 편이 바람직하다.
- 외부 코드는 익히기 어렵고 통합하기도 어렵다. (MSA 구조에서 이것이 얼마나 어려울지 상상도 하기 힘들다.)
- **학습 테스트** : 곧바로 우리쪽 코드를 작성해 외부 코드를 호출하는 대신 먼저 간단한 테스트 케이스를 작성해 외부 코드를 익히면 어떨까? (**결국 TDD**)
  - 프로그램에서 사용하려는 방식대로 외부 API를 호출한다.
  - 통제된 환경에서 API를 제대로 이해하는지를 확인하는 셈이다.



### log4j 익히기

- 밥 아저씨의 아파치의 log4j 연습 예시

```java
public class LogTest {
  private Logger logger;
  
  @Before
  public void initialize() {
    logger = Logger.getLogger("logger");
    logger.removeAllAppenders();
    Logger.getRootLogger().removeAllAppenders();
  }
  
  @Test
  public void basicLogger() {
    BasicConfigurator.configure();
    logger.info("basicLogger");
  }
  
  @Test
  public void addAppenderWithStream() {
    logger.addAppender(new ConsoleAppender(
    new PatternLayout("%p %t %m%n"),
    ConsoleAppender.SYSTEM_OUT));
    logger.info("addAppenderWithStream");
  }
  
  @Test
  public void addAppenderWithoutStream() {
    logger.addAppender(new ConsoleAppender(
    new PatternLayout("%p %t %m%n")));
    logger.info("addAppenderWithoutStream");
  }
}
```

- 콘솔 로거를 초기화하는 방법을 익힌 후 이 사용법들을 이용해 독자적인 로거 클래스로 캡슐화하면 나머지 프로그램은 log4j의 경계 인터페이스를 몰라도 된다. (현재의 slf4j는 위의 log4j에 비하면 훨씬 단순하게 사용하고 있는듯하다.)



### 학습 테스트는 공짜 이상이다.

- 학습 테스트에 드는 비용은 없다. (어쨌든 API를 배워야 하니까..? 오..)
- 필요한 지식만 확보할 수 있는 손쉬운 방법이며, 이해도를 높여주는 정확한 실험이다.
- 학습 테스트는 패키지가 예상대로 도는지 검증한다.
- 패키지는 패키지 작성자에 의해 계속해서 변경이 일어날 수 있고, 새 버전이 나올 때마다 새로운 위험이 생긴다. (많이 위험해보인다.)
- 새 버전이 우리 코드와 호환되지 않으면 **학습 테스트가** 이를 곧바로 밝혀낸다.
- 학습 테스트를 이용한 학습이 필요하든 그렇지 않든, 실제 코드와 동일한 방식으로 인터페이스를 사용하는 테스트 케이스가 필요하다.
- 경계 테스트가 있다면 패키지의 새 버전으로 이전하기 쉬울 것이고, 없다면 **낡은 버전을 필요 이상으로 오랫동안 사용하려는 유혹에** 빠지기 쉽다.



### 아직 존재하지 않는 코드를 사용하기

- 경계와 관련해 또 다른 유형은 아는 코드와 모르는 코드를 분리하는 경계다.
- 우리가 바라는 인터페이스를 구현하면 우리가 인터페이스를 전적으로 통제한다는 장점이 생긴다.
- 또한 코드 가독성도 높아지고 코드 의도도 분명해진다.
- (교재에서는 아직 구현되지 않은 API를 사용하는 것처럼 하기 위한 장치로 사용했다.)
  - Adapter 패턴으로 API 사용을 캡슐화해 API가 바뀔 때 수정할 코드를 한 곳으로 모았다. 



### 깨끗한 경계

- 경계에서는 변경이 대표적으로 많이 벌어진다.
- 통제하지 못하는 코드를 사용할 때는 너무 많은 투자를 하거나 향후 변경 비용이 지나치게 커지지 않도록 각별히 주의해야 한다.
- **경계에 위치하는 코드는 깔끔히 분리한다.**
  - 또한 기대치를 정의하는 테스트 케이스도 작성한다.
  - 이쪽 코드에서 외부 패키지를 세세하게 알아야 할 필요가 없다.
  - 통제가 불가능한 외부 패키지에 의존하는 대신 통제가 가능한 우리 코드에 의존하는 편이 훨씬 좋다.
  - 자칫하면 외부 코드에 휘둘린다.
- 외부 패키지를 호출하는 코드를 가능한 줄여 경계를 관리하자.
- Map에서 봤듯이 새로운 클래스로 경계를 감싸거나 Adapter 패턴을 사용해 우리가 원하는 인터페이스를 패키지가 제공하는 인터페이스로 변환하자. (번역이 잘못 된게 아닌가 싶은데, 하고 싶은 의도는 결국 외부 인터페이스에 휘둘리지 않고 우리가 원하는 인터페이스로 사용할 수 있도록 Adapter 패턴을 사용하자는 의미이다.)

