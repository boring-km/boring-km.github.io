---
layout: post
title: "10장 컴포즈 앱 테스트와 디버깅"
categories: jetpack-compose-study
date: 2023-04-03
tags: [dev, android]
toc: true
---

[Source Code](https://github.com/Just-Android-Compose-Study/JetpackComposeStudy/tree/master/Chapter10)

## 환경설정과 테스트 작성

**테스트와 디버깅은 꼭 필요하다.**

중요한 프로그램들이 모두 그래왔듯 결국 내가 개발하는 앱에도 버그가 발생할 것이다.

개발자로서의 삶을 더욱 윤택하게 만드려면 스스로 테스트 코드를 작성하고 자신의 코드나 다른 사람의 코드를 디버깅하는 데 익숙해야만 한다.

> - 유닛 테스트: 비즈니스 로직이 예상대로 동작함을 확인해야만 한다.
> - 통합 테스트: 앱의 모든 구성 요소가 적절히 통합되어 있는지 확인한다. ex) 앱이 하는 일에 따라 원격 서비스에 접근, DB와 연동, 디바이스에 파일을 읽고 쓰는 동작을
    포함할 수 있다.
> - UI 테스트: UI가 정확히 구현되었는지 테스트한다. 지원하는 모든 화면 크기에 대해 모든 UI 요소가 잘 나타나는지, 항상 적절한 값을 보여주는지, 버튼을 클릭하거나
    슬라이더를 이동하는 등의 상호작용이 의도된 함수를 호출시키는지? 앱의 모든 영역이 접근성을 가지는 지 확인해야 한다.

**테스트 피라미드**: Unit Test - 통합 테스트 - UI 테스트로 이어지는 구조

### 유닛 테스트 구현

- Unit은 작고 고립된 코드 조각으로, 프로그래밍 언어에 따라 일반적으로 function, method, sub routine, property가 이에 해당한다.
- Test class는 하나 이상의 테스트를 포함한다.
- 테스트는 잘 정의된 상황이나 조건 또는 기준이 포함되어 있는지를 확인한다.
- 테스트는 격리되어야 한다.
- **테스트는 이전 테스트에 의존해서는 안 된다.**
- **단언문(assertion)**은 예상되는 행위를 나타낸다. 단언문을 충족시키지 못하면 테스트는 실패한다.

### Composable 함수 테스트

```kotlin
@Composable
fun SimpleButtonDemo() {
    val a = stringResource(id = R.string.a)
    val b = stringResource(id = R.string.b)
    var text by remember { mutableStateOf(a) }
    Box(
        modifier = Modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        Button(onClick = {
            text = if (text == a) b else a
        }) {
            Text(text = text)
        }
    }
}
```

- UI 테스트 코드는 아래에서

```kotlin
// SimpleInstrumentedTest.kt
@RunWith(AndroidJUnit4::class)
class SimpleInstrumentedTest {

    @get:Rule
    var name = TestName()   // 테스트 메서드 내부에서 현재 테스트 이름을 제공할 수 있게 해준다.

    /* 
    createComposeRule: ComposeContentTestRule 구현체, AndroidComposeTestRule<ComponentActivity>
    createAndroidComposeRule: ComponentActivity 이외의 액티비티 클래스용 AndroidComposeTestRule을 생성할 수 있게 해준다.
     */
    @get:Rule
    val rule: ComposeContentTestRule = createComposeRule()

    @Before
    fun setup() {
        // 테스트할 Composable 함수 로딩
        // 테스트마다 정확히 한 번만 호출되어야 한다.
        rule.setContent {
            SimpleButtonDemo()
        }
    }

    @Test
    fun testInitialLetterIsA() {
        // onNodeWithText: finder라 불린다. 시맨틱 노드에서 동작한다.
        // 특정 컴포저블이 기대한 대로 나타나거나 동작하는지 테스트하려면 컴포즈 계층 구조의 모든 자식 사이에서 해당 컴포저블을 찾아내야 한다.
        // 여기서 시맨틱 트리가 동작한다. -> UI 계층 구조와 동시에 생성되며 Rule, Text, Action과 같은 속성을 사용해 계층 구조를 설명한다.
        rule.onNodeWithText("A").assertExists()
    }

    @Test
    fun testPrintMethodName() {
        println(name.methodName)    // Logcat에서 testPrintMethodName 확인하면 된다. 
    }
}
```

## 시맨틱 이해

교재 내용에서 제대로 동작하지 않는 예제가 있어 수정했다.

```kotlin
@Test
fun A에서_버튼을_누르면_B로_텍스트가_바뀐다() {
    rule.onNodeWithText("A").performClick()
    rule.onNodeWithText("B").assertExists()
}
```

- ```onNode()```, ```onAllNodes()``` 모두 **finder**로 불리며, 주어진 조건과 일치하는 semantic 노드를 찾아 반환한다.

1. "A"가 적혀있는 텍스트를 찾아 클릭을 한다.
2. "B"가 적혀있는 텍스트가 존재하는지 확인한다.

- 아래는 기존 예제

```kotlin
@Test
fun A에서_버튼을_누르면_B로_텍스트가_바뀐다() {
    rule.onNodeWithText("A")
        .performClick() // performClick() 이후 assert()가 불가능하다.
        .assert(hasText("B"))
}
```

### 시맨틱 노드로 작업

- 시맨틱 노드 로그 출력하기 ```printToLog("테스트명")```
- 이미지 노드 테스트

```kotlin
@Composable
fun ImageDemo() {
    Image(
        painter = painterResource(id = R.drawable.ic_baseline_airport_shuttle_24),
        contentDescription = stringResource(id = R.string.airport_shuttle),
        contentScale = ContentScale.FillBounds,
        modifier = Modifier
            .size(width = 128.dp, height = 128.dp)
            .background(Color.Blue)
    )
}
```

```kotlin
@RunWith(AndroidJUnit4::class)
class ContentDescriptionTest {

    @get:Rule
    val rule = createComposeRule()

    @Test
    fun 공항셔틀을_ContentDescription으로_설정한_이미지의_가로_길이는_128dp다() {
        var contentDescription = ""
        rule.setContent {
            ImageDemo()
            contentDescription = stringResource(id = R.string.airport_shuttle)
        }
        rule.onNodeWithContentDescription(contentDescription)
            .assertWidthIsEqualTo(128.dp)
            .printToLog("")
    }
}
```

### 커스텀 시맨틱 프로퍼티 추가

테스트에서 추가적인 정보를 노출하고 싶은 경우에는 커스텀 시맨틱 프로퍼티를 생성해 제공할 수 있다.

- 요구사항 1: SemanticsPropertyKey를 정의한다.
- 요구사항 2: SemanticsPropertyReceiver를 통해 사용할 수 있게 해준다.

```semantics { }``` 블록 내부에서 타입 안정성을 보장받는 상태로 key-value pair 저장이 가능하다.

```kotlin
// 1. 사전 정의
val BackgroundColorKey = SemanticsPropertyKey<Color>("BackgroundColor")
var SemanticsPropertyReceiver.backgroundColor by BackgroundColorKey

// 2. Composable 작성
@Composable
fun BoxButtonDemo() {
    var color by remember { mutableStateOf(COLOR1) }
    Box(
        modifier = Modifier
            .fillMaxSize()
            .testTag(TAG1)
            .semantics { backgroundColor = color }  // 프로퍼티 설정
            .background(color = color),
        contentAlignment = Alignment.Center
    ) {
        Button(onClick = {
            color = if (color == COLOR1)
                COLOR2
            else
                COLOR1
        }) {
            Text(text = stringResource(id = R.string.toggle))
        }
    }
}
```

```kotlin
// 3. 테스트 코드에서 접근
@RunWith(AndroidJUnit4::class)
class BoxButtonDemoTest {

    @get:Rule
    val rule = createComposeRule()

    @Test
    fun 텍스트박스의_배경색이_COLOR1인_노드가_존재한다() {
        rule.setContent {
            BoxButtonDemo()
        }
        // SemanticsMatcher의 expectValue()와 해당 프로퍼티 key 값을 이용해 값이 동일한지 확인한다.
        rule.onNode(SemanticsMatcher.expectValue(BackgroundColorKey, COLOR1)).assertExists()    // equals가 아닌 일치하는 Node가 있는지 존재 유무를 따짐
    }
}
```

## 컴포즈 앱 디버깅

직접 해보는게 제일 좋을 것 같다.

#### 추가적인 팁
- 커스텀 Modifier를 통한 현재 기본값 출력
- inspectorInfo parameter
- 디버그 인스펙터 정보 확인하기

```kotlin
isDebugInspectorInfoEnabled = true  // InspectableValue.kt 전역변수 설정

Modifier.semantics { backgroundColor = color }.also {
        (it as CombinedModifier).run {
            val inner = this.javaClass.getDeclaredField("inner")
            inner.isAccessible = true
            val value = inner.get(this) as InspectorValueInfo
            value.inspectableElements.forEach { ve: ValueElement ->
                Log.i("ValueElement", "value element: $ve") // 원하는 대로 값을 뽑아 사용 가능
            }
        }
    }
```
그래서 나온 결과는 아래와 같았다.

```text
I/ValueElement: value element: ValueElement(name=mergeDescendants, value=false)
I/ValueElement: value element: ValueElement(name=properties, value=Function1<androidx.compose.ui.semantics.SemanticsPropertyReceiver, kotlin.Unit>)
```

## 요약 및 더 읽을거리

- 기본적 테스트 작성법과 디버깅 방법 안내를 받았다.
- 구글 Android Compose Test 문서 확인하기
- JUnit, Kotest에 대해 더 알아보기
- 테스트 자동화 알아보기