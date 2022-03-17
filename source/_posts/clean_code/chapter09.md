---
layout: post
title: "Clean Code 9장 - 단위 테스트"
categories: Clean_Code
tags: [java, dev]
toc: true
---


- TDD 법칙 3가지
- 깨끗한 테스트 코드 유지하기
- 깨끗한 테스트 코드
- 테스트 당 assert 하나
- F.I.R.S.T
- 결론



### TDD 법칙 3가지

1. 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다.
2. 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다.
3. 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.

- 위 3가지 규칙에 따르면 테스트 코드와 실제 코드가 함께 나올뿐더러 테스트 코드가 실제 코드보다 불과 몇 초 전에 나온다.
- 이렇게 일하면 사실상 전부 테스트하는 테스트 케이스가 나온다.
- 실제 코드와 맞먹을 정도로 방대한 테스트 코드는 심각한 관리 문제를 유발하기도 한다.



### 깨끗한 테스트 코드 유지하기

- 문제는 실제 코드가 진화하면 테스트 코드도 변해야 한다는 데 있다.
- 그런데 테스트 코드가 지저분할수록 변경하기 어려워진다.
- 테스트 코드가 복잡할수록 실제 코드를 짜는 시간보다 테스트 케이스를 추가하는 시간이 더 걸리기 십상이다.
- **테스트 코드는 실제 코드 못지 않게 중요하다**
- 테스트 코드는 사고와 설계와 주의가 필요하다.
- 실제 코드 못지 않게 깨끗하게 써야 한다.



***테스트는 유연성, 유지보수성, 재사용성을 제공한다.***

- 테스트 케이스가 없다면 모든 변경이 잠정적인 버그다.
- 아키텍처가 아무리 유연하더라도, 설계를 아무리 잘 나눴더라도, 테스트 케이스가 없으면 개발자는 변경을 주저한다.
- 버그가 숨어들까 두렵기 때문이다.
- 테스트가 있다면 공포는 사라지고, 테스트 커버리지가 높을수록 공포는 줄어든다.
- 실제 코드를 점검하는 자동화된 단위 테스트 슈트는 설계와 아키텍처를 최대한 깨끗하게 보존하는 열쇠다.
- 테스트 코드가 지저분할수록 실제 코드도 지저분해진다.



### 깨끗한 테스트 코드

- **가독성, 가독성, 가독성**
- 교재의 개선한 테스트 코드(한글 버전)



```java
public void 페이지_계층을_XML형태로_가져오는지_테스트한다() {
  makePages("PageOne", "PageOne.ChildOne", "PageTwo");
  
  submitRequest("root", "type:pages");
  
  assertResponseIsXML();
  assertResponseContains(
    "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
  );
}

public void 심볼릭_링크들이_XML_페이지_계층에_없는지_테스트한다() {
  WikiPage page = makePage("PageOne");
  makePages("PageOne.ChildOne", "PageTwo");
  
  addLinkTo(page, "PageTwo", "SymPage");
  
  submitRequest("root", "type:pages");
  
  assertResponseIsXML();
  assertResponseContains(
    "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
  );
  assertResponseDoesNotContain("SymPage");
}

public void XML형태로_데이터를_가져오는지_테스트한다() throws Exception {
  makePageWithContent("TestPageOne", "test page");
  
  submitRequest("TestPageOne", "type:data");
  
  assertResponseIsXML();
  assertResponseContains("test page", "<Test");
}
```

- 테스트는 명확히 세 부분으로 나눠진다.
  - 1. 테스트 자료를 만든다.
    2. 테스트 자료를 조작하며,
    3. 조작한 결과가 올바른지 확인한다.
- 테스트 코드는 본론에 돌입해 진짜 필요한 자료 유형과 함수만 사용한다.
- 그러므로 코드를 읽는 사람은 온갖 잡다하고 세세한 코드에 주눅들고 헷갈릴 필요 없이 코드가 수행하는 기능을 재빨리 이해한다.
- [Build-Operate-Check 패턴](https://medium.com/swlh/usual-production-patterns-applied-to-integration-tests-50a941f0b04a)이 위와 같은 테스트 구조에 적합하다. (링크는 아래 그림에 대한 출처)
  - Build: 테스트 시나리오를 준비하는 단계 (주로 DB에 데이터를 넣는 단계이다.)
  - Operate: 테스트하고자 하는 객체/API를 사용하는 메서드를 실행하는 단계
  - Check: 실행된 메서드가 예상한 효과를 시스템에 가져다 주었는지 확인하는 단계

![Image for post](https://miro.medium.com/max/1260/1*YslDrfj6TUWlQvaUoC3xXQ.jpeg)



***도메인에 특화된 테스트 언어***

- 흔히 쓰는 시스템 조작 API를 사용하는 대신 **API 위에다 함수와 유틸리티를 구현한 후 그 함수와 유틸리티를 사용**하므로 테스트 코드를 짜기도 읽기도 쉬워진다. (도메인에 가까운 이름을 사용한 함수와 유틸리티로 테스트를 작성하라는 의미같다.)
- 이렇게 구현한 함수와 유틸리티는 테스트 코드에서 사용하는 특수 API가 된다.
- 즉, 테스트를 구현하는 당사자와 나중에 테스트를 읽어볼 독자를 도와주는 테스트 언어이다.



***이중 표준***

- 테스트 API 코드에 적용하는 표준은 실제 코드에 적용하는 표준과 확실히 다르다.
- 단순하고, 간결하고, 표현력이 풍부해야 하지만, 실제 코드만큼 효율적일 필요는 없다.
- 테스트 환경은 제한적일 가능성이 낮다.
- 실제 환경에서는 절대로 안 되지만 테스트 환경에서는 전혀 문제없는 방식이 있다.
- 대개 메모리나 CPU나 효율과 관련 있는 경우다. (**코드의 깨끗함과는 철저히 무관하다.**)
- (이 내용으로 볼 때, 테스트 코드에서는 Stream 사용이 좀더 자유로울 수 있겠다.)



### 테스트 당 assert 하나?

- JUnit으로 테스트 코드를 짤 때는 함수마다 assert 문을 단 하나만 사용해야 한다? (훌륭하지만 때로는 여러 개도 필요하다.)
  - 물론 코드를 이해하기 쉽고 빠르다.
  - 하지만.. 위에서 작성하고 온 코드에 보면 공통적으로 "출력이 XML"이다라는 assert 문과 "특정 문자열을 포함한다"는 assert 문을 하나로 병합하는 방식이 불합리해 보인다.
  - **given-when-then**  (TDD를 공부하면서도 봤던 방식이다.)



```java
public void 페이지_계층을_XML형태로_가져오는지_테스트한다() throws Exception {
  givenPages("PageOne", "PageOne.ChildOne", "PageTwo");
  
  whenRequestIsIssued("root", "type:pages");
  
  thenResponseShouldBeXML();
}

public void 페이지_계층이_옳은_태그들을_가져오는지_테스트한다() {
  givenPages("PageOne", "PageOne.ChildOne", "PageTwo");	// 중복
  
  whenRequestIsIssued("root", "type:pages");	// 중복
  
  thenResponseShouldContain(
    "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
  );
}
```

- 테스트 코드를 읽기 쉬워졌지만, 테스트를 분리하면 중복되는 코드가 많아진다.
- Template Method 패턴을 사용해 중복을 제거할 수 있다.
  - given/when 부분을 부모 클래스에 두고 then 부분을 자식 클래스에 두면된다.
  - 아니면 완전히 독자적인 테스트 클래스를 만들어 @Before 함수에 given/when 부분을 넣고 @Test  함수에 then 부분을 넣어도 된다.
  - 하지만 모두 배보다 배꼽이 더 크다. (**비용이 크다.**)
  - 결국 저 위에 있는 코드가 더 좋아보인다.
- ''단일 assert 문'이라는 규칙이 훌륭한 지침이긴 해도, 때로는 주저 없이 함수 하나에 여러 assert 문을 넣어야 할 수도 있다.
- 최대한 assert 문 개수를 줄이는 방법이 좋겠다.



***테스트 당 개념 하나***

- (테스트 함수마다 한 개념만 테스트하라)
- 이것저것 잡다한 개념을 연속으로 테스트하는 긴 함수는 피한다.



### F.I.R.S.T (깨끗한 테스트의 5가지 규칙)

- **Fast**: 테스트는 빨리 돌아야 한다.
- **Independent**: 각 테스트는 서로 의존하면 안 된다.
  - 실행 순서가 뒤바뀌어도 전혀 상관 없어야 한다.
  - (실제로 하나의 테스트 코드 자바 파일을 IDE에서 함수를 지정하지 않고 테스트하기 버튼을 누르면 모든 테스트 함수를 실행하는 것을 볼 수 있다.)
- **Repeatable**: 테스트는 어떤 환경에서도 반복 가능해야 한다.
  - 테스트가 돌아가지 않는 환경이 있다면, 테스트가 실패한 이유를 둘러댈 변명이 생긴다. (네트워크 연결이라도!)
- **Self-Validating**: 테스트는 bool 값으로 결과를 내야 한다. 성공 아니면 실패다.
  - 테스트가 스스로 성공과 실패를 가늠하지 않는다면 판단은 주관적이 되며 지루한 수작업 평가가 필요하게 된다.
- **Timely**: 테스트는 적시에 작성해야 한다.
  - 단위 테스트는 테스트하려는 실제 코드를 구현하기 직전에 구현한다.
  - 실제 코드를 구현한 다음에는 테스트가 어렵다는 사실을 발견할지도 모른다. (실제로 그랬다.. ㅠㅠ)



### 결론

- 테스트 코드는 실제 코드만큼이나 프로젝트 건강에 중요하다.
- 테스트 코드는 실제 코드의 유연성, 유지보수성, 재사용성을 보존하고 강화하기 때문이다.
- **테스트 코드는 지속적으로 깨끗하게 관리하자**
- 표현력을 높이고 간결하게 정리하자.
- 테스트 API를 구현해 도메인 특화 언어(DSL)를 만들자.