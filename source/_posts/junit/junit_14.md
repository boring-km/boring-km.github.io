---
layout: post
title: "14. 프로젝트에서 테스트"
categories: JUnit
tags: [dev, test, kotlin, java]
toc: true
---

- https://github.com/boring-km/JunitPractice

- 빠른 도입
- 팀과 같은 편 되기
- 지속적 통합으로 수렴
- 코드 커버리지
- 마치며


## 14.1 빠른 도입
- 테스트 없이 코드를 만드는 것은 매우 단기간에만 생산적이다.
- 이것저것 문제를 안고 개발된 마지막 순간에서 할 수 있는 것은 협상만 남게 된다.
- 단위 테스트가 팀 문화(cadence)의 습관적인 일부가 될 수 있을지 토론해보자.

## 14.2 팀과 같은 편 되기
- 개발자들이 단위 테스트에 접근하는 방식은 개인별로 매우 다르다.
- 적당한 타협을 통해 팀과 같은편이 되도록 하자

### 14.2.1 단위 테스트 표준 만들기
- 어떻게 표준을 도출할 것인가?

> 개발자들은 어던 것이 모든 사람의 시간을 많이 낭비하게 만든다고 느끼나요?
>
> 모두가 빠르게 동의할 수 있는 단순한 표준은 무엇인가요?

- 표준을 세우고, 지속할 필요가 있으며, 필요할 때마다 다시 살펴보고 수정해야 한다.
- 특히 초창기에는 더욱 그러하다.

#### 초창기에 표준화해야 하는 목록
- 코드를 체크인하기 전에 어떤 테스트를 실행해야 할지 여부
- 테스트 클래스와 메서드의 이름 짓는 방식
- 햄크레스트 혹은 전통적인 단언 사용 여부
- AAA 사용 여부
- 선호하는 Mock 도구 선택
- 체크인 테스트를 실행할 때 콘솔에 출력을 허용할지 여부
- 단위 테스트 스위트에서 느린 테스트를 분명하게 식별하고 막을 방법

### 14.2.2 리뷰로 표준 준수 높이기
- "어떻게 코드 리뷰를 할 것인가?"
- 리뷰 세션을 통해 단위 테스트 작성자가 다른 팀원에게 피드백을 요청할 수 있다.
- 책에서는 '일부 팀들이 채택하는'이라고 했지만 최근에는 많은 팀에서 **'Pull Request'**를 활용하고 있는 것 같다.

### 14.2.3 짝 프로그래밍을 이용한 리뷰
- 2명의 프로그래머가 함께 나란히 앉아서 소프트웨어를 개발하는 방법
- 최상의 리뷰는 코드를 깊이 이해한 사람에게서 나오지만, **현실적으로 많은 회사에 시간적 여유가 없습니다.**
- 결과적으로 리뷰는 바라는 것보다 더 적은 결함을 찾게 된다.
- 사후 리뷰는 심각한 문제를 고치는 데 너무 늦는다.
- 짝 프로그래밍은 두 번째 사람의 눈으로 시작부터 품질이 좋은 코드를 만들 수 있따는 희망을 줄 수 있따.
    - 더 많고 좋은 단위 테스트를 지속해야 이뤄진다.
    - 짝 프로그래밍을 하게 되면 단위 테스트의 가치는 더 높아진다.

> 짝 프로그래밍이 불편한 사람도 있으니 짝 프로그래밍을 성공적으로 수행하는 노하우를 충분히 이해해야 한다.

## 14.3 지속적 통합으로 수렴 (CI)
- CI 서버는 소스 저장소를 모니터링 한다.
- 새로운 코드가 체크인 되면 CI 서버는 소스 저장소에서 코드를 가져와 빌드를 초기화한다.
- 빌드에 문제가 있다면 CI 서버는 개발 팀에 통지한다.
- CI 서버가 어떤 가치를 제공하려면 빌드가 단위 테스트를 함께 수행해야 한다.
- CI 서버는 나쁜 코드를 용납하지 않도록 건강한 동료 압박을 지원한다.
- 개발자들은 습관적으로 스스로에게 체크인하기 전에 단위 테스트를 먼저 돌려보아 CI 빌드가 실패해 다른 팀원들의 시간을 낭비하지 않게 한다.

> CI 서버는 현대 개발 팀을 구성하는 최소 요건이다.

## 14.4 코드 커버리지
- 단위 테스트가 실행한 코드의 전체 퍼센트를 측정
- 개발 환경에 맞는 커버리지 도구를 사요해보라

### 14.4.1 커버리지는 어느 정도여야 하는가?
- 대부분의 사람은 70% 이하의 커버리지는 불충분하다고 말한다.
- 코드를 작성하고 습관적으로 단위 테스트를 작서앟는 팀들은 비교적 쉽게 70%의 커버리지를 달성한다.
    - 보통 테스트되지 않은 부분은 나쁜 의존성 때문에 그 코드가 어렵거나 테스트하기 어렵기 때문이다.
    - 정말 이상한 것은 코드 결함의 30%는 이러한 테스트 되지 않은 코드에 있다. (어려운 코드는 많은 결함을 숨기기 마련이다.)

> **제프의 코드 커버리지 이론:** 낮은 커버리지의 영역에서 나쁜 코드의 양도 증가한다.

### 14.4.2 100% 커버리지는 진짜 좋은가?
- TDD를 수행하는 개발자들은 일반적으로 정의상 90%를 초과 달성한다.
- (많은 팀이 높은 커버리지만 달성하고 가치는 별로 없는 단위 테스트를 작성하느라 시간 낭비하는 것을 보았다고 한다.)

### 14.4.3 코드 커버리지의 가치
- 테스트 작성을 완료했다고 생각할 때 커버리지 도구를 실행해라
- 아직 커버되지 않은 영역을 보라.
- 커버하지 않은 코드 영역을 염려한다면 더 많은 테스트를 작성해라
- 커버리지 도구를 주기적으로 바라보면 지속적으로 단위 테스트에 솔직해질 수 있다.
- 시간이 지나면서 커버리지 퍼센트가 높아져야 하고, 적어도 아래 방향으로 내려가면 안 된다.

## 14.5 마치며
- 단위 테스트를 프로페셔널하게 사용해서 소프트웨어의 품질을 높일 수 있다.
- 계속 주시하고 단위 테스트와 TDD에 관해 조사해라.