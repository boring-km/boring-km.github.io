---
layout: post
title: "Clean Code 11장 - 시스템"
categories: Clean_Code
tags: [java, dev]
toc: true
---

- 도시를 세운다면?
- 시스템 제작과 시스템 사용을 분리하라
- 확장
- 자바 프록시
- 순수 자바 AOP 프레임워크
- AspectJ 관점
- 테스트 주도 시스템 아키텍처 구축
- 의사 결정을 최적화하라
- 명백한 가치가 있을 때 표준을 현명하게 사용하라
- 시스템은 도메인 특화 언어가 필요하다
- 결론



### 도시를 세운다면?

- 혼자서 온갖 세세한 상황을 직접 관리할 수는 없지
- 각 분야를 관리하는 팀이 있기 때문에 도시는 돌아간다.
- 도시의 큰 그림을 그리는 사람도 있고, 작은 사항에 집중하는 사람들도 있다.
- 적절한 추상화와 모듈화로 큰 그림을 이해하지 못할지라도 개인과 개인이 관리하는 '구성요소'는 효율적으로 돌아간다.
- **깨끗한 코드를 구현하면 낮은 추상화 수준에서 관심사를 분리하기 쉬워진다**
- 높은 추상화 수준, 즉 **시스템 수준**에서도 깨끗함을 유지하는 방법을 알아보자



### 시스템 제작과 시스템 사용을 분리하라

- **제작(construction)은 사용(use)과 아주 다르다**

> **소프트웨어 시스템은 (애플리케이션 객체를 제작하고 의존성을 서로 '연결'하는)**
>
> **준비 과정과 (그 후에 이어지는) 런타임 로직을 분리해야 한다.**

- 시작 단계는 모든 애플리케이션이 풀어야 할 **관심사**다.

- **관심사 분리는** 우리 분야에서 가장 오래되고 가장 중요한 설계 기법 중 하나이다.

  ```java
  public Service getService() {
    if (service == null) 
      service = new MyServiceImpl(...);	// 모든 상황에 적합한 기본값일까?
    return service;
  }
  ```

  - *전형적으로 관심사를 분리하지 않는 예시*
    - 초기화 지연 방식 (Kotlin에서 lateinit을 사용하는 것이 생각난다.)
    - 실제로 필요할 때까지 객체를 생성하지 않으므로 불필요한 부하가 걸리지 않는다.
    - 어떤 경우에도 null 포인터를 반환하지 않는다.
  - 하지만 getService 메서드가 MyServiceImpl과 (위에서는 생략한) 생성자 인수에 명시적으로 의존한다.
  - **런타임 로직에서 MyServiceImpl 객체를 전혀 사용하지 않더라도 의존성을 해결하지 않으면 컴파일이 안 된다.** (해당 서비스가 불필요하더라도 무조건 객체를 생성해야 하는 의존성 발생한다. 메서드 이름이라도 좀더 구체적이었으면 혹시 몰라도..)
    - 테스트 할 때도 MyServiceImpl이 무거운 객체라면 단위 테스트에서 getService 메서드를 호출하기 전에 적절한 테스트 전용 객체(Mock Object)를 service 필드에 할당해야 테스트 가능하다.
    - **또한 일반 런타임 로직에 객체 생성 로직을 섞어놓은 탓에 (service가 null인 경로와 null이 아닌 경로 등) 모든 실행 경로도 테스트해야 한다.** (팩토리 메서드 패턴을 이래서 사용하나보다.)

  - 작지만 **SRP** 위반이다.
  - 초기화 지연 기법을 한 번 정도 사용한다면 별로 심각한 문제는 아니지만, 이런 설정 기법이 수시로 사용되면 모듈성은 저조해지고 대개 중복이 심각하다.
  - **설정 논리**는 일반 실행 논리와 분리해야 모듈성이 높아진다.



#### Main 분리

- 생성과 관련한 코드는 모두 main이나 main이 호출하는 모듈로 옮기고, 나머지 시스템은 모든 객체가 생성되었고 모든 의존성이 연결되었다고 가정한다.
- main 함수에서 시스템에 필요한 객체를 생성한 후 이를 애플리케이션에 넘긴다.
- **애플리케이션은 main이나 객체가 생성되는 과정을 전혀 모른다.** (단지 모든 객체가 적절히 생성되었다고 가정한다.)



#### 팩토리

- 물론 때로는 객체가 생성되는 **시점**을 애플리케이션이 결정할 필요도 생긴다.
- Abstract Factory: 교재 그림을 보고 예상되는 코드 작성함 (코드 참조)
- 객체의 생성 시점은 애플리케이션이 결정하지만 객체를 생성하는 코드는 애플리케이션이 모르도록 구현



#### 의존성 주입(DI)

- 클래스가 의존성을 해결하려 시도하지 않는다.
- 클래스는 완전히 수동적이다.
- 대신 의존성을 주입하는 방법으로 (Object 교재에서 보았던 3가지 방법 - [링크](https://github.com/boring-km/Object-Study/blob/master/chapter09/chapter09.md))
  - setter 메서드
  - 생성자 인수
  - 혹은 둘 다
- ~ DI 컨테이너는 필요한 객체의 인스턴스를 만든 후 생성자 인수나 설정자 메서드를 사용해 의존성을 설정한다.
- 초기화 지연으로 얻는 장점을 무조건 포기할 필요는 없다. DI를 사용하더라도 때론 여전히 유용하다.
  - 대다수 DI 컨테이너는 필요할 때까지는 객체를 생성하지 않고, 대부분은 계산 지연이나 비슷한 최적화를 쓸 수 있도록 팩토리를 호출하거나 프록시를 생성하는 방법을 제공한다.
  
### 확장
- '처음부터 올바르게' 시스템을 만들 수 있다는 믿음은 미신이다.
- 소프트웨어 시스템은 '수명이 짧다'는 본질로 인해 아키텍처의 점진적인 발전이 가능하다.
- EJB1과 EJB2 예시를 통해 관심사를 적절히 분리하지 못한 경우는 책에서 확인 (p.200 ~ 202)

### 자바 프록시
- 개별 객체나 클래스에서 메서드 호출을 wrapping 하는 경우
- JDK에서는 기본으로 인터페이스만 지원하고, 클래스 프록시를 사용하려면 추가 라이브러리가 필요하다. (현재도 그런지 확인 필요!)

```java
// JDK 프록시 예제

// Bank.java - 은행 추상화

import java.util.*;

public interface Bank {
  Collection<Account> getAccounts();

  void setAccounts(Collection<Account> accounts);
}

// BankImpl.java - 추상화를 위한 POJO 구현
import java.util.*;

public class BankImpl implements Bank {
  private List<Account> accounts;

  public Collection<Account> getAccounts() {
    return accounts;
  }

  public void setAccounts(Collection<Account> accounts) {
    this.accounts = new ArrayList<Account>();
    this.accounts.addAll(accounts);
  }
}

// BankProxyHandler.java - Proxy API가 필요로 하는 InvocationHandler
import java.lang.reflect.*;
        import java.util.*;

public class BankProxyHandler implements InvocationHandler {
  private Bank bank;

  public BankProxyHandler(Bank bank) {
    this.bank = bank;
  }

  // 자바 리플렉션 사용
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    String methodName = method.getName();
    if (methodName.equals("getAccounts")) {
      bank.setAccounts((Collection<Account>) args[0]);
      return bank.getAccounts();
    } else if (methodName.equals("setAccounts")) {
      bank.setAccounts((Collection<Account>) args[0]);
      setAccountsToDatabase(bank.getAccounts());
      return null;
    } else {
      // 교재에서도 생략된 부분
    }
  }

  protected Collection<Account> getAccountsFromDatabase() { /* 구현 코드 */ }

  protected void setAccountsToDatabase(Collection<Account> accounts) { /* 구현 코드 */ }
}

// BankService.java (교재에서는 이름 따로 없음)
import java.lang.reflect.Proxy;

public class BankService {

  public void someMethod() {
    Bank bank = (Bank) Proxy.newProxyInstance(
            Bank.class.getClassLoader(),
            new Class[]{Bank.class},
            new BankProxyHandler(new BankImpl()));
  }
}
```

- 단점1: 코드의 양과 크기 -> 깨끗한 코드를 작성하기 어렵다
- 단점2: 시스템 단위로 실행 지점을 명시하는 메커니즘을 제공하지 않는다. -> (AOP에서 해결)

### 순수 자바 AOP 프레임워크
- 내부적으로는 프록시를 사용한다.
- 스프링은 비즈니스 로직을 POJO로 구현한다.
- POJO는 순수하게 도메인에 초점을 맞추기 때문에 테스트가 개념적으로 더 쉽고 간단하여 보수하고 개선하기도 쉽다.
- 결론적으로 스프링과 같은 프레임워크에서 사용자가 모르게 프록시나 바이트코드 라이브러리를 사용해 **영속성, 트랜잭션, 보안, 캐시, 장애조치**와 같은 횡단 관심사를 구현하고 있다.
- 이는 프레임워크에서 코드를 작성한다고 하지만 사실상 많은 부분에서 이미 프레임워크와 독립적인 코드가 만들어지게 되었다.
- XML이 조금 읽기 어렵지만 그런 설정 파일에 명시된 정책이 겉으로 잘 보이지 않지만 자동으로 생성되는 프록시나 관점 논리보다 단순하다는 평가에 따라 Spring과 EJB3에 기여한다.
- 개선된 코드는 교재에서 확인 (p.208~p.209)

### AspectJ 관점
- 관심사를 관점으로 분리하는 가장 강력한 도구: AspectJ 언어
- 새로 배워야하는 어려움이 있어서 AspectJ를 쉽게 사용하도록 도와주는 다양한 도구가 마련되어 있다.
- (추가) 개발해보면서 spring aop 라이브러리만 추가해도 대부분 사용이 가능한 것 같다.

### 테스트 주도 시스템 아키텍처 구축
- 애플리케이션 도메인 논리를 POJO로 작성할 수 있다면 테스트 주도 아키텍처 구축이 가능해진다.
- 아주 단순하면서도 멋지게 분리된 아키텍처로 프로젝트를 재빨리 진행한 다음, 기반 구조를 추가하면서 조금씩 확장해도 괜찮다.
- 하지만! 설계가 아무리 멋진 API라도 정말 필요하지 않으면 **과유불급**

### 의사 결정을 최적화하라
- 모듈을 나누고 관심사를 분리하면 지엽적인 관리와 결정이 가능해진다.
- **가능한 마지막 순간까지 결정을 미루는 방법**이 좋을 때도 있다.
- 요약: 관심사를 모듈로 분리한 POJO 시스템은 기민함을 제공한다. 이런 기민함 덕택에 최신 정보에 기반해 최선의 시점에 최적의 결정을 내리기가 쉬워진다. 또한 결정의 복잡성도 줄어든다.

### 명백한 가치가 있을 때 표준을 현명하게 사용하라
- 표준을 사용하면 아이디어와 컴포넌트를 재사용하기 쉽다.
- 적절한 경험을 가진 사람을 구하기 쉽다.
- 컴포넌트를 엮기 쉽다.
- **하지만** 때로는 표준을 만드는 시간이 너무 오래 걸려 업계가 기다리지 못하거나 표준을 제정한 본래 목적을 잊어버리기도 한다.

### 시스템은 도메인 특화 언어가 필요하다
- DSL: 간단한 스크립트 언어나 표준 언어로 구현한 API를 가리킨다.
- 좋은 DSL은 도메인 개념과 그 개념을 구현한 코드 사이에 존재하는 '의사소통 간극'을 줄여준다.
- 효과적으로 잘 사용한다면 추상화 수준을 코드 관용구나 디자인 패턴 이상으로 끌어올려서 개발자가 적절한 추상화 수준에서 코드 의도를 표현할 수 있다!

### 결론
- 시스템 역시 **깨끗해야** 한다.
- 깨끗하지 못한 아키텍처는 도메인 논리를 흐리며 기민성을 떨어뜨린다.
- 도메인 논리가 흐려지면 제품 품질이 떨어진다. 버그가 숨어들기 쉬워지고, 스토리를 구현하기 어려워진다.
- 기민성이 떨어지면 생산성이 낮아져 TDD가 제공하는 장점이 사라진다.
- 모든 추상화 단계에서 의도는 명확히 표현해야 한다.
- **POJO를 작성하고 관점 or 관점과 유사한 메커니즘을 사용해 각 구현 관심사를 분리해야 한다.**
- 시스템을 설계하든 개별 모듈을 설계하든, **실제로 돌아가는 가장 단순한 수단**을 사용해야 한다는 사실을 명심하자!

