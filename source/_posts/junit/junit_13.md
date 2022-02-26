---
layout: post
title: "13. 까다로운 테스트"
categories: JUnit
tags: [dev, test, kotlin, java]
toc: true
---

- https://github.com/boring-km/JunitPractice

- 멀티스레드 코드 테스트
- 데이터베이스 코드 테스트
- 마치며

> 여기서 스레드와 영속성을 테스트하는 접근 방법은 2가지 주제에 기반을 둔다.
>
> 더 좋은 테스트 지원을 위한 '설계 다시 하기', 'stub과 mock을 사용해 의존성 끊기'

## 13.1 멀티스레드 코드 테스트
- 동시성 처리가 필요한 애플리케이션 코드를 테스트하는 것은 기술적으로 단위 테스트가 아닌 통합 테스트 영역이다.
- 멀티스레드 코드를 테스트하는 예제를 통해 방법을 익혀보자

### 13.1.1 단순하고 똑똑하게 유지
- **스레드 통제와 애플리케이션 코드 사이의 중첩을 최소화해라**
    - 스레드 없이 다량의 애플리케이션 코드를 단위 테스트할 수 있도록 설계를 변경해라
    - 남은 작은 코드에 대해 스레드에 집중적인 테스트를 해라
- **다른 사람의 작업을 믿어라**
    - 너무 자바의 내용이라 패스
    - (다른 사람들이 잘 만들어놓은 util 클래스 사용하라는 얘기, BlockingQueue)

### 13.1.2 모든 매칭 찾기
- 관련 있는 모든 프로파일을 수집하는 ProfileMatcher 클래스 예시
- (12장의 코드에서 많은 변화가 있는 것 같음)

```kotlin
import java.util.concurrent.Executors
import java.util.stream.Collectors


class ProfileMatcher {
   private val profiles: MutableMap<String, Profile> = HashMap()
   fun add(profile: Profile) {
      profiles[profile.getId()] = profile
   }

   fun findMatchingProfiles(
      criteria: Criteria?, listener: MatchListener
   ) {
      val executor = Executors.newFixedThreadPool(DEFAULT_POOL_SIZE)
      val matchSets = profiles.values.stream()
         .map { profile: Profile ->
            profile.getMatchSet(criteria)
         }
         .collect(Collectors.toList())
      for (set in matchSets) {
         val runnable = Runnable {
           if (set.matches())
               listener.foundMatch(profiles[set.profileId], set)
         }
         executor.execute(runnable)
      }
      executor.shutdown()
   }

   companion object {
      private const val DEFAULT_POOL_SIZE = 4
   }
}
```

- 각 프로파일에 대해 MatchSet 인스턴스를 모으는 findMatchingProfiles()
- 각 MatchSet에 대해 메서드는 별도의 스레드를 생성해 MatchSet 객체의 matches() return 값이 true이면 프로파일과 그에 맞는 MatchSet 객체를 listener로 보낸다.

### 13.1.3 애플리케이션 로직 추출
- findMatchingProfiles() 분리
- 분리한 collectMatchSets() 테스트 작성
- 매칭된 프로파일 정보를 listener로 넘기는 로직도 추출한다.
- 분리한 process() 테스트 작성
    - 모키토의 정적 mock() 메서드를 사용해 MatchListener 목 인스턴스를 생성한다.
    - 매칭되는 프로파일(주어진 조건에 매칭될 것으로 기대되는 프로파일)을 matcher 변수에 추가한다.
    - 주어진 조건 집합에 매칭되는 프로파일에 대한 MatchSet 객체를 요청한다.
    - mock listener와 MatchSet 객체를 넘겨 matcher 변수에 매칭 처리를 지시한다.
    - mockito를 활용해 mock으로 만든 listener 객체에 foundMatch() 메서드가 호출되었는지 확인한다.
      이때 매칭 프로파일과 MatchSet 객체를 인수로 넘긴다. 기대 사항이 맞지 않으면 mockito에 의해 테스트는 실패한다.

### 13.1.4 스레드 로직의 테스트 지원을 위해 재설계
- 남아있는 findMatchingProfiles() 메서드의 코드 대부분은 스레드 로직이다.
- 테스트를 위해 재설계를 해본다.

```kotlin
import java.util.concurrent.Executors
import java.util.function.BiConsumer
import java.util.stream.Collectors


class ProfileMatcher {
   private val profiles: MutableMap<String, Profile> = HashMap()
   fun add(profile: Profile) {
      profiles[profile.getId()] = profile
   }

   val executor = Executors.newFixedThreadPool(DEFAULT_POOL_SIZE)

   fun findMatchingProfiles(
      listener: MatchListener,
      matchSets: MutableList<MatchSet>,
      processFunction: BiConsumer<MatchListener, MatchSet>
   ) {
      for (set in matchSets) {
         val runnable = Runnable { processFunction.accept(listener, set) }
         executor.execute(runnable)
      }
      executor.shutdown()
   }

   fun findMatchingProfiles(criteria: Criteria, listener: MatchListener) {
      findMatchingProfiles(
         listener, collectMatchSets(criteria), ::process
      )
   }

   // 비동기로 실행할 코드
   fun process(listener: MatchListener, set: MatchSet) {
      if (set.matches()) listener.foundMatch(profiles[set.profileId], set)
   }

   fun collectMatchSets(criteria: Criteria?): MutableList<MatchSet> = profiles.values.stream()
      .map { profile: Profile ->
         profile.getMatchSet(
            criteria
         )
      }
      .collect(Collectors.toList())

   companion object {
      private const val DEFAULT_POOL_SIZE = 4
   }
}
```

### 13.1.5 스레드 로직을 위한 테스트 작성
- 테스트 코드 수정

```kotlin
import org.hamcrest.CoreMatchers
import org.hamcrest.CoreMatchers.equalTo
import org.hamcrest.MatcherAssert.assertThat
import org.junit.Before
import org.junit.Test
import org.mockito.Mockito.mock
import org.mockito.Mockito.verify
import java.util.*
import java.util.function.BiConsumer
import java.util.stream.Collectors


class ProfileMatcherTest {
  private lateinit var question: BooleanQuestion
  private lateinit var criteria: Criteria
  private lateinit var matcher: ProfileMatcher
  private lateinit var matchingProfile: Profile
  private lateinit var nonMatchingProfile: Profile
  private lateinit var listener: MatchListener

  @Before
  fun create() {
    question = BooleanQuestion(1, "")
    criteria = Criteria()
    criteria.add(Criterion(matchingAnswer(), Weight.MustMatch))
    matchingProfile = createMatchingProfile("matching")
    nonMatchingProfile = createNonMatchingProfile("nonMatching")
  }

  private fun createMatchingProfile(name: String): Profile {
    val profile = Profile(name)
    profile.add(matchingAnswer())
    return profile
  }

  private fun createNonMatchingProfile(name: String): Profile {
    val profile = Profile(name)
    profile.add(nonMatchingAnswer())
    return profile
  }

  @Before
  fun createMatcher() {
    matcher = ProfileMatcher()
  }

  @Before
  fun createMatchListener() {
    listener = mock(MatchListener::class.java)
  }

  @Test
  fun processNotifiesListenerOnMatch() {
    matcher.add(matchingProfile)
    val set = matchingProfile.getMatchSet(criteria)

    matcher.process(listener, set)

    verify(listener).foundMatch(matchingProfile, set)
  }

  @Test
  fun collectsMatchSets() {
    matcher.add(matchingProfile)
    matcher.add(nonMatchingProfile)

    val sets = matcher.collectMatchSets(criteria)

    assertThat(
      sets.stream()
        .map { set: MatchSet -> set.profileId }.collect(Collectors.toSet()),
      CoreMatchers.equalTo(
        HashSet(
          listOf(
            matchingProfile.getId(), nonMatchingProfile.getId()
          )
        )
      )
    )
  }

  private fun matchingAnswer(): Answer {
    return Answer(question, Bool.TRUE)
  }

  private fun nonMatchingAnswer(): Answer {
    return Answer(question, Bool.FALSE)
  }

  @Test
  fun gathersMatchingProfiles() {
    // 리스너가 수신하는 MatchSet 객체들의 프로파일 ID 목록을 저장할 문자열 Set 객체를 생성한다.
    val processedSets = Collections.synchronizedSet(HashSet<String>())

    // process() 메서드의 프로덕션 버전을 대신하는 함수
    val processFunction = BiConsumer { _: MatchListener, set: MatchSet ->
      // 각 콜백에서 MatchSet 객체의 프로파일 ID를 processedSets 변수에 추가
      processedSets.add(set.profileId)
    }

    // 테스트용 MatchSet 객체 생성
    val matchSets = createMatchSets(100)

    // processFunction() 구현을 넘긴다.
    // 실제로는 criteria로 테스트를 해야하지만, 내부의 로직은 matchSets의 내용을 그대로 MutableList<MatchSet>에 담아서 리턴한다.
    matcher.findMatchingProfiles(criteria, listener, matchSets, processFunction)

    // ExecutorService 객체를 가져와 모든 스레드의 실행이 완료될 때까지 기다림
    while (!matcher.executor.isTerminated);

    // processedSets 컬렉션이 테스트에서 생성된 모든 MatchSet 객체의 ID와 매칭되는지 검증한다.
    assertThat(processedSets, equalTo(matchSets.stream().map(MatchSet::profileId).collect(Collectors.toSet())))
  }

  private fun createMatchSets(count: Int): MutableList<MatchSet> {
    val sets = arrayListOf<MatchSet>()
    for (i in 0 until count) {
      sets.add(MatchSet(i.toString(), null, null))
    }
    return sets
  }
}
```

## 13.2 데이터베이스 테스트
- 챕터 5에서 잠깐 소개된 StatCompiler 코드에서 QuestionController와 상호 작용하는 questionText() 메서드에 관한 테스트를 작성해보기
- (5장 공부할 때도 따로 기록을 안해놔서 일단 아래에 적음)
- [StatCompiler.java](https://github.com/gilbutITbook/006814/blob/master/iloveyouboss_16-branch-persistence-redesign/src/iloveyouboss/domain/StatCompiler.java)

### 13.2.1 고마워, Controller
- questionText() 메서드에서 DB와 통신하는 **controller** 변수 때문에 테스트하기가 어려울 수 있다.
- [QuestionController.java](https://github.com/gilbutITbook/006814/blob/03260ef4099f6e4efa0779a598c3b7fcd2565fcd/iloveyouboss_16-branch-persistence-redesign/src/iloveyouboss/controller/QuestionController.java#L17)
- 해당 클래스에 대한 단위 테스트를 일일이 작성하는 것보다 진짜 DB와 성공적으로 상호 작용하는 QuestionController 클래스에 대한 테스트를 작성하는 것이 좋다.

### 13.2.2 데이터 문제
- JUnit의 테스트 대다수는 속도가 빠르길 원하는데 DB 테스트가 느려지지 않도록, 영속적인 모든 상호 작용을 시스템의 한곳으로 고립시켜 통합 테스트의 대상을 줄이도록 하자

> 테스트 안에서 데이터를 생성하고 관리해라
>
> 매 테스트는 그다음 자기가 쓸 데이터를 추가하거나 그것으로 작업해라(테스트 간 의존성 문제가 생기지 않도록 하기 위함)
>
> 테스트마다 트랜잭션을 초기화하고, 테스트가 끝나면 롤백하는 방법을 선택하자

- 통합 테스트는 작성과 유지 보수가 어렵다. 자주 망가지고, 그들이 깨졌을 때 문제를 디버깅하는 것도 상당히 오래걸리지만 여전히 **테스트 전략의 필수적인 부분이다.**

### 13.2.3 클린 룸 데이터베이스 테스트
- [예제의 코드](https://github.com/gilbutITbook/006814/blob/master/iloveyouboss_16-branch-persistence-redesign/test/iloveyouboss/controller/QuestionControllerTest.java)를 그대로 가져왔다.
- @Before, @After 메서드 모두에서 deleteAll() 메서드를 통해 매번 데이터를 초기화하고 있다.

```java
import static org.junit.Assert.*;
import static org.hamcrest.CoreMatchers.*;
import java.time.*;
import java.util.*;
import java.util.stream.*;
import iloveyouboss.domain.*;
import org.junit.*;
public class QuestionControllerTest {

   private QuestionController controller;
   
   @Before
   public void create() {
      controller = new QuestionController();
      controller.deleteAll();
   }
   
   @After
   public void cleanup() {
      controller.deleteAll();
   }

   @Test
   public void findsPersistedQuestionById() {
      int id = controller.addBooleanQuestion("question text");
      
      Question question = controller.find(id);
      
      assertThat(question.getText(), equalTo("question text"));
   }
   
   @Test
   public void questionAnswersDateAdded() {
      Instant now = new Date().toInstant();
      controller.setClock(Clock.fixed(now, ZoneId.of("America/Denver")));
      int id = controller.addBooleanQuestion("text");
      
      Question question = controller.find(id);
      
      assertThat(question.getCreateTimestamp(), equalTo(now));
   }
   
   @Test
   public void answersMultiplePersistedQuestions() {
      controller.addBooleanQuestion("q1");
      controller.addBooleanQuestion("q2");
      controller.addPercentileQuestion("q3", new String[] { "a1", "a2"});
      
      List<Question> questions = controller.getAll();
      
      assertThat(questions.stream()
            .map(Question::getText)
            .collect(Collectors.toList()), 
         equalTo(Arrays.asList("q1", "q2", "q3")));
   }
   @Test
   public void findsMatchingEntries() {
      controller.addBooleanQuestion("alpha 1");
      controller.addBooleanQuestion("alpha 2");
      controller.addBooleanQuestion("beta 1");

      List<Question> questions = controller.findWithMatchingText("alpha");
      
      assertThat(questions.stream()
            .map(Question::getText)
            .collect(Collectors.toList()),
         equalTo(Arrays.asList("alpha 1", "alpha 2")));
   }
}
```

### 13.2.4 controller를 목 처리
- 다시 questionText() 메서드의 테스트로 돌아가 QuestionController를 Mocking 해보는 것으로 마무리한다.
- [링크](https://github.com/gilbutITbook/006814/blob/master/iloveyouboss_16-branch-persistence-redesign/test/iloveyouboss/domain/StatCompilerTest.java)

```java
import static org.junit.Assert.*;
import iloveyouboss.controller.*;
import java.util.*;
import java.util.concurrent.atomic.*;
import org.junit.*;
import org.mockito.*;
import static org.hamcrest.CoreMatchers.*;
import static org.mockito.Mockito.*;

public class StatCompilerTest {

  @Mock private QuestionController controller;  // Mocking할 객체 선언
  @InjectMocks private StatCompiler stats;  // Mock 객체를 주입할 객체 선언

  @Before
  public void initialize() {
    stats = new StatCompiler();
    MockitoAnnotations.initMocks(this);
  }

  @Test
  public void questionTextDoesStuff() {
    when(controller.find(1)).thenReturn(new BooleanQuestion("text1"));
    when(controller.find(2)).thenReturn(new BooleanQuestion("text2"));
    List<BooleanAnswer> answers = new ArrayList<>();
    answers.add(new BooleanAnswer(1, true));
    answers.add(new BooleanAnswer(2, true));

    Map<Integer, String> questionText = stats.questionText(answers);

    Map<Integer, String> expected = new HashMap<>();
    expected.put(1, "text1");
    expected.put(2, "text2");
    assertThat(questionText, equalTo(expected));
  }
}
```

- StatCompiler 내부에 있는 QuestionController 인스턴스를 mockito를 이용해 생성해주었다.
- 그리고 테스트 코드 내 'when().thenReturn()'을 통해 QuestionController 인스턴스가 가상으로 동작할 코드와 그 결과를 정의한다.
- questionText()가 정상적으로 동작한다면 DB 의존성 없이 간단하게 테스트를 해볼 수 있게 된다.

## 13.3 마치며
- 멀티스레드와 데이터베이스 상호 작용은 그 자체로 험난하며, 많은 결함이 이 영역에서 출몰한다.

> 관심사를 분리해라. 애플리케이션 로직은 '스레드, 데이터베이스 혹은 문제를 일으킬 수 있는 다른 의존성'과 분리해라.
>
> 느리거나 휘발적인 코드를 mock으로 대체해 단위 테스트의 의존성을 끊어라
>
> 필요한 경우에는 통합 테스트를 작성하되, 단순하고 집중적으로 만들어라.

