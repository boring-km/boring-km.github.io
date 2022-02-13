---
layout: post
title: "8. 깔끔한 코드로 리팩토링하기"
date: 2022-01-25
categories: JUnit
tags: [dev, test, kotlin, java]
toc: true
---

- https://github.com/boring-km/JunitPractice

- 작은 리팩토링
- 메서드를 위한 더 좋은 집 찾기
- 자동 및 수동 리팩토링
- 과한 리팩토링?
- 마치며



## 8.1 작은 리팩토링

- 리팩토링은 코드를 이리저리 옮겨서 시스템이 정상 동작함을 보장하는 것이다.



### 8.1.1 리팩토링의 기회

- 2장에서 다뤘던 Profile 클래스의 matches() 살펴보기

```kotlin
package chapter02

class Profile() {

    fun matches(criteria: Criteria) : Boolean { // 기준과 맞으면 true 아니면 false
        score = 0

        var kill = false
        var anyMatches = false
        for (criterion: Criterion in criteria) {    // 해당 기준이 프로파일에 있는 답변과 맞는지 확인
            val answer = answers[criterion.answer.questionText]
            val match = criterion.weight == Weight.DontCare || answer!!.match(criterion.answer)
            if (!match && criterion.weight == Weight.MustMatch) {
                kill = true
            }
            if (match) {
                score += criterion.weight.value
            }
            anyMatches = anyMatches or match
        }
        if (kill)
            return false
        return anyMatches
    }
}

```



## 8.1 ~ 8.3 리팩토링

```kotlin
package chapter02

class Profile {

    fun matches(criteria: Criteria) : Boolean { // 기준과 맞으면 true 아니면 false
        score = 0

        var kill = false

        for (criterion: Criterion in criteria) {    // 해당 기준이 프로파일에 있는 답변과 맞는지 확인
            val match = criterion.matches(answerMatching(criterion))
            if (!match && criterion.weight == Weight.MustMatch) {
                kill = true
            }
            if (match) {
                score += criterion.weight.value
            }
        }
        if (kill)
            return false
        return anyMatches(criteria)
    }

    private fun anyMatches(criteria: Criteria) : Boolean {
        var anyMatches = false
        for (criterion: Criterion in criteria) {
            anyMatches = anyMatches or criterion.matches(answerMatching(criterion))
        }
        return anyMatches
    }

    private fun answerMatching(criterion: Criterion) = answers[criterion.answer.questionText]


}
```



## 8.4 과한 리팩토링?

- 모든 매칭의 전체 가중치를 계산하는 코드 추출

- 새로운 메서드와 반복문이 각각 3개가 되었다.

```kotlin
package chapter02

class Profile {

    fun matches(criteria: Criteria) : Boolean { // 기준과 맞으면 true 아니면 false
        calculateScore(criteria)

        if (doesNotMeetAnyMustMatchCriterion(criteria))
            return false
        return anyMatches(criteria)
    }

    private fun doesNotMeetAnyMustMatchCriterion(criteria: Criteria): Boolean {

        for (criterion: Criterion in criteria) {    // 해당 기준이 프로파일에 있는 답변과 맞는지 확인
            val match = criterion.matches(answerMatching(criterion))
            if (!match && criterion.weight == Weight.MustMatch) {
                return true
            }
        }
        return false
    }

    private fun calculateScore(criteria: Criteria) {
        score = 0
        for (criterion: Criterion in criteria) {
            if (criterion.matches(answerMatching(criterion)))
                score += criterion.weight.value
        }
    }

    private fun anyMatches(criteria: Criteria) : Boolean {
        var anyMatches = false
        for (criterion: Criterion in criteria) {
            anyMatches = anyMatches or criterion.matches(answerMatching(criterion))
        }
        return anyMatches
    }

    private fun answerMatching(criterion: Criterion) = answers[criterion.answer.questionText]


}

```



### 8.4.1 보상: 명확하고 테스트 가능한 단위들

- matches() 메서드는 이제 바로 이해 가능할 정도로 전체 알고리즘이 깔끔하게 정리되었다.

1. 주어진 조건에 따라 점수를 계산한다.
2. 프로파일이 어떤 필수 조건에 부합하지 않으면 false를 반환한다.
3. 그렇지 않으면 어떤 조건에 맞는지 여부를 반환한다.



### 8.4.2 성능 염려: 그러지 않아도 된다.

- 반복문이 3개나 만들어졌고, 메서드는 잠재적으로 실행 시간이 4배가 되었다.
- **그래서 어쩌라고?**
- 프로파일을 수백만 개 처리해야 한다면 성능은 최우선 고려 대상이 될 것이다.
- **하지만 성능이 즉시 문제가 되지 않는다면 어설픈 최적화 노력으로 시간을 낭비하기보다 코드를 깔끔하게 유지해라!!!**
- 깔끔한 설계는 성능을 위해 최적화할 때 즉시 대응할 수 있는 최선의 보호막이다.

- 성능이 당장 문제가 된다면 다른 일을 하기 전에 먼저 문제가 얼마나 심각한지 성능을 측정해보라
  - 작은 테스트 코드를 만들어 예전 코드와 리팩토링한 코드 간 몇 퍼센트의 성능 저하가 있는지 판단하고 비교해보라



## 8.5 마치며

- 대량의 코드를 빠르게 작성하는 것은 쉽다. (코드가 더러워지고 어떤 절차를 따르는지 파악하기가 어려워지지만)
- 단위 테스트는 기본 원칙을 깨지 않고 코드를 깔끔하게 유지해 주는 보호 장치를 제공한다.

