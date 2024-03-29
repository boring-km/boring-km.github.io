---
layout: post
title: "5. 좋은 테스트의 FIRST 속성"
date: 2021-10-05
categories: JUnit
tags: [dev, test, kotlin, java]
toc: true
---

- https://github.com/boring-km/JunitPractice

## 5.1 FIRST: 좋은 테스트 조건

> Fast: 빠른
> Isolated: 고립된
> Repeatable: 반복 가능한
> Self-validating: 스스로 검증 가능한
> Timely: 적시의

- TDD를 적용하면 좀 더 좋게 작성할 수도...!

## 5.2 Fast: 빠르게
- 테스트를 실행하는 데에 시간이 많이 걸린다면 설계가 깨끗하지 않은 것이다.

### 느린 테스트 해결?
- 느린 테스트에 대한 의존성을 줄여라
- **더 많은 로직을 커버하는 소수의 빠른 테스트는 DB 호출에 의존하는 단일 테스트보다 수월하게 실행된다.**
- **코드를 클린 객체 지향 설계 개념과 맞출수록 단위 테스트 작성도 쉬워진다.**

## 5.3 Isolated: 고립시킨다
- **좋은 단위 테스트는 검증하려는 작은 양의 코드에 집중한다.**
- 테스트 대상 코드는 DB를 읽는 다른 코드와 상호 작용할 수도 있다.
    - 그렇게 해서 데이터 의존성은 많은 문제를 만든다.
    - 단순히 외부 저장소와 상호 작용하게 되면 테스트가 가용성 혹은 접근성 이슈로 실패할 가능성이 높다.
- **좋은 단위 테스트는 다른 단위 테스트에 의존하지 않는다.**
- 어떤 순서나 시간에 관계없이 실행할 수 있어야 한다.
- 각 테스트가 작은 양의 동작에만 집중하면 테스트 코드를 집중적이고 독립적으로 유지하기 쉬워진다.
- **테스트를 고립시켜 시계처럼 동작시켜라~**

## 5.4 Repeatable: 좋은 테스트는 반복 가능해야 한다.
- 반복 가능한 테스트는 실행할 때마다 결과가 같아야 한다.
- 따라서 반복 가능한 테스트를 만들려면 직접 통제할 수 없는 외부 환경에 있는 항목들과 격리시켜야 한다.
- 하지만 불가피하게 통제할 수 없는 요소와 상호 작용해야 할 때도 있다. (mock 객체)

### 시간을 활용하는 테스트 예제
- 시간은 계속 흐르기 때문에 일정한 테스트가 불가능할 때가 있다.
- 코드가 진짜 시간을 가진 것처럼 속여보기..?

> **직접 교재에 있는 코드 작성하려니까 추가할 게 너무 많아서 링크**
> https://github.com/gilbutITbook/006814/tree/master/iloveyouboss_16-branch-persistence-3

- QuestionController 클래스는 Clock 객체의 출처는 신경 쓰지 않고 오직 현재의 Instant 객체로만 대답한다.
- DB를 동시에 변경할 수 있는 다른 개발자들과의 충돌의 피하고자 사적인 서버를 활용할 수도 있다.
- 유령 문제를 좇아 시간을 낭비하지 말고 테스트를 일관되게 고립시켜서 반복 가능하도록 하자.

## 5.5 Self-validating: 스스로 검증 가능하다
- **단위 테스트는 시간을 소모하는 것이 아닌 절약하는 방법이다.**
- 테스트 결과를 수동으로 검증하는 것은 시간 소모적인 절차고 리스크가 늘어난다.
    - 코드가 출력해 내는 거대한 로그를 보다가 중요한 신호를 놓칠 수도 있다.
- 테스트는 스스로 검증 가능할 뿐만 아니라 준비할 수도 있어야 한다.
    - 수동으로 준비 단계를 만들지 마라
    - 어떤 설정 단계든 자동화해라
    - 외부 설정이 필요하다면 고립성 위반이다.
- Infinitest: 시스템이 변경되면 이들을 식별하고 백그라운드로 잠재적으로 영향을 받는 테스트들을 실행한다.
- Jenkins, TeamCity와 같은 CI 도구를 이용해 빌드와 테스트를 수행할 수 있다.
    - 더 나아가 빌드 서버가 확신 정도를 판단하고, 프로덕션 시스템(운영 시스템)에 변경 사항을 반영한다.

## 5.6 Timely: 적시에 사용한다
- 단위 테스트로 코드를 검증하는 것을 미룰수록 치석이 끼고 충치가 늘어날 것이다.
- 심지어는 리뷰 프로세스, 심지어 충분한 테스트가 없을 때 코드를 거부하는 자동화 도구를 사용하기도 한다.
- **CI 환경에 자주 체크인하여 개발자들이 적시에 단위 테스트를 작성하는 습관을 들이도록 하고 있습니다.**
- **단위 테스트를 더 많이 할수록 테스트 대상 코드가 줄어든다.**

## 5.7 마무리
- 단위 테스트를 작성하는 것은 상당한 시간이 필요하다.
- 테스트 코드가 그에 상응하는 가치가 있다고 해도 이러한 테스트 코드 또한 유지보수 해야 한다.
- 테스트 코드를 고품질로 유지하여 이러한 유지보수 비용을 줄이자
