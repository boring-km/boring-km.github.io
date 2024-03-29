---
layout: post
title: "3. JUnit 단언 깊게 파기"
date: 2021-08-30
categories: JUnit
tags: [dev, test, kotlin, java]
toc: true
---

- https://github.com/boring-km/JunitPractice

## 3.1 JUnit Assert
- Assert(단언): 테스트에 넣을 수 있는 정적 메서드 호출이다.
- 어떤 조건이 참인지 검증하는 방법이며, 참이 아니면 테스트는 그 자리에서 멈추고 실패를 보고한다.

### assertThat
- 명확한 값을 비교한다.
- hamcrest Assert
- assertThat(실제 표현식[actual], 검증하고자 하는 값[matcher])
- JUnit이 제공하는 핵심 hamcrest 매처를 사용하려면 코드에 정적 임포트를 추가해야 한다.

```java
import static org.hamcrest.CoreMatchers.*;
import java.io.*;
import java.util.*;
```

- 일반적인 assert문 보다 hamcrest assert가 실패할 때에 오류 메시지에서 더 많은 정보를 확인할 수 있다.

### 중요한 hamcrest matcher 살펴보기
- 때로는 매처 표현의 가독성을 높이기 위해 is(), not()을 사용하기도 한다.
- null이 아닌 값을 자주 검사하는 것은 설계 문제이거나 지나치게 걱정하는 것이다. (대부분 불필요하다.)

### 부동소수점 수 비교
- 단순 equalTo()로는 실패한다.
- closeTo()가 있다.

### Assert 설명
- 모든 JUnit Assert의 형식에는 message라는 선택적 첫 번째 인자가 있다.
- message 인자는 Assert의 근거를 설명해준다.

## 3.2 예외를 기대하는 3가지 방법

### 1) 단순한 방식: Annotation 사용
- @Test Annotation에 expected 값에 Exception 값을 매핑
- Exception이 발생하면 테스트 통과

### 2) 옛 방식: try/catch와 fail
- 예외가 발생하지 않으면 org.junit.Assert.fail() 메서드를 호출하여 강제로 실패

### 3) 새로운 방식: ExpectedException 규칙
- 자동으로 테스트 집합에 종단 관심사(cross-cutting concern)을 부착할 수 있다. (like AOP)

### 예외 무시
- 검증된 예외를 무시하기 위해 메서드에서 throws 처리해버리기
- 근데 코틀린은 안된다... (이런 상황이면 그냥 Annotation으로 처리하는 게 나을듯)

```java
@Test
public void someMethod() throws IOException {
        ...
}
```
