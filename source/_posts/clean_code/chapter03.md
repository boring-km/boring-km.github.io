---
layout: post
title: "Clean Code 장 - 함수"
categories: Clean_Code
tags: [java, dev]
toc: true
---

- 함수를 만드는 규칙
- 한 가지만 해라!
- 함수 당 추상화 수준은 하나로
- Switch 문
- 서술적인 이름을 사용하라
- 함수 인수
- 부수 효과를 일으키지 마라
- 명령과 조회를 분리하라
- 오류 코드보다 예외를 사용하라
- 반복하지 마라
- 구조적 프로그래밍
- 그래서 함수는 어떻게 작성하는지
- 결론



### 작게 만들어라!

- 함수를 만드는 첫째 규칙은 '작게!'다
- 둘째 규칙은 '더 작게!'다.
- If/else/while 문 등에 들어가는 블록은 한 줄이어야 한다.



### 한 가지만 해라!

- **함수는 한 가지를 해야 한다. 그 한가지를 잘 해야 한다. 그 한 가지만을 해야 한다.**
- 지정된 함수 이름 아래에서 추상화 수준이 하나인 단계만 수행한다면 그 함수는 한 가지 작업만 한다.
- 단순히 다른 표현이 아니라 의미 있는 이름으로 다른 함수를 추출할 수 있다면 그 함수는 여러 작업을 하는 셈이다.



### 함수 당 추상화 수준은 하나로!

- 함수가 확실히 '한 가지' 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일해야 한다. (이게 어려운 것 아닌가..?)
- 위에서 아래로 코드 읽기: ***내려가기*** 규칙
  - 코드는 위에서 아래로 이야기처럼 읽혀야 좋다.
  - 위에서 아래로 TO 문단을 읽어내려 가듯이 코드를 구현하면 추상화 수준을 일관되게 유지하기가 쉬워진다.



### Switch 문

- 작게 만들기가 어렵다. (If/else가 여러번 이어지는 구문도 포함)
- 본질적으로 switch 문은 N가지를 처리한다.

```java
public Money calculatePay(Employee e)
  throws InvalidEmployeeType {
  switch (e.type) {
    case COMMISSIONED:
      return calculateCommissionedPay(e);
    case HOURLY:
      return calculateHourlyPay(e);
    case SALARIED:
      return calculateSalariedPay(e);
    default:
      throw new InvalidEmployeeType(e.type);
  }
}
```

- 위 함수의 문제
  1. 함수가 길다. 새 직원 유형을 추가하면 더 길어진다.
  2. '한 가지' 작업만 수행하지 않는다.
  3. SRP(Single Responsibility Principle)를 위반한다. 코드를 변경할 이유가 여럿이기 때문이다.
  4. OCP를 위반한다. 새 직원 유형을 추가할 때마다 코드를 변경하기 때문이다.



### 서술적인 이름을 사용하라!

- 길고 서술적인 이름이 짧고 어려운 이름보다 좋다.
- 길고 서술적인 이름이 길고 서술적인 주석보다 좋다.
- 함수 이름을 정할 때는 여러 단어가 쉽게 읽히는 명명법을 사용한다. 그리고 여러 단어를 사용해 함수 기능을 잘 표현하는 이름을 선택한다.
- 서술적인 이름을 사용하면 개발자 머릿속에서도 설계가 뚜렷해지므로 코드를 개선하기 쉬워진다.
- 이름을 붙일 때는 일관성이 있어야 한다.
- 모듈 내에서 함수 이름은 같은 문구, 명사, 동사를 사용한다.



### 함수 인수

- 이상적인 인수 개수는 0개 (적을수록 좋다. 1개까지는 양호함)

- 인수는 개념을 이해하기 어렵게 만든다.

- 함수에 인수 1개를 넘기는 이유 (*많이 쓰는 단항 형식*)

  - 인수에 질문을 던지는 경우

  - 인수를 뭔가로 변환해 결과를 반환하는 경우 (변환 함수)

    - 인수에 출력 인수를 사용하면 혼란을 일으킨다.

    ```java
    boolean fileExists(String fileName) {}	// 변환 함수
    passwordAttemptFailedNtimes(int attempts);	// 이벤트 함수, 입력 인수만 있다.
    ```

- 플래그 인수

  - 추하다.
  - 함수로 부울 값을 넘기는 관례는 함수가 한꺼번에 여러 가지를 처리한다고 대놓고 공표하는 셈이다.

- 이항 함수/삼항 함수

  - 이해하기가 어렵다.
  - 프로그램을 짜다보면 불가피한 경우가 생기기도 하지만 그만큼 위험이 따른다는 사실을 이해하고 가능하면 단항함수로 바꾸도록 애써야 한다.

- 인수 객체
  - 인수가 2~3개 필요하다면 일부를 독자적인 클래스 변수로 선언할 가능성을 짚어본다.
- 인수 목록
  - 인수 개수가 가변적인 함수도 필요하다.
- 동사와 키워드
  - 함수의 의도나 인수의 순서와 의도를 제대로 표현하려면 결국 좋은 함수 이름이 필수다.
  - 단항 함수는 함수와 인수가 동사/명사 쌍을 이뤄야 한다.
  - 함수 이름에 키워드를 추가하는 형식
    - assertEquals보다 assertExpectedEqualsActual(expected, actual)이 더 좋다. 인수 순서를 기억할 필요가 없어지니까



### 부수 효과를 일으키지 마라!

- 부수 효과는 거짓말이다.

  - 함수에서 한 가지를 하겠다고 해놓고선 남몰래 다른 짓을 하니까
  - 때로는 예상치 못하게 클래스 변수를 수정한다. (교활하고 해롭다)
  - 때로는 함수로 넘어온 인수나 시스템 전역 변수를 수정한다. (교활하고 해롭다)

  - 출력 인수

    - 일반적으로 인수는 함수 입력으로 해석한다.
    -  함수 선언부를 찾아보는 행위는 코드를 보다가 주춤하는 행위와 동급이다. 인지적으로 거슬린다는 뜻이므로 피해야 한다.
    -  객체 지향 언어에서는 출력 인수를 사용할 필요가 거의 없다.

```java
// 3-6 UserValidator.java 부수 효과 예시
public class UserValidator {
  private Cryptographer cryptographer;
  
  public boolean checkPassword(String userName, String password) {
    User user = UserGateway.findByName(userName);
    if (user != User.NULL) {
        String codedPhrase = user.getPhraseEncodedByPassword();
        String phrase = cryptographer.decrypt(codedPhrase, password);
        if ("Valid Password".equals(phrase)) {
            Session.initialize();
            return true;
        }
    }
    return false;
  }
}
```



### 명령과 조회를 분리하라!

- 함수는 뭔가를 수행하거나 뭔가에 답하거나 둘 중 하나만 해야 한다.



### 오류 코드보다 예외를 사용하라!

- 명령 함수에서 오류 코드를 반환하는 방식은 명령/조회 분리 규칙을 미묘하게 위반한다.
- 오류 코드를 반환하면 호출자는 오류 코드를 곧바로 처리해야 한다는 문제에 부딪힌다.
- 반면 오류 코드 대신 예외를 사용하면 오류 처리 코드가 원래 코드에서 분리되므로 코드가 깔끔해진다.

- Try/Catch 블록 뽑아내기
  - 원래 추하다.
  - 코드 구조에 혼란을 일으킨다.
  - 정상 동작과 오류 처리 동작을 뒤섞는다.

```java
// 오류 코드 대신 예외를 사용한 코드
try {
  deletePage(page);
  registry.deleteReference(page.name);
  configKeys.deleteKey(page.name.makeKey());
}
catch (Exception e) {
  logger.log(e.getMessage());
}

// try/catch 블록을 별도 함수로 뽑아내는 편이 좋다.
public void delete(Page page) {	// 오류를 처리하는 함수는 오류만 처리해야 마땅하다.
  try {
    deletePageAndAllReferences(page);
  }
  catch (Exception e) {
    logError(e);
  }
}
private void deletePageAndAllReferences(Page page) throws Exception {
  deletePage(page);
  registry.deleteReference(page.name);
  configKeys.deleteKey(page.name.makeKey());
}
private void logError(Exception e) {
  logger.log(e.getMessage());
}
```

- Error.java 의존성 자석
  - 오류 코드를 반환한다는 이야기는, 클래스든 열거형 변수는, 어디선가 오류 코드를 정의한다는 뜻이다.
  - Error 클래스 변경은 어렵고 번거로워 새 오류 코드를 정의하고 싶지 않기 때문에 기존 오류 코드를 재사용한다.
  - 오류 코드 대신 예외를 사용하면 새 예외는 Exception 클래스에서 파생된다.
  - 따라서 재컴파일/재배치 없이도 새 예외 클래스를 추가할 수 있다. (OCP)



### 반복하지 마라! (DRY)

- 중복은 소프트웨어에서 만악의 근원
- 많은 원칙과 기법이 중복을 없애거나 제어할 목적으로 나왔다.
- 객체 지향 프로그래밍은 코드를 부모 클래스로 몰아 중복을 없앴다.
- 구조적 프로그래밍, AOP(Aspect Oriented Programming), COP(Component Oriented Programming)



### 구조적 프로그래밍

- 모든 함수와 함수 내 모든 블록에 입구와 출구가 하나만 존재해야 한다고 말했다. (Return 하나)
- loop 안에서 break나 continue, goto 안된다.

- 함수가 아주 클 때만 상당한 이익을 제공한다.
- 그래서 함수를 작게 만든다면 return, break, continue를 여러 차례 사용해도 괜찮다. (오히려 의도를 표현하기 쉬워짐)
- goto는 큰 함수에서만 의미가 있다고 하지만 그냥 안 쓰는게..?



### 함수를 어떻게 짜죠?

- 글짓기와 비슷하다.
- 처음에는 길고 복잡하다.
  - 들여쓰기 단계도 많고 중복된 루프도 많다. 인수 목록도 아주 길다.
  - 이름은 즉흥적이고 코드는 중복된다.
  - 하지만 그 서투른 코드를 빠짐없이 테스트하는 Unit Test를 만들자
- 코드를 다듬고, 함수를 만들고, 이름을 바꾸고, 중복을 제거한다.
- 메서드를 줄이고 순서를 바꾼다. 때로는 전체 클래스를 쪼개기도 한다.
- 이와중에 코드는 항상 단위 테스트를 통과한다.



### 결론

- 모든 시스템은 특정 응용 분야 시스템을 기술할 목적으로 프로그래머가 설계한 도메인 특화 언어(Domain Specific Language, DSL)로 만들어진다.
- 함수는 그 언어에서 동사며, 클래스는 명사다.
- 대가 프로그래머는 시스템을 (구현할) 프로그램이 아니라 (풀어갈) 이야기로 여긴다.
- 프로그래밍 언어라는 수단을 통해 이야기를 풀어간다.
- 작성하는 함수가 분명하고 정확한 언어로 깔끔하게 같이 맞아떨어져야 이야기를 풀어가기가 쉬워진다는 사실을 기억하자

