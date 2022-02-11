---
layout: post
title: "9. 더 큰 설계 문제"
date: 2022-02-02
categories: JUnit
tags: [dev, test, kotlin, java]
toc: true
author:
- Kangmin
---

- https://github.com/boring-km/JunitPractice
- Profile 클래스와 SRP
- 새로운 클래스 추출
- 명령-질의 분리
- 단위 테스트의 유지 보수 비용
- 다른 설계에 관한 생각들
- 마치며

## 9.1 Profile 클래스와 SRP

- [책임1] Profile 클래스는 회사 혹은 인물 정보를 추적하고 관리한다.
- Profile 클래스가 포착하는 이러한 정보 집합들은 시간이 지나면서 많이 바뀔 수 있다.
  - 더 많은 정보가 추가되거나 몇 가지 정보는 제거 혹은 변경될 수 있다.
- [책임2] 조건의 집합이 프로파일과 매칭되는지 여부 혹은 그 정도를 알려 주는 점수를 계산하는 것이다.
- **SRP 위반!** : 클래스는 변경할 때 한 가지 이유만 있어야 한다. 클래스는 작고 단일 목적을 추구한다.

## 9.2 새로운 클래스 추출
- 책임 두 개를 정의하고 있는 Profile
  1. 프로파일에 관한 정보 추적하기
  2. 조건 집합이 프로파일에 매칭되는지 혹은 그 정도를 판단하기

- 매칭 책임(2번)에 관한 코드를 MatchSet 클래스로 추출해보기

```kotlin
// 프로파일에 관한 정보를 추적하는 Profile 클래스
import java.util.function.Predicate
import java.util.stream.Collectors

class Profile(val name: String) {
  private val answers = HashMap<String, Answer>()
  var score: Int = 0
    private set


  // 회사에게 받은 질문에 대한 답변을 저장
  fun add(answer: Answer) {
    answers[answer.questionText] = answer
  }

  fun matches(criteria: Criteria) : Boolean {
    val matchSet = MatchSet(answers, criteria)
    score = matchSet.score
    return matchSet.matches()
  }

  override fun toString(): String {
    return name
  }

  fun find(pred: Predicate<Answer>) : List<Answer> {
    return answers.values.stream()
      .filter(pred)
      .collect(Collectors.toList())
  }
}

// 매칭에 대한 책임을 담당하는 MatchSet 클래스
class MatchSet(private var answers: HashMap<String, Answer>, private var criteria: Criteria) {

  var score = 0
    private set

  init {
    calculateScore(criteria)
  }

  private fun calculateScore(criteria: Criteria) {
    for (criterion: Criterion in criteria) {
      if (criterion.matches(answerMatching(criterion)))
        score += criterion.weight.value
    }
  }

  private fun answerMatching(criterion: Criterion) = answers[criterion.answer.questionText]

  fun matches(): Boolean {
    if (doesNotMeetAnyMustMatchCriterion(criteria)) // 특정 조건에 걸리면 false
      return false
    return anyMatches(criteria) // 조건이 맞는 다른 경우를 찾기
  }

  private fun doesNotMeetAnyMustMatchCriterion(criteria: Criteria): Boolean {

    for (criterion: Criterion in criteria) {
      val match = criterion.matches(answerMatching(criterion))
      if (!match && criterion.weight == Weight.MustMatch) {
        return true
      }
    }
    return false
  }

  private fun anyMatches(criteria: Criteria) : Boolean {
    var anyMatches = false
    for (criterion: Criterion in criteria) {
      anyMatches = anyMatches or criterion.matches(answerMatching(criterion))
    }
    return anyMatches
  }
}

```

## 9.3 명령-질의 분리
- 어떤 값을 반환하고 부작용을 발생시키는 (시스템에 있는 어떤 클래스 혹은 엔터티의 상태 변경) 메서드는 명령-질의 분리 원칙을 위반한다.
- 어떤 메서드는 실행(부작용을 생성하는 어떤 작업을 함) 하거나 질의에 대답(어떤 값을 반환)할 수 있으며, 두 작업을 모두 하면 안 된다.

```kotlin
// 문제의 명령-질의 분리 원칙을 위배한 부분
fun matches(criteria: Criteria) : Boolean {
  val matchSet = MatchSet(answers, criteria)
  score = matchSet.score
  return matchSet.matches()
}
```

- score 필드를 Profile에서 제거

```kotlin
import java.util.function.Predicate
import java.util.stream.Collectors

class Profile(val name: String) {
    private val answers = HashMap<String, Answer>()


    // 회사에게 받은 질문에 대한 답변을 저장
    fun add(answer: Answer) {
        answers[answer.questionText] = answer
    }

    fun getMatchSet(criteria: Criteria) : MatchSet {    // MatchSet을 반환하도록 수정 -> score로 쓰고 있던 코드에 영향이 발생!
        return MatchSet(answers, criteria)
    }

    override fun toString(): String {
        return name
    }

    fun find(pred: Predicate<Answer>) : List<Answer> {
        return answers.values.stream()
            .filter(pred)
            .collect(Collectors.toList())
    }
}

```

## 9.4 단위 테스트의 유지 보수 비용
- 리팩토링은 코드 동작을 변경하지 않고 코드 구현을 바꾸는 활동이며 테스트는 그 동작을 반영한다.
  - 하지만 현실에서는 클래스 동작을 변경하고 있다.
- 보통은 돌아오는 가치가 훨씬 크기 때문에 깨진 테스트 코드를 고치는 비용을 받아들인다.
  1. 결함이 거의 없는 코드를 갖는 이점
  2. 다른 코드가 깨질 것을 걱정하지 않으면서도 코드를 변경할 수 있는 이점
  3. 코드가 정확히 어떻게 동작하는지 알 수 있는 이점
- **좀 더 나아가서 실패하는 테스트의 정도를 부정적인 설계 지표로 인식하는 것도 생각해보자.**
- **더 많은 테스트가 동시에 깨질수록 더욱더 많은 설계 문제가 있을 것이다.**

### 9.4.1 자신을 보호하는 방법
- 코드 중복은 가장 큰 설계 문제이다.
- 코드 중복의 2가지 문제점
  1. 테스트를 따르기가 어려워진다.
  2. 작은 코드 조각들을 단일 메서드로 추출하면 그 코드 조각들을 젼경해야 할 때 미치는 영향을 최소화할 수 있다.
- 단위 테스트를 설정하는 데 코드가 몇 줄 혹은 수십 줄 필요하다면 그것은 시스템 설계에 문제가 있다는 것이다.
  - SRP 지켜라
- **private 메서드를 테스트하려는 충동은 클래스가 필요 이상으로 커졌다는 또 다른 힌트이다.**
- 단위 테스트가 어려워 보인다면 그것도 좋은 힌트다.
  - 설계를 개선하여 단위 테스트를 쉽게 만들자
  - 그러면 단위 테스트를 유지하는 비용을 줄일 수 있을 것이다.

### 9.4.2 깨진 테스트 고치기
- 직접 해보는 걸로 대신함

## 9.5 다른 설계에 관한 생각들
- Profile 클래스에서 질문 내용을 키로 사용하는 HashMap<String, Answer> 객체를 생성하고 있다.
- 하지만 동시에 answers 맵 참조를 새로 생성되는 MatchSet 객체로 넘긴다.
  - **두 클래스가 어떻게 답변을 탐색하고 점수를 구하는지에 대한 정보를 너무 많이 가지고 있다는 의미!**
  - 여러 클래스에 구현 상태가 흩어져 있을 때의 코드 냄새를 기능의 산재라고 한다.
  - answers 맵을 데이터베이스 테이블로 교체한다면 결국 여러 군데를 고쳐야 하기 때문이다.
- 답변 저장소를 AnswerCollection 클래스로 분리
- 변경된 git 코드 참조

## 9.6 마치며
- **설계를 지속적으로 개선해 나가는 자신감을 키우기 위해 단위 테스트의 커버리지를 높이세요**
- 기꺼이 새롭고 작은 클래스들과 메서드들을 만들라!
- 실무에서 코드가 여러 서비스와 상호작용하기에 항상 단위 테스트를 만드는 것이 쉽지만은 않다.
- Mock 객체를 도입해보자!
