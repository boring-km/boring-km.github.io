---
layout: post
title: "Five Lines of Code - Chapter 02"
categories: five-lines-of-code
tags: [dev, refactoring]
toc: true
---

# 2. Looking under the hood of Refactoring

[You can also read in Notion](https://www.notion.so/2-Looking-under-the-hood-of-Refactoring-7701be7f88ee4cc58897f5af894fb82f?pvs=4)

## 느낀점

- 가독성 → 너무나도 당연하게 중요하다. 하지만 잘 안지켜질 때가 은근 많다.
- 유지보수성 → 변경이 일어나는 유지보수를 할 때 안전하게 하는 방법으로 “불변속성의 범위제한”을 제시했지만, 이것만으로는 부족하다. 여전히 여러명이 개발하는 프로젝트에서는 잘못 사용될 가능성이 있다. (불변속성의 범위제한이 100% 지켜졌는지 확인하기가 어렵기 때문)
    - 디자인 패턴 중에 service locator 패턴이라는게 있다.
    - 전역으로 접근하게 하는 코드 자체가 정말 필요한 지 먼저 판단해야 한다. 필요 없다면, 필요한 객체를 인수로 넘겨줄 수는 없는지 판단해보자

- [클린코드 5장 내용 인용](https://boring-km.dev/2021/02/13/clean_code/chapter05/) → 가독성이 좋아지면 유지보수성도 올라간다.
- 코드가 하는 일을 바꾸지 않고 유지보수하려면 제일 안전한 방법은 역시 테스트 코드를 작성하는 방법인 것 같다. - [https://blog.wadiz.kr/프론트엔드-개발자의-tdd-적응하기/](https://blog.wadiz.kr/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%EC%9E%90%EC%9D%98-tdd-%EC%A0%81%EC%9D%91%ED%95%98%EA%B8%B0/)
- 지금 돌이켜보면, 새로운 기능을 개발할 때에는 계속 코드 작성하고 실행해보고 고쳐보고를 반복하다보니 다 작성하고 나서 리팩토링을 시도해 본 적은 상대적으로 적었던 것 같다. (반성)
- 유연성 있는 코드를 작성하기 위해 계속해서 고민 중이다. → 그리고 이 유연성은 단순히 소스파일 하나에서 적용하기 보다는 아키텍처 레벨에서 생각해야 할 문제 같기도 하다.
- App 개발 아키텍처에서는 Clean Architecture 라는 용어가 계속 사용되고 있는데, Web에서도 적용되는 말인지는 잘 모르겠다.
    - Android의 공식문서에도 적혀있는 아키텍처 가이드: https://developer.android.com/topic/architecture?hl=ko
    - 컴포지션이 나와있는 문서: https://developer.android.com/training/dependency-injection?hl=ko
    - React Clean Architecture: https://betterprogramming.pub/clean-architecture-with-react-cc097a08b105

> TDD 연습해보기: https://github.com/boring-km/Android_Simple_Counter

## 기능 변경 없이 리팩토링 해보기

- https://github.com/boring-km/Android_Simple_Counter/commits/master

# 내용 정리

## 2.1 가독성을 통한 의도 전달

### 2.1.1 코드 개선

**가독성:** 의도를 전달하기 위한 코드의 성질

- 읽기 힘든 코드의 예

```tsx
function checkValue(str: boolean) {
	// 값 체크

	if (str !== false)
		// 반환
		return true;

	else; // 그렇지 않으면
		return str;
}
```

- 읽기 쉽게 작성된 동일한 코드

```tsx
function isTrue(bool: boolean) {
	if (bool)
		return true;
	else
		return false;
}
```

- 단순화한 동일한 코드

```tsx
function isTrue(bool: boolean) {
	return bool;
}
```

************************유지보수성:************************ 버그를 고치거나 기능을 추가하기 위해 일부 기능을 변경해야 할 때마다 새 코드를 어디에 놓을지 그 후보 위치를 얼마나 많이 조사해야 하는지를 나타내는 표현

> 범위가 제한되지 않은 불변속성(nonlocal invariant)의 손상: 나중에 변경되면서 예상치 못한 에러상황에 놓일 수 있다.
>

→ 불변속성을 더욱 쉽게 볼 수 있도록 서로 가깝게 이동시켜 유지보수성을 향상시켜야 한다.

→ **불변속성의 범위제한**: 함께 변하는 것은 함께 있어야 한다.

### 2.1.2 코드가 하는 일을 바꾸지 않고 유지보수하기

리팩터링 과정에서 성능이 다소 느려져도 거의 신경쓰지 않는다.

1. 대부분의 시스템에서 성능은 유지보수성과 가치가 떨어진다.
2. 성능이 중요한 경우 프로파일링 도구나 성능 전문가의 지도를 받아 리팩터링과 다른 단계에서 처리해야 한다.
- 리팩터링을 할 때는 블랙박스의 경계를 고려해야 한다.

### 핵심

1. 의도를 전달함으로써 가독성 향상
2. 불변속성의 범위제한을 통한 유지보수성 향상
3. 범위 밖의 코드에 영향을 주지 않고 1항과 2항을 수행

## 2.2 속도, 유연성 및 안정성 확보

### 2.2.1 상속보다는 컴포지션 사용

- 객체 내부에 다른 객체의 참조를 가지는 것

```tsx
interface Bird {
	hasBreak(): boolean;
	canFly(): boolean;
}
class CommonBird implements Bird {
	hasBreak() { return true; }
	canFly() { return true; } 
}
class Penguin extends CommonBird {
	canFly() { return false; }
}
```

```tsx
interface Bird {
	hasBreak(): boolean;
	canFly(): boolean;
}
class CommonBird implements Bird {
	hasBreak() { return true; }
	canFly() { return true; }
}
class Penguin implements Bird {
	private bird = new CommonBird();
	hasBreak() { return bird.hasBreak(); }
	canFly() { return false; }
}
```

- 상속하지 않고 인스턴스로 갖고 있는다.

************유연성************

컴포지션을 중심으로 만들어진 시스템을 사용하면 다른 방식보다 더 깔끔하게 코드를 결합하고 재사용할 수 있다.

### 2.2.2 수정이 아니라 추가로 코드를 변경

- 기존 기능에 영향을 주지 않고 기능을 추가하거나 변경할 수 있음 → 기존 코드를 변경하지 않음

************************************프로그래밍 속도************************************

새로운 것을 구현하거나 버그를 수정할 때는 주변 코드를 고려하여 아무것도 손상시키지 않는 것이다.

************안정성************

기존 코드를 항상 보존할 수 있다.

## 2.3 리팩터링과 일상 업무

리팩터링은 일상 업무가 돼야 한다.

- 레거시 시스템에서는 변경하기 전에 먼저 리팩터링하고 나서 작업 절차를 따르라
- 코드를 변경한 후에도 리팩터링 해라

### 2.3.1 학습 방법으로서의 리팩터링

- 배우는데 시간이 많이 걸린다.
- 코드를 연구하는 방법이다.
- 유연성이 있으면 환경설정 관리나 기능을 관리하는 시스템을 만드는 것이 가능하지만 유연성 없이는 유지보수가 불가능하다.

### 2.4 소프트웨어 분야에서 ‘도메인’ 정의하기

소프트웨어는 실생활의 특정 측면을 모델링한 것이다.

도메인: 실제 세계의 구성요소를 소프트웨어에서는 도메인이라 부른다.

## 요약

- 리팩터링은 기능 변경 없이 코드의 의도를 전달하고 불변속성의 범위를 제한하는 것이다.
- 상속보다 컴포지션을 사용함으로써, 추가를 통한 변경으로 개발 속도, 유연성, 안정성을 확보한다.
- 리팩터링을 일상 업무에 포함시켜 기술 부채가 쌓이지 않도록 해야 한다.
- 리팩터링을 연습하면 코드에 대한 독특한 관점을 얻을 수 있으며, 이로 인해 더 나은 해결책을 찾을 수 있다.