---
layout: post
title: "Five Lines of Code - Chapter 01"
categories: five-lines-of-code
tags: [dev, refactoring]
toc: true
---

# 1. Refactoring Refactoring

[You can also read in Notion](https://www.notion.so/1-Refactoring-Refactoring-9c026e40f2f6433eafa240fcca8d2396?pvs=4)

# 책 내용 정리

- Skill, Culture, Tool 요소가 모두 필요한 리팩터링

## 1.1 리팩터링이란 무엇인가?

### 리팩터링을 해야하는 이유

- 코드를 더 빠르게 만들기 위해
- 더 작은 코드를 만들기 위해
- 코드를 더 일반적이거나 재사용 가능하게 하기 위해
- 코드의 가독성을 높이고 유지보수를 용이하게 하기 위해

<aside>
💡 좋은 코드: 사람이 읽기 쉽고, 유지보수가 용이하며, 의도한 대로 잘 동작하는 코드

</aside>

<aside>
💡 Refactoring: 기능을 변경하지 않고 코드의 가독성과 유지보수가 쉽도록 코드를 변경하는 것

</aside>

## 1.2 Skill: 무엇을 리팩터링할 것인가?

**Code Smell: 코드가 나쁘다는 것을 암시**

**코드 스멜과 규칙은 완전히 겹칠 수 없다**

이 책에서 제시하는 ‘다섯 줄 제한’ 규칙에 명심해보자

## 1.3 Culture: 리팩터링은 언제 할까?

리팩터링은 정기적으로 수행하는 것이 효과적이고 비용이 적게 들기 때문에 가능하면 일상 업무에 통합하는 것이 좋다.

Green-Red-Refactor의 테스트 주도 개발 방식을 벗어난 6단계 절차

1. 탐색: 무엇을 만드는지 실험부터 시작해보자
2. 명세화: 무엇을 만들지 알게 되면 그것을 명세화하자
3. 구현: 코드 구현
4. 테스트: 코드가 2단계의 명세를 따르는지 확인하자
5. 리팩터링: 코드를 전달하기 전에 다음 사람이 쉽게 작업할 수 있는지 확인한다.
6. 전달: Pull Request나 특정 branch로 Push하는 방법

********************************리팩터링 5단계********************************

1. 메서드 선택
2. 메서드가 규칙을 위반했는가? (Yes → 1. 메서드 선택)
3. 규칙에 해당하는 리팩터링 패턴 찾기 (No)
4. 지시에 따르기
5. 모든 컴파일 오류 수정하기

**************************************리팩터링이 필요하지 않는 3가지 경우**************************************

1. 한 번 실행하고 삭제할 코드
2. 폐기되기 전 유지보수 모드에 있는 코드
3. 임베디드 시스템이나 게임의 고급 물리엔진과 같이 엄격한 성능 요구사항이 있는 코드

## 1.4 Tool: (안전한) 리팩터링 방법

***자동화된 테스트를 통해 안전한 브레이크를 확보하자***

이 책에서는 새로운 스킬을 배우는 것이기 때문에 천천히 가도 된다. (일단 배제하라는 의미 같음)

- 레시피처럼 상세하고 단계별로 구조화된 리팩터링 패턴
- 버전 관리(Git)
- 컴파일러

## 1.5 시작하는 데 필요한 도구 (생략)

## 1.6 핵심 예제: 2D 퍼즐 게임

https://github.com/wikibook/five-lines.git

https://github.com/wikibook/bomb-guy

## 1.7 실제 환경에서 소프트웨어에 대한 주의 사항

- 실무에서 적용하기 위한 리팩터링 규칙을 배우는 책이다.
- 코드 스멜에 대한 기본적인 이해를 쌓자
- 더 잘 알 때까지 규칙을 따르자

# 요약

- 리팩터링을 수행하려면 리팩터링 대상을 식별하는 **Skill**과 리팩터링 단계를 명시적으로 가진 **Culture**, 리팩터링을 돕는 **Tool**의 조합이 필요하다.
- 일반적으로 Code Smell은 리팩터링 대상을 설명하는 데 사용된다. 이 책에서 학습하는 동안 Code Smell을 대체할 구체적인 규칙을 제공한다.
- 3가지 추상화 수준: 매우 구체적인 이름, 예외 형태로 뉘앙스를 더하는 설명, 그것들이 나오게 된 Smell의 의도
- 자동화된 테스트와 리팩터링을 별도로 학습하면 진입 장벅을 낮출 수 있다. 여기서는 자동화된 테스트 대신 컴파일러, 버전 관리 및 수동 테스트를 사용한다.
- 리팩터링의 작업 절차는 보통 Red-Green-Refactor의 방식이나 이는 자동화된 테스트에 의존했을 때의 얘기이다.
  여기서는 새로운 코드를 만들 때 사용하는 6단계의 작업 절차로 수행해본다. (탐색 - 명세화 - 구현 - 테스트 - 리팩터링 - 전달)

## 내 생각/느낀점

- 자동화 테스트의 중요성!
- 이 책에서 어떤 리팩터링 기법/패턴/규칙들을 소개해 줄지 조금 기대가 된다.
- 무의식적으로 코드를 정리하다보면 기능까지 조금 개선하면서 리팩토링을 할 때가 있는데, 옳은 방법이었는지 아닌지 확인이 필요할 것 같다. (리팩터링의 정의만 보면 기능은 일체 바꾸지 않는 것처럼 정의하고 있음)

> *”이 책에서는 완전히 새로운 스킬을 배우기 때문”* 이라는 문구가 나오는데, 얼마나 대단하길래 새롭다는 말을 하는지 궁금해졌다.

회사 업무에 적용한다면? (신규 프로젝트 or 새로운 기능 추가 상황)

1. 탐색: 새로 개발할 기능이 구현 가능한 기술인지 검증
2. 명세화: Jira 개인 업무 리스트에 추가
3. 구현: 코드 작성
4. 테스트: 테스트 코드 작성 or 직접 테스트
5. 리팩터링: 코드 리팩터링 수행
6. 전달: 대부분 개인 프로젝트라 전달할 일이 없긴 하지만, 여럿이서 하는 프로젝트라면 현재 PR 올릴 수단이 없으니 특정 branch로 Push하고
   해당 Git Repository 담당자에게 Push한 branch의 리뷰를 요청한다.

# 억지로 규칙 적용해보기

## 이슈

- 전에 만들었던 Flutter 이미지 Crop 패키지를 개선해야 하는 상황이 있었음
- Flutter 3.10.x → 3.13.x 버전으로 올라가면서 이에 대응해야 할 필요도 있었음
- Flutter의 오픈 소스 라이브러리 공개 사이트인 pub.dev에서는 업로드한 공개 라이브러리의 점수를 평가하는 게 있는데, 여기서 점수가 일부 깎여 있어 다시 채우고자 했음
  <br/>https://github.com/boring-km/dynamic_image_crop/issues/11

### 기존 리팩터링 방식

- Red: https://github.com/boring-km/dynamic_image_crop/pull/10
- Green: https://github.com/boring-km/dynamic_image_crop/pull/12
- Refactoring: https://github.com/boring-km/dynamic_image_crop/commit/1fa211f84a7b49b37a47a0442f7dab7c369dafce

### 끼워 맞춰 보기

1. 탐색: 이슈 확인

   [Update 0.0.9 · boring-km/dynamic_image_crop@4955089](https://github.com/boring-km/dynamic_image_crop/actions/runs/6058750212/job/16441185826?pr=12)

2. 명세화: 무엇을 만들지 알게 되면 그것을 명세화하자

   https://github.com/boring-km/dynamic_image_crop/issues/13

3. 구현: 코드 구현
    - (새로 개발하는 기능이 있다면 여기서…)
4. 테스트: 코드가 2단계의 명세를 따르는지 확인하자

   expected behavior: All test passed!

   [Codecov](https://app.codecov.io/gh/boring-km/dynamic_image_crop)

   [Update 0.0.9 · boring-km/dynamic_image_crop@bf78318](https://github.com/boring-km/dynamic_image_crop/actions/runs/6058912112/job/16441485532)

5. 리팩터링: 코드를 전달하기 전에 다음 사람이 쉽게 작업할 수 있는지 확인한다.
    - (새로 개발한 기능에 대해 리팩토링 수행…)
    - 여기서는 bug fix: https://github.com/boring-km/dynamic_image_crop/pull/12/commits/bf78318656d70b36c79426f804a3f498214cf34d
6. 전달: Pull Request나 특정 branch로 Push하는 방법

   https://github.com/boring-km/dynamic_image_crop/pull/12

> 결론: **bugfix**와 **refactoring**은 다르다.


# 최근에 수행했던 다른 리팩터링

https://github.com/boring-km/Android_Simple_Counter/commit/71b082e8128b2f06ecd3bc8b6097b16d1dc9edd7

# 예제 분석 및 Re-Engineering to Compose Multiplatform

https://github.com/FiveLinesofCodeStudy/2D-Puzzle-Compose

https://github.com/FiveLinesofCodeStudy/Bomb-Guy-Compose