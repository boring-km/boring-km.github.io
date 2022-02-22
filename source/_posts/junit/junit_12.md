---
layout: post
title: "12. 테스트 주도 개발"
categories: JUnit
tags: [dev, test, kotlin, java]
toc: true
---

- https://github.com/boring-km/JunitPractice/commits/master/src/test/kotlin/chapter12

- TDD의 주된 이익
- 단순하게 시작
- 또 다른 증분 추가
- 테스트 정리
- 또 다른 작은 증분
- 다수의 응답 지원: 작은 설계 우회로
- 인터페이스 확장
- 마지막 테스트들
- 문서로서의 테스트
- TDD의 리듬
- 마치며

> TDD에서 단위 테스트를 시스템의 모양을 잡고 통제하는 도구로 활용해야 한다.
>
> 단위 테스트는 소프트웨어를 어떻게 만들어야 할지에 관한 잘 훈련된 사이클의 핵심적인 부분이다.

## 12.1 TDD의 주된 이익
- **코드가 예상한 대로 동작한다는 자신감을 얻는 것**
- TDD에서는 코드가 변경될 것이라는 두려움을 지울 수 있다.
- (전에 공부한 TDD 책의 내용과 유사한 점이 많아 PASS)

## 12.2 단순하게 시작

1. 실패하는 테스트 코드 작성하기
2. 테스트 통과시키기
3. 이전 두 단계에서 추가되거나 변경된 코드 개선하기

- (전에 읽었던 TDD 책의 3단계와 표현된 방식은 좀 달라도 이해는 하고 있다.)
- 일단 테스트 통과시키기까지 해보기

## 12.3 또 다른 증분 추가
- 실패하는 각 테스트에 대해 그 테스트를 통과할 수 있는 코드만 추가해라
- 가능한 가장 작은 증분을 추가하는 것이다.
- 프로파일이 매칭되는 경우를 찾는 테스트를 추가해보자.

## 12.4 테스트 정리
- 2번까지 진행한 코드에서 테스트 코드를 한번 정리해준다.

## 12.5 또 다른 작은 증분
- Profile 인스턴스가 매칭되는 Answer 객체가 없을 때 matches() 메서드가 false를 반환하는 것이다.
- TDD로 생각하는 부분의 일부는 작성할 필요가 있는 다음 테스트를 결정하는 것이다.
- 프로그래머로서의 임무는 코드가 다루어야 하는 모든 가능한 순열과 시나리오를 이해하는 것이다.
- **TDD로 성공하려면 이들 시나리오를 테스트로 만들고 각 테스트를 통과하게 만드는 코드 증분을 최소화하는 순으로 코드를 작성하는 것이다.**

## 12.6 다수의 응답 지원: 작은 설계 우회로
- 다수의 응답을 포함하는 Profile에 대한 시나리오를 처리해본다.
- TDD를 할 때 다른 코드를 전혀 건드리지 않고 Profile 클래스만 변경할 필요는 없다.
- 필요하다면 설계를 변경하여 다른 클래스로 넘어가도 된다.

## 12.7 인터페이스 확장
- 컬렉션 객체 Criteria를 matches()로 처리할 수 있도록 수정해본다.

## 12.8 마지막 테스트들
- 다른 특수 경우 추가해보기
- 조건이 "don't care" 라면 matches() 메서드는 true 반환
- 점수 계산 요구사항 추가해보기
- ProfileMatch 클래스로 책임 이동하여 Profile 클래스의 SRP 준수하자

```kotlin
class Profile {
    private var answers: MutableMap<String, Answer> = HashMap()

    fun add(answer: Answer) {
        answers[answer.questionText] = answer
    }

    fun match(criteria: Criteria): ProfileMatch {
        return ProfileMatch(answers, criteria)
    }
}

class ProfileMatch(private val answers: MutableMap<String, Answer>, private val criteria: Criteria) {
    var score = 0
    var isMatch = false

    private fun matches(): Boolean {
        var matches = false
        for (criterion: Criterion in criteria) {
            if (matches(criterion)) {
                matches = true
            } else if (criterion.weight == Weight.MustMatch) {
                return false
            }
        }
        if (matches) score = 100
        return matches
    }

    init {
        isMatch = matches()
    }

    private fun matches(criterion: Criterion): Boolean {
        return criterion.weight == Weight.DontCare ||
                criterion.answer.match(getMatchingProfileAnswer(criterion))
    }

    private fun getMatchingProfileAnswer(criterion: Criterion): Answer? {
        return answers[criterion.answer.questionText]
    }
}
```

## 12.9 문서로서의 테스트
- 테스트 코드들을 하나의 클래스에 넣을 필요도 없다.
- 테스트 이름도 수정해보자

```kotlin
// 최종
import org.hamcrest.CoreMatchers.equalTo
import org.hamcrest.MatcherAssert.assertThat
import org.junit.Assert.assertFalse
import org.junit.Assert.assertTrue
import org.junit.Before
import org.junit.Test

open class ProfileTest {

    lateinit var profile: Profile
    lateinit var questionIsThereRelocation: BooleanQuestion
    lateinit var answerThereIsRelocation: Answer
    lateinit var answerThereIsNotRelocation: Answer
    lateinit var questionReimbursesTuition: BooleanQuestion
    lateinit var answerDoesNotReimburseTuition: Answer
    lateinit var answerReimbursesTuition: Answer

    lateinit var criteria: Criteria

    @Before
    fun createCriteria() {
        criteria = Criteria()
    }

    @Before
    fun createProfile() {
        profile = Profile()
    }

    @Before
    fun createQuestionAndAnswer() { // 이렇게 2가지를 같이 생성하는 의미로 사용해도 되려나...?
        questionIsThereRelocation = BooleanQuestion(1, "Relocation package?")
        answerThereIsRelocation = Answer(questionIsThereRelocation, Bool.TRUE)
        answerThereIsNotRelocation = Answer(questionIsThereRelocation, Bool.FALSE)
        questionReimbursesTuition = BooleanQuestion(1, "Reimburses tuition?")
        answerDoesNotReimburseTuition = Answer(questionReimbursesTuition, Bool.FALSE)
        answerReimbursesTuition = Answer(questionReimbursesTuition, Bool.TRUE)
    }
}

class Profile_MatchesCriterionTest: ProfileTest() {

    @Test
    fun trueWhenMatchesSoleAnswer() {
        profile.add(answerThereIsRelocation)
        val criterion = Criterion(answerThereIsRelocation, Weight.Important)

        val result = profile.match(criterion).isMatch

        assertTrue(result)
    }

    @Test
    fun falseWhenNoMatchingAnswerContained() {
        profile.add(answerThereIsNotRelocation)
        val criterion = Criterion(answerThereIsRelocation, Weight.Important)

        val result = profile.match(criterion).isMatch

        assertFalse(result)
    }

    @Test
    fun matchesWhenContainsMultipleAnswers() {
        profile.add(answerThereIsRelocation)
        profile.add(answerDoesNotReimburseTuition)
        val criterion = Criterion(answerThereIsRelocation, Weight.Important)

        assertTrue(profile.match(criterion).isMatch)
    }

    @Test
    fun matchesWhenCriterionIsDontCare() {
        profile.add(answerDoesNotReimburseTuition)
        val criterion = Criterion(answerReimbursesTuition, Weight.DontCare)

        assertTrue(profile.match(criterion).isMatch)
    }
}

class Profile_MatchesCriteriaTest: ProfileTest() {

    @Test
    fun falseWhenNoneOfMultipleCriteriaMatch() {
        profile.add(answerDoesNotReimburseTuition)
        val criteria = Criteria()
        criteria.add(Criterion(answerThereIsRelocation, Weight.Important))
        criteria.add(Criterion(answerReimbursesTuition, Weight.Important))

        val result = profile.match(criteria).isMatch

        assertFalse(result)
    }

    @Test
    fun trueWhenAnyOfMultipleCriteriaMatch() {
        profile.add(answerThereIsRelocation)
        val criteria = Criteria()
        criteria.add(Criterion(answerThereIsRelocation, Weight.Important))
        criteria.add(Criterion(answerReimbursesTuition, Weight.Important))
        assertTrue(profile.match(criteria).isMatch)   // AAA 규칙을 안지켜도 잘 읽힌다.
    }

    @Test
    fun falseWhenAnyMustMeetCriteriaNotMet() {
        profile.add(answerThereIsRelocation)
        profile.add(answerDoesNotReimburseTuition)
        criteria.add(Criterion(answerThereIsRelocation, Weight.Important))
        criteria.add(Criterion(answerReimbursesTuition, Weight.MustMatch))

        assertFalse(profile.match(criteria).isMatch)
    }
}

class Profile_ScoreTest: ProfileTest() {
    @Test
    fun zeroWhenThereAreNoMatches() {
        criteria.add(Criterion(answerThereIsRelocation, Weight.Important))

        val match: ProfileMatch = profile.match(criteria)

        assertThat(match.score, equalTo(0))
    }

    @Test
    fun 모두_일치하면_Score는_100이다() {
        criteria.add(Criterion(answerThereIsRelocation, Weight.DontCare))

        val match = profile.match(criteria)

        assertThat(match.score, equalTo(100))
    }
}
```

## 12.10 TDD의 리듬
- TDD의 리듬을 형성하면 좋다! ㅋㅋ
- 10분 정도 시간 제한을 걸어보고 테스트 통과를 못했다면 작업 중인 코드를 버리고 다시 좀 더 작은 단계로 도전해보자
- 각 TDD 사이클은 테스트를 가설로 한 시간 제한이 있는 실험으로 취급해라

## 12.11 마치며
- **테스트를 작성하고, 그것을 통과하고 코드가 깔끔한지 확인하고 반복하는 것이다!**
