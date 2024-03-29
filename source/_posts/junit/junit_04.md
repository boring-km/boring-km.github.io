---
layout: post
title: "4. JUnit 단언 깊게 파기"
date: 2021-09-30
categories: JUnit
tags: [dev, test, kotlin, java]
toc: true
---


- https://github.com/boring-km/JunitPractice
- 테스트 코드를 잘 조직하고 구조화할 수 있는 JUnit 기능 위주..

## 4.1 AAA로 테스트 일관성 유지
- 테스트 코드를 가시적으로 **준비, 실행, 단언(Arrange, Act, Assert)** 부분으로 조직 -> triple-A(AAA)
- given, when, then이랑 비슷하다고 보면 될 것 같음.

### Arrange, Act, Assert
- Arrange: 테스트 코드를 실행하기 전에 시스템이 적절한 상태에 있는지 확인
- Act: 테스트 코드 실행
- Assert: 실행한 코드가 기대한 대로 동작하는 지 확인

> 경우에 따라서 After 과정이 있음: 테스트 코드에서 자원을 할당하는 로직이 있었다면 clean up 되었는지 확인

## 4.2 동작 테스트 vs 메서드 테스트
- 개별 메서드를 테스트한다는 생각보다 클래스의 종합적인 동작에 집중해야 한다.

## 4.3 테스트와 프로덕션 코드의 관계
- 당연히 테스트 코드가 프로덕션 코드에 의존하는 관계이지, 그 반대는 말이 안 됨.
- 더 많은 단위 테스트를 작성할수록 설계를 변경했을 시 테스트 작성이 훨씬 용이해지는 경우가 늘어난다.
- 테스트 친화적인 설계를 채택할수록 편해지고, 설계 자체도 더 좋아진다.

### 4.3.1 테스트와 프로덕션 코드 분리
- 테스트를 별도 디렉터리로 분리하지만 프로덕션 코드와 같은 패키지에 넣기 (현재도 그렇게 하고 있음)

### 4.3.2 내부 데이터 노출 vs 내부 동작 노출
- 비공개 코드를 호출하는 테스트는 그 자체로 구현 세부 사항과 결속하게 된다.
    - 이 경우 세부 사항이 변경되면 기술적으로 공개적인 행동(인터페이스)이 그대로라고 해도 테스트는 깨질 수 있다.
- 코드의 작은 변화가 수많은 테스트를 깨면 프로그래머는 리팩토링을 꺼리게 되고, 코드 베이스는 퇴화한다.
- 종종 테스트를 작성하려고 객체에 대해 과도하게 사적인 질문을 하게 될 수도 있다.
- **테스트를 위해 내부 데이터를 노출하는 것은 테스트와 프로덕션 코드 사이에 과도한 결합을 초래한다.**
- 복잡한 private 메서드를 테스트하고 싶어질 수 있다.
    - **내부 행위를 테스트하려는 충동이 든다면 설계에 문제가 있는 것이다!**
    - **이 문제는 대부분 SRP를 지키기 않아서 그렇다!**
    - **가장 좋은 해결책은 흥미로운 private 메서드를 추출하여 다른 클래스로 이동하는 것이다.**

## 4.4 집중적인 단일 목적 테스트의 가치
- 작은 단위 테스트들을 주석으로 분리해서 하나의 테스트로 만들 수도 있다.
- 하지만 JUnit이 제공하는 테스트 고립의 이점을 잃게 된다.
- **테스트 분리의 장점**
    - 단언이 실패했을 때 실패한 테스트 이름이 표시되기 때문에 어느 동작에 문제가 있는지 빠르게 파악 가능
    - 실패한 테스트를 해독하는 데 필요한 시간 줄일 수 있음
    - 모든 케이스가 실행되었음을 보장할 수 있음

## 4.5 문서로서의 테스트
- 단위 테스트는 클래스에 대한 지속적이고 믿을 수 있는 문서 역할을 해야 한다.

### 4.5.1 일관성 있는 이름으로 테스트 문서화
- 테스트하려는 맥락을 제안하기 보다는 어떤 맥락에서 일련의 행동을 호출했을 때 어떤 결과가 나오는지를 명시해라
- 테스트 메서드의 이름을 상세하게 쓰되 의미가 명확하도록
- ex) 어떤 동작을 하면 어떤 결과가 나온다. | 어떤 결과는 어떤 조건에서 발생한다.
- 행위 주도 개발(BDD): 주어진 조건에서 어떤 일을 하면 어떤 결과가 나온다.
    - given-when-then: 조금 길어서 given 부분을 제거해서 작성하기도 함

### 4.5.2 테스트를 의미 있게 만들기
- 다른 사람들이 테스트가 어떤 일을 파악하기 어려워 한다면...
    - 테스트 이름 개선
    - 지역 변수 이름 개선
    - 의미 있는 상수 도입
    - 햄크레스트 단언 사용하기
    - 커다란 테스트를 작게 나누어 집중적인 테스트 만들기
    - 테스트 군더더기들을 도우미 메서드와 @Before 메서드로 이동하기

## 4.6 @Before와 @After
- @Before 초기화 코드가 늘어나면 @Before 메서드를 여러개로 분리해도 되지만, 실행순서는 보장되지 않는다.
- @After는 테스트가 실패해도 동작한다.
    - 테스트 후에 발생하는 부산물들을 정리해준다.
    - 예를 들어 DB와의 연결을 종료

### 4.6.1 BeforeClass와 AfterClass Annotation
- 매우 드물게 테스트 클래스 수준의 초기화인 @BeforeClass가 사용된다.
- 그 반대의 경우인 @AfterClass도 있지만 역시 잘 사용하지 않음

## 4.7 녹색이 좋다: 테스트를 의미 있게 유지
- 테스트를 빠르고 많이 실행할 수 있게 하라
- 녹색으로 항상 코드에 오류가 없도록 하자
- 빠른 피드백을 얻을 수 있는 단위 테스트에 집중하자
- 실패하는 테스트를 빨리 못 고친다면 차라리 @Ignore Annotation 달자

## 4.8 마무리
- JUnit이 할 수 있는게 의미 있으니 정신 차리고 잘 써라~
