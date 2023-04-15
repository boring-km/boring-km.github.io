---
layout: post
title: "11. 테스트 리팩토링"
categories: JUnit
date: 2022-02-20
tags: [dev, test, kotlin, java]
toc: true
---

- https://github.com/boring-km/JunitPractice

- 이해 검색
- 테스트 냄새: 불필요한 테스트 코드
- 테스트 냄새: 추상화 누락
- 테스트 냄새: 부적절한 정보
- 테스트 냄새: 부푼 생성
- 테스트 냄새: 다수의 단언
- 테스트 냄새: 테스트와 무관한 세부 사항들
- 테스트 냄새: 잘못된 조직
- 테스트 냄새: 암시적 의미
- 새로운 테스트 추가
- 마치며

> 테스트는 결함을 최소화하고 리팩토링으로 프로덕션 시스템을 깔끔하게 유지시켜 주지만, 이것은 지속적인 비용을 의미한다.
>
> 비용 증가로 이어지는 테스트 문제들을 해결해보자

## 11.1 이해 검색
- 많이 복잡해보이는 Search 클래스의 테스트 코드
- 테스트가 무엇을 하는지 완전히 이해하려면 테스트를 매 행마다 꼼꼼하게 읽고 의미 조각들을 맞추어 보아야 한다.
- 리팩토링을 시작해보자...
- (**나는 역시 Kotlin 코드로 변환한 후부터 시작하고자 한다.**)

```kotlin
import org.hamcrest.CoreMatchers
import org.junit.Assert
import org.junit.Test
import java.io.ByteArrayInputStream
import java.net.URL
import java.util.logging.Level


class SearchTest {
    @Test
    fun testSearch() {
        try {
            val pageContent = ("There are certain queer times and occasions "
                    + "in this strange mixed affair we call life when a man "
                    + "takes this whole universe for a vast practical joke, "
                    + "though the wit thereof he but dimly discerns, and more "
                    + "than suspects that the joke is at nobody's expense but "
                    + "his own.")
            val bytes = pageContent.toByteArray()
            val stream = ByteArrayInputStream(bytes)
            // search
            var search = Search(stream, "practical joke", "1")
            Search.LOGGER.level = Level.OFF
            search.setSurroundingCharacterCount(10)
            search.execute()
            Assert.assertFalse(search.errored())
            val matches: List<Match> = search.getMatches()
            Assert.assertThat<List<Match>>(matches, CoreMatchers.`is`(CoreMatchers.notNullValue()))
            Assert.assertTrue(matches.size >= 1)
            val match: Match = matches[0]
            Assert.assertThat(match.searchString, CoreMatchers.equalTo("practical joke"))
            Assert.assertThat(match.surroundingContext,
                CoreMatchers.equalTo("or a vast practical joke, though t"))
            stream.close()

            // negative
            val connection = URL("http://bit.ly/15sYPA7").openConnection()
            val inputStream = connection.getInputStream()
            search = Search(inputStream, "smelt", "http://bit.ly/15sYPA7")
            search.execute()
            Assert.assertThat(search.getMatches().size, CoreMatchers.equalTo(0))
            stream.close()
        } catch (e: Exception) {
            e.printStackTrace()
            Assert.fail("exception thrown in test" + e.message)
        }
    }
}
```

## 11.2 테스트 냄새: 불필요한 코드
- 테스트 코드가 예외를 기대하지 않는다면 그냥 throw 해버리면 된다. (try-catch 지우자)
- not null assert는 유용한 정보를 담고 있지 않은 불필요한 테스트이다.

```kotlin
import org.hamcrest.CoreMatchers
import org.junit.Assert
import org.junit.Test
import java.io.ByteArrayInputStream
import java.io.IOException
import java.net.URL
import java.util.logging.Level
import kotlin.jvm.Throws


class SearchTest {
    @Test
    @Throws(IOException::class)
    fun testSearch() {
        val pageContent = ("There are certain queer times and occasions "
                + "in this strange mixed affair we call life when a man "
                + "takes this whole universe for a vast practical joke, "
                + "though the wit thereof he but dimly discerns, and more "
                + "than suspects that the joke is at nobody's expense but "
                + "his own.")
        val bytes = pageContent.toByteArray()
        val stream = ByteArrayInputStream(bytes)
        // search
        var search = Search(stream, "practical joke", "1")
        Search.LOGGER.level = Level.OFF
        search.setSurroundingCharacterCount(10)
        search.execute()
        Assert.assertFalse(search.errored())
        val matches: List<Match> = search.getMatches()
        Assert.assertTrue(matches.size >= 1)
        val match: Match = matches[0]
        Assert.assertThat(match.searchString, CoreMatchers.equalTo("practical joke"))
        Assert.assertThat(match.surroundingContext,
            CoreMatchers.equalTo("or a vast practical joke, though t"))
        stream.close()

        // negative
        val connection = URL("http://bit.ly/15sYPA7").openConnection()
        val inputStream = connection.getInputStream()
        search = Search(inputStream, "smelt", "http://bit.ly/15sYPA7")
        search.execute()
        Assert.assertThat(search.getMatches().size, CoreMatchers.equalTo(0))
        stream.close()
    }
}
```

## 11.3 테스트 냄새: 추상화 누락
- 잘 구성된 테스트는 시스템과 상호 작용을 '데이터 준비하기', '시스템과 동작하기', '결과 단언하기' 세 가지 관점에서 보여준다.
- 추상화로 필수적인 개념을 최대화하고 불필요한 세부 사항은 감춘다.

> 좋은 테스트는 클라이언트가 시스템과 어떻게 상호 작용하는지 추상화한다.

```kotlin
import chapter11.util.ContainsMatches.Companion.containsMatches
import org.hamcrest.MatcherAssert.assertThat
import org.junit.Assert.*
import org.junit.Test
import java.io.ByteArrayInputStream
import java.io.IOException
import java.net.URL
import java.util.logging.Level


class SearchTest {
    @Test
    @Throws(IOException::class)
    fun testSearch() {
        val pageContent = ("There are certain queer times and occasions "
                + "in this strange mixed affair we call life when a man "
                + "takes this whole universe for a vast practical joke, "
                + "though the wit thereof he but dimly discerns, and more "
                + "than suspects that the joke is at nobody's expense but "
                + "his own.")
        val bytes = pageContent.toByteArray()
        val stream = ByteArrayInputStream(bytes)
        // search
        var search = Search(stream, "practical joke", "1")
        Search.LOGGER.level = Level.OFF
        search.setSurroundingCharacterCount(10)
        search.execute()
        assertFalse(search.errored())
        assertThat(search.getMatches(),
            containsMatches<Match>(arrayOf(
                Match("1",
                    "practical joke",
            "or a vast practical joke, though t"
                ))))
        stream.close()

        // negative
        val connection = URL("http://bit.ly/15sYPA7").openConnection()
        val inputStream = connection.getInputStream()
        search = Search(inputStream, "smelt", "http://bit.ly/15sYPA7")
        search.execute()
        assertTrue(search.getMatches().isEmpty())
        stream.close()
    }
}

```

- search.getMatches() 호출에서 반환된 매칭 목록에 대한 구현 세부 사항 5줄을 커스텀 Matcher Class로 구현 (ContainsMatches)

```kotlin
import org.hamcrest.Description
import org.hamcrest.Factory
import org.hamcrest.Matcher
import org.hamcrest.TypeSafeMatcher


class ContainsMatches(private val expected: Array<Match>) : TypeSafeMatcher<List<Match>>() {
    override fun describeTo(description: Description) {
        description.appendText("<$expected>")
    }

    private fun equals(expected: Match, actual: Match): Boolean {
        return expected.searchString == actual.searchString && expected.surroundingContext == actual.surroundingContext
    }

    override fun matchesSafely(actual: List<Match>): Boolean {
        if (actual.size != expected.size) return false
        for (i in expected.indices) if (!equals(expected[i], actual[i])) return false
        return true
    }

    companion object {
        @Factory
        fun <T> containsMatches(expected: Array<Match>): Matcher<List<Match>> {
            return ContainsMatches(expected)
        }
    }
}
```

## 11.4 테스트 냄새: 부적절한 정보
- 매직 리터럴: 프로그래밍에서 상수로 선언하지 않은 숫자 리터럴을 '매직 넘버'라고 하며, 코드에는 되도록 사용하면 안 된다. (**무의식적으로 쓴 코드가 생각난다. 내일 바로 수정해야겠다.**)
- "1"과 Search 생성자에 들어있는 불필요한 URL 값을 같은 상수로 전환
- **상수로 표현해 두면 의미를 분명하게 전달할 수 있다.**

## 11.5 테스트 냄새: 부푼 생성
- 구현 세부 사항 추상화를 위해 InputStream 객체를 생성해주는 도우미 메서드를 만들자


## 11.6 테스트 냄새: 다수의 단언
- search, negative assert를 분리하기
- 테스트마다 assert 하나로 만들면 테스트 이름을 깔끔하게 만들기 쉽다.

## 11.7 테스트 냄새: 테스트와 무관한 세부 사항들
- 군더더기가 되는 코드들을 @Before, @After로 분리하기
- 좋은 테스트는 독자가 테스트를 이해하는 데 다른 함수를 파헤치지 않도록 한다.

## 11.8 테스트 냄새: 잘못된 조직
- 테스트에서 어느 부분들이 준비(Arrange), 실행(Act), 단언(Assert) 부분인지 아는 것은 테스트를 빠르게 인지할 수 있게 한다.
- 띄어쓰기로 각 영역을 분리해준다.

## 11.9 테스트 냄새: 암시적 의미
- 각 테스트가 분명하게 대답해야 할 가장 큰 질문은 "왜 그러한 결과를 기대하는가?" 이다.
- 좀 더 나은 테스트 데이터를 골라서 명시적으로 바꿔보자

## 11.10 새로운 테스트 추가
- 검색할 때 errored() query에 true가 반환되는 테스트를 작성해보자
- 반대의 경우도 테스트

```kotlin
// 최종
import chapter11.util.ContainsMatches.Companion.containsMatches
import org.hamcrest.MatcherAssert.assertThat
import org.junit.After
import org.junit.Assert.*
import org.junit.Before
import org.junit.Test
import java.io.ByteArrayInputStream
import java.io.IOException
import java.io.InputStream
import java.net.MalformedURLException
import java.net.URL
import java.util.logging.Level


class SearchTest {

    private lateinit var stream: InputStream

    companion object {
        private const val A_TITLE = "1"
    }

    @Before
    fun turnOFfLogging() {
        Search.LOGGER.level = Level.OFF
    }

    @After
    @Throws(IOException::class)
    fun closeResources() {
        stream.close()
    }

    @Test
    fun testSearch() {
        stream = streamOn("rest of text here" +
                "1234567890search term1234567890" +
                "more rest of text")
        val search = Search(stream, "search term", A_TITLE)
        search.setSurroundingCharacterCount(10)

        search.execute()

        assertThat(search.getMatches(), containsMatches<Match>(arrayOf(
            Match(A_TITLE,
                "search term",
                "1234567890search term1234567890"
            ))))

    }

    private fun streamOn(pageContent: String): InputStream {
        return ByteArrayInputStream(pageContent.toByteArray())
    }

    @Test
    fun noMatchesReturnedWhenSearchStringNotInContent() {
        stream = streamOn("any text")
        val search = Search(stream, "text that doesn't match", A_TITLE)

        search.execute()

        assertTrue(search.getMatches().isEmpty())
    }

    @Test
    fun returnsErroredWhenUnableToReadStream() {
        stream = createStreamThrowingErrorWhenRead()
        val search = Search(stream, "", "")

        search.execute()

        assertTrue(search.errored())
    }

    private fun createStreamThrowingErrorWhenRead(): InputStream {
        return object : InputStream() {
            override fun read(): Int {
                throw IOException()
            }
        }
    }

    @Test
    fun erroredReturnsFalseWhenReadSucceeds() {
        stream = streamOn("")
        val search = Search(stream, "", "")

        search.execute()

        assertFalse(search.errored())
    }
}

```

## 11.11 마치며
- 리팩토링된 테스트는 단순해진다.
- 독자는 테스트 이름을 읽고 어떤 케이스인지 이해할 수 있다.
- 먼저 테스트의 실행 부분에 집중하여 코드가 무엇을 실행하는지 안다.
- **테스트로 시스템을 이해하고자 한다면 테스트를 깔끔하게 유지하는 것이 좋다.**
