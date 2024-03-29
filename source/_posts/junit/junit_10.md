---
layout: post
title: "10. 목 객체 사용"
date: 2022-02-05
categories: JUnit
tags: [dev, test, kotlin, java]
toc: true
---

- https://github.com/boring-km/JunitPractice
- 
- 테스트 도전 과제
- 번거로운 동작을 스텁으로 대체
- 테스트를 지원하기 위한 설계 변경
- 스텁에 지능 더하기: 인자 검증
- 목 도구를 사용하여 테스트 단순화
- 마지막 하나의 단순화: 주입 도구 소개
- 목을 올바르게 사용할 때 중요한 것
- 마치며

## 10.1 테스트 도전 과제
- 주소를 입력하는 대신 사용자는 지도에서 Profile 주소를 나타내는 지점을 선택할 수 있다.
- 애플리케이션은 선택된 지점의 위도와 경도 좌표를 AddressRetriever 클래스의 retrieve() 메서드로 넘긴다.
- chapter10 패키지 참조
- HTTP 호출을 실행하기 때문에 생기는 2가지 시사점
  - 실제 호출에 대한 테스트는 나머지 대다수의 빠른 테스트들에 비해 속도가 느릴 것이다.
  - Nominatim HTTP API가 항상 가용한지 보장할 수 없다. 통제 밖에 있다.
- API의 테스트 버전이라면 가용성 부분을 통제할 수 있겠지만 여전히 느리다.
- 의존성 있는 다른 코드와 분리하여 retrieve() 메서드의 로직에 관한 단위 테스트를 하자

## 10.2 번거로운 동작을 스텁(stub)으로 대체
- 먼저 HTTP 호출에서 반환되는 JSON 응답을 이용해 Address 객체를 생성하는 로직을 검증하는 데 집중하자
- stub: 테스트 용도로 하드 코딩한 값을 반환하는 구현체

```kotlin
import chapter10.AddressRetriever
import chapter10.util.Http
import org.hamcrest.CoreMatchers.equalTo
import org.hamcrest.MatcherAssert.assertThat
import org.junit.Test

class AddressRetrieverTest {
    @Test
    fun answersAppropriateAddressForValidCoordinates() {
        val http: Http = object : Http {
            override fun get(url: String): String {
                return "{\"address\": {" +
                        "\"house_number\":\"324\"," +
                        "\"road\":\"North Tejon Street\"," +
                        "\"city\": \"Colorado Springs\"," +
                        "\"state\": \"Colorado\"," +
                        "\"postcode\": \"80903\"," +
                        "\"country_code\": \"us\"}" +
                        "}"
            }
        }
        val retriever = AddressRetriever(http)
        val address = retriever.retrieve(38.0, -104.0)

        assertThat(address.houseNumber, equalTo("324"))
        assertThat(address.road, equalTo("North Tejon Street"))
        assertThat(address.city, equalTo("Colorado Springs"))
        assertThat(address.state, equalTo("Colorado"))
        assertThat(address.zip, equalTo("80903"))
    }
}
```

- Http의 stub 인스턴스 생성한다.
  - 스텁은 get(url: String) 단일 메서드가 있으며 하드 코딩된 JSON 문자열을 반환한다.
- 테스트는 AddressRetriever 객체를 생성하고 생성자에 스텁을 전달한다.
- AddressRetriever 객체는 스텁을 저장한다.
- 실행될 때 retrieve() 메서드는 먼저 넘어온 파라미터의 포맷을 정한다.
  그다음 스텁이 저장된 http 필드에 get() 메서드를 호출한다.
  retrieve() 메서드는 http 필드가 스텁을 참조하는지 프로덕션 구현을 참조하는지 관여하지 않는다.
  메서드가 아는 것은 get() 메서드를 구현한 객체와 상호 작용하고 있다는 점이다.
- 스텁은 테스트에 하드 코딩된 JSON 문자열을 당연히 반환하겠지!!
- 나머지 retrieve() 메서드는 하드 코딩된 JSON 문자열을 파싱하고 그에 따라 Address 객체를 구성한다.
- 테스트는 반환된 Address 객체의 요소를 검증한다.

## 10.3 테스트를 지원하기 위한 설계 변경
- 이전에 Http 인스턴스는 retrieve() 메서드에서 생성되어 AddressRetriever 클래스의 세부 사항이었다.
- 이제 AddressRetriever 클래스와 상호 작용하는 어떤 클라이언트는 다음과 같이 적절한 Http 인스턴스를 생성하여 넘겨주어야 한다.

```kotlin
val retriever = AddressRetriever(HttpImpl())
```

- **간단한 방법으로 시스템이 기대하는 방식으로 동작함을 보여 주는 것이 가장 중요하다.**
- 생성자 주입 말고도 다른 많은 방법으로 stub을 주입할 수 있다.
- setter, factory method override, abstract factory 등을 사용하거나 프레임워크 사용

## 10.4 스텁에 지능 더하기: 인자 검증
- retrieve() 메서드의 코드가 올바르게 HttpImpl 코드와 상호 작용하는지 검증해보자
- stub에 Http 클래스의 get() 메서드에 전달되는 URL을 검증하는 보호절을 추가
- 기대하는 인자 문자열을 포함하지 않으면 그 시점에 명시적으로 테스트를 실패 처리한다.
- 일부러 문자열 하나 빼놓고 테스트
- **목은 의도적으로 흉내 낸 동작을 제공하고 수신한 인자가 모두 정상인지 여부를 검증하는 일을 하는 테스트 구조물이다.**

## 10.5 목 도구를 사용하여 테스트 단순화
- stub을 Mock으로 바꿔보자
  - 테스트에 어떤 인자를 기대하는지 명시하기
  - get() 메서드에 넘겨진 인자들을 잡아서 저장하기
  - get() 메서드에 저장된 인자들이 기대하는 인자들인지 테스트가 완료될 때 검증하는 능력 지원하기
- Mockito 사용해보자

- 테스트의 기대 사항 설정은 실제 테스트보다 먼저 해야 한다.
- when(...).thenReturn(...)의 대안으로 처리 과정에서 어떤 메서드가 호출되었는지 검증하는 verify()도 있다.

## 10.6 마지막 하나의 단순화: 주입 도구 소개
- mockito DI 기능 사용하기
1. @Mock 사용하여 Mock 인스턴스 생성하기
2. @InjectMocks을 붙인 대상 인스턴스 변수를 선언한다.
3. 대상 인스턴스를 인스턴스화 한 후에 MockitoAnnotations.initMocks(this) 호출하기

```kotlin
import chapter10.AddressRetriever
import chapter10.util.Http
import org.hamcrest.CoreMatchers.equalTo
import org.hamcrest.MatcherAssert.assertThat
import org.junit.Before
import org.junit.Test
import org.mockito.InjectMocks
import org.mockito.Mock
import org.mockito.Mockito.*
import org.mockito.MockitoAnnotations

class AddressRetrieverTest {

    @Mock private lateinit var http: Http
    @InjectMocks private lateinit var retriever: AddressRetriever

    @Before
    fun createRetriever() {
        retriever = AddressRetriever()
        MockitoAnnotations.openMocks(this)  // 교재 코드에서는 initMocks(this)
    }

    @Test
    fun answersAppropriateAddressForValidCoordinates() {
        // 테스트의 기대 사항들을 설정
        `when`(http.get(contains("lat=38.000000&lon=-104.000000")))
            .thenReturn(    // 기대 사항이 충족되었을 때 처리
            "{\"address\": {" +
                    "\"house_number\":\"324\"," +
                    "\"road\":\"North Tejon Street\"," +
                    "\"city\": \"Colorado Springs\"," +
                    "\"state\": \"Colorado\"," +
                    "\"postcode\": \"80903\"," +
                    "\"country_code\": \"us\"}" +
                    "}"
        )

        val address = retriever.retrieve(38.0, -104.0)

        assertThat(address.houseNumber, equalTo("324"))
        assertThat(address.road, equalTo("North Tejon Street"))
        assertThat(address.city, equalTo("Colorado Springs"))
        assertThat(address.state, equalTo("Colorado"))
        assertThat(address.zip, equalTo("80903"))
    }
}
```

- Http와 retriever 인스턴스를 DI를 통해 얻었다.
- 더이상 생성자에 파라미터도 없게 되었다.

## 10.7 목을 올바르게 사용할 때 중요한 것
- mock을 사용한 테스트는 진행하길 원하는 내용을 분명하게 기술해야 한다.
- **연관성**
- 테스트를 보는 사람이 코드를 깊게 파지 않아도 이러한 연관성을 쉽게 파악할 수 있도록 코드는 더 좋아진다. -> 결국 클린코드
- mock이 실제 동작을 대신한다는 것을 잊지 말라
  - mock이 프로덕션 코드의 동작을 올바르게 묘사하고 있는가?
  - 프로덕션 코드는 생각하지 못한 다른 형식으로 반환하는가?
  - 프로덕션 코드는 예외를 던지는가? null을 반환하는가?
- 프로덕션 코드를 직접 테스트하고 있지 않다는 걸 기억하자

## 10.8 마치며
- **테스트는 라이브 서비스, 파일, 데이터베이스, 다른 번거로운 의존성들과 상호 작용할 필요가 없다!**
- 적절한 도구를 활용해 목을 생성하고 주입하는 노력을 최소화할 수도 있다.
