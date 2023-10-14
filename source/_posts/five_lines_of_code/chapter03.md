---
layout: post
title: "Five Lines of Code - Chapter 03"
categories: five-lines-of-code
tags: [dev, refactoring]
toc: true
---

# 3. Shatter Long Functions

# 내용 정리

## 3.1 첫 번째 규칙: 왜 다섯 줄인가?

### 3.3.1 규칙: 다섯 줄 제한

정의: 메서드는 ({ 및 })를 제외하고 5줄 이상이 되어서는 안 된다.

스멜: 메서드가 길다는 것 자체가 스멜이다.  여기서 5줄 제한을 지키려다가 SRP(메서드는 한 가지 작업만 해야한다)를 지키는게 어려워질수도 있다. → 예외적으로 line 수를 변경해 줄 수도 있겠지만, 보통은 5줄 정도로 끝나는 경우가 많다.

의도: 관심을 가지지 않으면 시간이 지남에 따라 더 많은 기능이 추가되면서 메서드가 커지는 경향이 있고, 그로 인해 코드를 점점 더 이해하기 어려워진다.

참조: 메서드 추출 리팩터링을 참고하자

```kotlin
fun containsEven(arr: List<List<Int>>): Boolean {
    for (x in arr.indices) {
        for (y in arr[x].indices) {
            if (arr[x][y] % 2 == 0) {
                return true
            }
        }
    }
    return false
}

fun minimum(arr: List<List<Int>>): Int {
    var result = Int.MAX_VALUE
    arr.forEach { it.forEach { result = it.coerceAtMost(result) } }
    return result
}
```

## 3.2 함수 분해를 위한 리팩터링 패턴 소개

1. 코드를 이해하기 위해 함수명을 고려하라
2. 코드의 ‘형태’ 를 살펴보자

### 3.2.1 리팩터링 패턴: 메서드 추출

설명: 한 메서드의 일부를 취해서 자체 메서드로 추출합니다.

절차

1. 추출할 줄의 주변을 빈 줄로 표시하는데, 주석으로 표시할 수도 있습니다.
2. 원하는 이름으로 새로운 빈 메서드를 만듭니다.
3. 그룹의 맨 위에서 새로운 메서드를 호출합니다.
4. 그룹의 모든 줄을 선택해서 잘라내어 새로운 메서드의 본문에 붙여 넣습니다.
5. 컴파일합니다.
6. 매개변수를 도입하여 호출하는 쪽의 오류를 발생시킵니다.
7. 이러한 매개변수 중 하나를 반환 값으로 할당해야 할 경우
    1. 새로운 메서드의 마지막에 return p;를 추가합니다.
    2. 새로운 메서드를 호출하는 쪽에서 p = newMethod(…)와 같이 반환 값을 할당합니다.
8. 컴파일합니다.
9. 호출 시 인자를 전달해서 오류를 잡습니다.
10. 사용하지 않는 빈 줄과 주석을 제거합니다.

예제: 함수 중간에 if문으로 감싸여져 있는 코드를 메서드로 추출하여 그 결과를 이용하는 리팩터링이 들어가 있다.

https://github.com/FiveLinesofCodeStudy/2D-Puzzle-Compose/commit/29dacd79fd6e7f0911ada24aef5cf7509fb2cf24

## 3.3 추상화 수준을 맞추기 위한 함수 분해

### 3.3.1 규칙: 호출 또는 전달, 한 가지만 할 것

정의: 함수 내에서는 객체에 있는 메서드를 호출하거나 객체를 인자로 전달할 수 있지만 둘을 섞어 사용해서는 안 된다.

설명: 더 많은 메서드를 도입하고 여러 가지를 매개변수로 전달하기 시작하면 결국 책임이 고르지 않게 될 수 있다. 동일한 수준의 추상화를 유지하는 편이 코드를 읽기가 더 쉬워진다.

- 변경 전

```tsx
function average(arr: number[]) {
	return sum(arr) / arr.length;
}
```

- 변경 후

```tsx
function average(arr: number[]) {
	return sum(arr) / size(arr);
}
```

스멜: ‘함수의 내용은 동일한 추상화 수준에 있어야 한다’ (클린코드에도 있음)

- 찾는 법?: 인자로 전달된 변수 옆 ‘.’ 으로 찾아보자

의도: 메서드에서 몇 가지 세부적인 부분을 추출해서 추상화를 도입하면 이 규칙은 연관된 다른 세부적인 부분도 추출하게 한다. 메서드 내부의 추상화 수준이 항상 동일하게 유지된다.

참조: 메서드 추출 리팩터링 패턴을 참조

### 3.3.2 규칙 적용

- 내가 작성한 Kotlin Multiplatform에서는 적용이 안되어서 주석만 남기고 패스

## 3.4 좋은 함수 이름의 속성

- 정직해야 한다. 함수의 의도를 설명해야 한다.
- 완전해야 한다. 함수가 하는 모든 것을 담아야 한다.
- 도메인에서 일하는 사람이 이해할 수 있어야 한다. 작업 중인 도메인에서 사용하는 단어를 사용해라. 그렇게 하면 의사소통이 더욱 효율적이게 되고 팀원 및 고객과 코드에 대해 더 쉽게 이야기할 수 있다는 장점이 있다.

https://github.com/FiveLinesofCodeStudy/2D-Puzzle-Compose/commit/7a181b758a8a689d83ecfc2b75238e107dd73d70

## 3.5 너무 많은 일을 하는 함수 분리하기

### 3.5.1 규칙: if 문은 함수의 시작에만 배치

정의: if 문이 있는 경우 해당 if 문은 함수의 첫 번째 항목이어야 한다.

- 변경 전

```kotlin
fun reportPrimes(n : Int) {
    for (i in 2 until n) {
        if (isPrime(i)) {
            println(i)
        }
    }
}
```

- 변경 후

```kotlin
fun reportPrimes(n : Int) {
    for (i in 2 until n) {
        reportIfPrime(i)
    }
}

private fun reportIfPrime(i: Int) {
    if (isPrime(i)) {
        println(i)
    }
}
```

스멜: 다섯 줄 제한과 같이, 이 규칙은 함수가 한 가지 이상의 작업을 수행하는 스멜을 막기 위해 존재한다.

의도: if 문이 하나의 작업이기 때문에 이를 분리할 때 이어지는 else if는 if 문과 분리할 수 없는 원자 단위로 본다.

### 3.5.2 규칙 적용

https://github.com/FiveLinesofCodeStudy/2D-Puzzle-Compose/commit/7551da1184fc9c812f7f02442b77015be3cf83a9

## 요약

- 다섯 줄 제한 규칙은 메서드는 다섯 줄 이하여야 한다는 말
- 호출 또는 전달, 한 가지만 할 것: 하나의 메서드 내에서 객체에 있는 메서드를 호출하거나 객체를 매개변수로 전달할 수 있지만, 둘 다 해서는 안 된다는 말
- 메서드 이름은 투명하고 완전해야 하며 이해할 수 있어야 한다. 메서드 추출을 사용하면 가독성도 올라간다.
- if 문은 함수의 시작에만 배치: if를 사용해 조건을 확인하는 경우 한 가지 작업만 수행하므로 메서드가 다른 작업을 수행하지 못하게 한다.


# 느낀점

- 3.5.1 규칙(if 문은 함수의 시작에만 배치) 은 해본적이 없지만 정말 괜찮은 방법인 것 같다.
- 3.3의 추상화 수준을 맞추는 리팩터링도 말만 들어봤지 이 규칙을 적용하려고 리팩토링을 수행했던 적은 거의 없던 것 같다.
- 나머지 리팩터링 패턴에 대해서는 이미 자주하고 있는 방법들이었지만, 무의식적으로 하기 보다 구체적으로 `어떤 상황에서 이렇게 리팩터링을 해라!` 를 알려주니까 더 좋았다.
- 빨리 기존 코드들을 리팩터링 하고 싶어졌다.

# 짧게나마 리팩터링을 해보았다

https://github.com/ZeroWastePortenday/green_life_app/commit/fe5e01fb9b988e47c7e6a5a013d24569bfa2b19a

### 리팩터링 예정 1

→ https://play.google.com/store/apps/details?id=com.yeolsimee.moneysaving

Android 앱으로만 되어 있는데 이번에 Flutter 앱으로 다시 만들어서 iOS 동시 배포예정

### 리팩터링 예정 2

https://github.com/boring-km/color-from-image

### 리팩터링 예정 3

- 여기가 정말 고비
- https://github.com/Monday-Rocket/ac_project_app