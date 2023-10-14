---
layout: post
title: "Five Lines of Code - Chapter 04"
categories: five-lines-of-code
tags: [dev, refactoring]
toc: true
---

# 4. Make Type Codes Work

# 느낀점

- 결국 대부분의 내용은 if-else 문 제거하면서 interface로 비슷한 동작들을 하나의 메소드로 추상화하여 클래스로 이관하는 리팩터링이었다.
- 지금은 코드가 많이 늘어난 것처럼 보이지만, 변경에 얼마나 유리할 지 잘 드러나는 구조가 되었다.

# 내용 정리

## 4.1 간단한 if문 리팩터링

### 4.1.1 규칙: if 문에서 else를 사용하지 말 것

정의: 프로그램에서 이해하지 못하는 타입을 검사하지 않는 한 if 문에서 else 문 사용하지 말자

- 애플리케이션 외부에서 입력을 받는 프로그램의 경계라면 문제 없다.
- if: 검사, if-else: 의사결정

### 4.1.2 규칙 적용

- handleInput에서 if-else를 제거
    - 근데 나는 이미 Kotlin 버전으로 작성하면서 `when`을 사용하고 있었긴했다.

### 4.1.3 리팩터링 패턴: 클래스로 타입 코드 대체

- enum을 interface로 변환하고 enum의 값들은 class가 된다.
- 각 값에 속성을 추가하고 해당 특정 값과 관련된 기능을 특성에 맞게 만들 수 있다.

### 4.1.4, 4.1.5 리팩터링 패턴: 클래스로의 코드 이관

- 4.1.3에서 변환한 interface의 클래스들에서 공통으로 사용할 method를 만든다.

https://github.com/FiveLinesofCodeStudy/2D-Puzzle-Compose/commit/965e583f623808d2e08c33f8615c8f1d855553c2

### 4.1.6, 4.1.7 리팩터링 패턴: 메서드 인라인화

https://github.com/FiveLinesofCodeStudy/2D-Puzzle-Compose/commit/1dd3a0e2a4ead9065d84bc4c956de58c94ee0caa

- 모든 호출 측을 수정하여 원래의 메서드를 제거한다.
- 인텔리제이에서는 이런 단축키가 있다.

![img.png](/images/five_lines_of_code/chapter04/img.png)

- 변경 전

```kotlin
private fun handleInputs(
    inputs: SnapshotStateList<Input>,
    mapState: MutableState<Array<Array<Int>>>,
    playerx: MutableState<Int>,
    playery: MutableState<Int>
) {
    while (inputs.size > 0) {
        val current = inputs.removeLast()
        handleInput(current, mapState, playerx, playery)
    }
}

private fun handleInput(
    input: Input,
    mapState: MutableState<Array<Array<Int>>>,
    playerx: MutableState<Int>,
    playery: MutableState<Int>
) {
    input.handle(mapState, playerx, playery)
}
```

- 변경 후

```kotlin
private fun handleInputs(
    inputs: SnapshotStateList<Input>,
    mapState: MutableState<Array<Array<Int>>>,
    playerx: MutableState<Int>,
    playery: MutableState<Int>
) {
    while (inputs.size > 0) {
        val current = inputs.removeLast()
        current.handle(mapState, playerx, playery)
    }
}
```

## 4.2 긴 if 문의 리팩터링

https://github.com/FiveLinesofCodeStudy/2D-Puzzle-Compose/commit/c88de4fe37c5cf6e5d7a91ea95b9ed6a728073ed

- 이 리팩토링 하면서 map 객체도 전역으로 참조하는 것으로 예제처럼 바꿨다.

### 4.2.1, 4.2.2 리팩터링 패턴: 메서드 전문화

- 일반성 제거
- 공통 메서드로 사용하고 있는 걸 아예 각각의 구체적 이름으로 된 메서드들로 나누면서 매개변수 제거

### 4.2.3, 4.2.4 switch 사용 여부

정의: default 케이스가 없고 모든 case 반환 값이 있는 경우가 아니라면 `switch` 사용 x (Kotlin에서는 `when`이 되겠다)

의도: switch를 else if 체인 문으로 변환하고 이를 다시 클래스로 만들어라

### 4.2.5 if 제거하기

if-else를 클래스로 만드는 과정에서 사라진다.

## 4.3 코드 중복 처리

### 4.3.1, 4.3.2 규칙: 인터페이스에서만 상속받을 것

- 클래스나 추상 클래스에서 상속받지 말라!
- 중복을 줄이고 코드의 줄을 줄이고자 할 때 편리하지만 단점이 훨씬 많다

### 4.3.3 클래스에 있는 코드의 중복은 다 무엇일까?

- 코드의 중복이 좋지 않은 것은 누구나 알고 있지
- 이 책에서는 변경이 많이 일어날 것을 전제로 하기 때문에 나중에는 중복이 아니게 됨을 암시하고 있다.

https://github.com/FiveLinesofCodeStudy/2D-Puzzle-Compose/commit/7ec9c84b4be26a59032ad275362999458952c8ab

## 4.4 복잡한 if 체인 구문 리팩터링

- `||` 표현식 사용되고 있는 if 문들
- `moveHorizontal`, `moveVertical`, `updateTile` 클래스로 이관

https://github.com/FiveLinesofCodeStudy/2D-Puzzle-Compose/commit/1b226173707daa8c9fde7c7e425536d0438bffe9

- 키보드 키 이벤트 처리하는 것도 최대한 리팩토링을 해봤다. KeyEvent 클래스에는 손을 댈 수가 없어

## 4.5 필요 없는 코드 제거하기

- 불필요한 메서드들을 인터페이스에서 제거해보자

### 4.5.1 리팩터링 패턴: 삭제 후 컴파일하기

- 새로운 기능을 구현하는 동안에는 수행하지 말라
- 인터페이스가 범위 내에서만 사용된다는 것을 알고 있다면 수동으로 정리해야 한다.
- `color()`, `isEdible()`, `isPushable()` 인라인화

https://github.com/FiveLinesofCodeStudy/2D-Puzzle-Compose/commit/bce7c1c902b10a51824e2f99077ca87e250587fe