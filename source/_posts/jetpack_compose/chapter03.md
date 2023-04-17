---
layout: post
title: "3장 컴포즈 핵심 원칙 자세히 알아보기"
categories: jetpack-compose-study
date: 2023-02-14
tags: [dev, android]
toc: true
---

[Source Code](https://github.com/Just-Android-Compose-Study/JetpackComposeStudy/tree/master/Chapter03)

## 컴포저블 함수 자세히 살펴보기

### 컴포저블 함수의 구성 요소
- 가시성 변경자: 용도에 따라 접근 범위를 정하면 되겠다.
- fun
- 함수명: **파스칼 표기법**으로 작성
- 매개변수: 선택적
- 반환 타입: 선택적 -> 값을 리턴하지 않아도 Composable 함수가 화면에 표현된다.
- 코드 블록

### UI 요소 내보내기
- IDE를 이용해 직접 확인해보기

#### androidx.compose.material.Text()

```kotlin
val textColor = color.takeOrElse {
  style.color.takeOrElse {
    LocalContentColor.current
  }
}
// NOTE(text-perf-review): It might be worthwhile writing a bespoke merge implementation that
// will avoid reallocating if all of the options here are the defaults
val mergedStyle = style.merge(
  TextStyle(
    color = textColor,
    fontSize = fontSize,
    fontWeight = fontWeight,
    textAlign = textAlign,
    lineHeight = lineHeight,
    fontFamily = fontFamily,
    textDecoration = textDecoration,
    fontStyle = fontStyle,
    letterSpacing = letterSpacing
  )
)
BasicText(
  text,
  modifier,
  mergedStyle,
  onTextLayout,
  overflow,
  softWrap,
  maxLines,
)
```

#### BasicText()

> **BasicText**에서 찾은 CoreText 관련 주석
>
> The ID used to identify this CoreText. If this CoreText is removed from the composition tree and then added back, this ID should stay the same.
>
> Notice that we need to update selectable ID when the input text or selectionRegistrar has been updated.
>
> When text is updated, the selection on this CoreText becomes invalid. It can be treated as a brand new CoreText.
>
> When SelectionRegistrar is updated, CoreText have to request a new ID to avoid ID collision.

![CoreText](/images/jetpack_compose/chapter03/CoreText.png)
- https://developer.android.com/jetpack/androidx/releases/compose-foundation#1.0.0-alpha12
- CoreText는 public API에서 사용 불가
- BasicText() 하단부에서 Layout()을 호출하고 있다.

#### Layout()

```kotlin
@Suppress("NOTHING_TO_INLINE")
@Composable
@UiComposable
inline fun Layout(
    modifier: Modifier = Modifier,
    measurePolicy: MeasurePolicy
) {
    val density = LocalDensity.current
    val layoutDirection = LocalLayoutDirection.current
    val viewConfiguration = LocalViewConfiguration.current
    val materialized = currentComposer.materialize(modifier)
    ReusableComposeNode<ComposeUiNode, Applier<Any>>(
        factory = ComposeUiNode.Constructor,
        update = {
            set(measurePolicy, ComposeUiNode.SetMeasurePolicy)
            set(density, ComposeUiNode.SetDensity)
            set(layoutDirection, ComposeUiNode.SetLayoutDirection)
            set(viewConfiguration, ComposeUiNode.SetViewConfiguration)
            set(materialized, ComposeUiNode.SetModifier)
        },
    )
}
```

- ```ReusableComposeNode()```를 호출한다.
- Node라고 하는 UI 요소 계층 구조를 내보낸다.
- 노드는 factory를 통해 생성된다.
- ```update```: 업데이트를 수행하는 코드 작성
- ```skippableUpdate```: 변경자를 조작하는 코드 작성 -> ```materialized```로 바뀜

#### ReusableComposeNode()

```kotlin
/**
 * Emits a recyclable node into the composition of type [T].
 *
 * This function will throw a runtime exception if [E] is not a subtype of the applier of the
 * [currentComposer].
 *
 * @sample androidx.compose.runtime.samples.CustomTreeComposition
 *
 * @param factory A function which will create a new instance of [T]. This function is NOT
 * guaranteed to be called in place.
 * @param update A function to perform updates on the node. This will run every time emit is
 * executed. This function is called in place and will be inlined.
 *
 * @see Updater
 * @see Applier
 * @see Composition
 */
// ComposeNode is a special case of readonly composable and handles creating its own groups, so
// it is okay to use.
@Suppress("NONREADONLY_CALL_IN_READONLY_COMPOSABLE", "UnnecessaryLambdaCreation")
@Composable inline fun <T : Any, reified E : Applier<*>> ReusableComposeNode(
    noinline factory: () -> T,
    update: @DisallowComposableCalls Updater<T>.() -> Unit
) {
    if (currentComposer.applier !is E) invalidApplier()
    currentComposer.startReusableNode()
    if (currentComposer.inserting) {
        currentComposer.createNode { factory() }    // 여기서 노드를 생성한다.
    } else {
        currentComposer.useNode()   // 기존 노드 사용
    }
    currentComposer.disableReusing()
    Updater<T>(currentComposer).update()
    currentComposer.enableReusing()
    currentComposer.endNode()
}
```

- ```ReusableComposeNode```는 **새로운 노드가 생성돼야 할지 또는 기존 노드를 재사용해야 할지를 결정한다.**
- ```currentComposer```는 androidx.compose.runtime.Composables.kt에 있는 최상위 Composer 변수

#### ComposableUiNode

```kotlin
/**
 * Interface extracted from LayoutNode to not mark the whole LayoutNode class as @PublishedApi.
 */
@PublishedApi
internal interface ComposeUiNode {
    var measurePolicy: MeasurePolicy
    var layoutDirection: LayoutDirection
    var density: Density
    var modifier: Modifier
    var viewConfiguration: ViewConfiguration

    /**
     * Object of pre-allocated lambdas used to make use with ComposeNode allocation-less.
     */
    companion object {
        val Constructor: () -> ComposeUiNode = LayoutNode.Constructor
        val VirtualConstructor: () -> ComposeUiNode = { LayoutNode(isVirtual = true) }
        val SetModifier: ComposeUiNode.(Modifier) -> Unit = { this.modifier = it }
        val SetDensity: ComposeUiNode.(Density) -> Unit = { this.density = it }
        val SetMeasurePolicy: ComposeUiNode.(MeasurePolicy) -> Unit =
            { this.measurePolicy = it }
        val SetLayoutDirection: ComposeUiNode.(LayoutDirection) -> Unit =
            { this.layoutDirection = it }
        val SetViewConfiguration: ComposeUiNode.(ViewConfiguration) -> Unit =
            { this.viewConfiguration = it }
    }
}
```

- Modifier
- Density
- MeasurePolicy
- LayoutDirection
- VirtualConstructor (새로 생김)
- ViewConfiguration (새로 생김)

### 값 반환

- Composable 함수의 주목적은 UI 구성이기 때문에 대부분 반환 타입을 명시하지 않는다.
- Composition이나 Recomposition의 일부인 무언가를 반환해야 한다면 반드시 Composable 함수여야 한다.
- 반환된 데이터가 Compose와 아무 관련이 없더 상태를 유지하기 위한 값으로 사용한다면 Composable 함수로 만들어야 한다.
  - **이런 경우에는 함수명을 CamelCase로 작성**

## UI 구성과 재구성

- Jetpack Compose는 앱 데이터가 변경되어 UI가 변경되어야 하는 경우 개발자가 선제적으로 컴포넌트 트리를 변경하는 행위에 의존하지 않는다.
- 대신 이러한 변화를 자체적으로 감지하고 영향을 받는 부분만 갱신한다.

### Composable 함수 간 상태 공유

```kotlin
@Composable
fun ColorPicker(color: MutableState<Color>) {
    val red = color.value.red
    val green = color.value.green
    val blue = color.value.blue
    Column {
        Slider(
            value = red,
            onValueChange = { color.value = Color(it, green, blue) })
        Slider(
            value = green,
            onValueChange = { color.value = Color(red, it, blue) })
        Slider(
            value = blue,
            onValueChange = { color.value = Color(red, green, it) })
    }
}
```

- ```ColorPicker()``` 함수가 ```Color```가 아닌 ```MutableState<Color>``` 를 파라미터로 받는 이유
  - color의 값이 ColorPicker 안에서만 변경될 것이기 때문에 상위 Composable 함수에서 해당 변화에 대해 알 수 있어야 한다.

- 전역 프로퍼티를 사용하는 방식도 있겠지만 이는 권고하지 않는 방식이며, 되도록 Composable 함수의 모습과 행위에 영향을 주는 모든 데이터는 매개변수(parameter)로 전달하는 것이 좋다.

- state hoisting(상태 호이스팅): 상태를 전달받아 Composable 함수를 호출한 곳으로 상태를 옮기는 것

> ### 중요 사항
>
> Composable을 SideEffect가 없게 만들자. (동일한 인자로 함수를 반복적으로 호출해도 항상 동일한 결과를 생산함)
>
> 호출자로부터 모든 관련 데이터를 얻는 것 이외에도 전역프로퍼티에 의존하거나 예측 불가능한 값을 반환하는 함수 호출하는 것도 금지된다. -> 보통 IDE에서 경고해주는 것 같다.

- Compose UI는 Composable 함수의 중첩 호출로 정의된다.
- Composable 함수는 UI 요소 또는 UI 요소 계층 구조를 발행한다.
- UI를 처음 구성하는 것을 Composition이라고 부른다.
- 앱 데이터 변경 시 UI를 재구성하는 것을 Recomposition이라 부른다.
- Recomposition은 자동으로 발생한다.

### 크기 제어

- ```fillMaxSize()```, ```fillMAxWidth()```, ```BoxWidthConstraints()``` 등을 잘 활용해보기

### 액티비티 내에서 Composable 계층 구조 나타내기

> #### setContent
> **parent**: null 값이 가능한 CompositionContext
> **content**: 선언하는 UI를 위한 Composable function

```kotlin
// ComponentActivity.kt
public fun ComponentActivity.setContent(
    parent: CompositionContext? = null,
    content: @Composable () -> Unit
) {
	// 액티비티가 이미 ComposeView의 인스턴스를 포함하는지 알아내기 위해 사용된다.
    val existingComposeView = window.decorView
        .findViewById<ViewGroup>(android.R.id.content)
        .getChildAt(0) as? ComposeView

    if (existingComposeView != null) with(existingComposeView) {
        setParentCompositionContext(parent)
        setContent(content)
    } else ComposeView(this).apply { // findViewById 실패 시 새로운 인스턴스 생성: ComposeView
        // Set content and parent **before** setContentView
        // to have ComposeView create the composition on attach
        setParentCompositionContext(parent)
        setContent(content)
        // Set the view tree owners before setting the content view so that the inflation process
        // and attach listeners will see them already present
        setOwners()
        setContentView(this, DefaultActivityContentLayoutParams)
    }
}
```

```kotlin
// ComposeView.android.kt AbstractComposeView
private var parentContext: CompositionContext? = null
    set(value) {
        if (field !== value) {
            field = value
            if (value != null) {
                cachedViewTreeCompositionContext = null
            }
            val old = composition
            if (old !== null) {
                old.dispose()
                composition = null
                // Recreate the composition now if we are attached.
                if (isAttachedToWindow) {
                    ensureCompositionCreated()
                }
            }
        }
    }
```

```kotlin
// ComposeView.android.kt

// parentContext 대체재 찾기 
private fun resolveParentCompositionContext() = parentContext
    ?: findViewTreeCompositionContext()?.cacheIfAlive()
    ?: cachedViewTreeCompositionContext?.get()?.takeIf { it.isAlive }
    ?: windowRecomposer.cacheIfAlive()

@Suppress("DEPRECATION") // Still using ViewGroup.setContent for now
private fun ensureCompositionCreated() {
    if (composition == null) {
        try {
            creatingComposition = true
            composition = setContent(resolveParentCompositionContext()) {
                Content()
            }
        } finally {
            creatingComposition = false
        }
    }
}
```

## Composable 함수의 행위 수정

- 컴포넌트의 프로퍼티와 달리 Modifier는 전적으로 개발자의 판단에 따라 사용될 수 있다.
- Modifier는 행동이나 정렬, 그리기와 같은 여러 범주 중 하나에 할당될 수 있다. [Modifier 목록 링크](https://developer.android.com/jetpack/compose/modifiers-list)

- Modifier Chaining: 빌더 패턴처럼 Modifier의 속성을 정의

### Modifier 동작 이해

Modifier를 올바르게 사용하지 않으면, IDE에서는 다음과 같은 안내를 해준다.

![Modifier parameter shouble be the first optional parameter](/images/jetpack_compose/chapter03/modifier.png)

어떤 경우에 이런 경고가 나오는 지 예시 코드를 보자

- modifier는 첫 번째로 오는 nullable parameter가 되어야 한다. 말이 어려우니까 예시를 보자

```kotlin
@Composable
fun TextWithWarning1(
    name: String = "Default",
    modifier: Modifier = Modifier,
    callback: () -> Unit
) {
    Text(text = "TextWithWarning1 $name!", modifier = modifier
        .background(Color.Yellow)
        .clickable { callback.invoke() })
}

@Composable
fun TextWithWarning2(test: Modifier = Modifier, name: String = "", callback: () -> Unit) {
    Text(text = "TextWithWarning2 $name!", modifier = test
        .background(Color.Yellow)
        .clickable { callback.invoke() })
}

@Composable
fun TextWithoutWarning(
    modifier: Modifier = Modifier,
    buttonModifier: Modifier,
    name: String = "",
    callback: () -> Unit
) {
    Column(horizontalAlignment = Alignment.CenterHorizontally) {
        Text(text = "TextWithoutWarning $name!", modifier = modifier
            .padding(10.dp) // margin concept
            .background(Color.Yellow)
            .padding(10.dp) // real padding
            .clickable { callback.invoke() })

        val context = LocalContext.current
        Button(
            modifier = buttonModifier.clickable {
                Toast.makeText(context, "버튼에 clickable을 넣으면?", Toast.LENGTH_SHORT).show()
            },
            onClick = { Toast.makeText(context, "버튼 클릭됨", Toast.LENGTH_SHORT).show() }) {
            Text("버튼")
        }
    }
}
```

- 위의 예시 코드에서 보듯이 nullable한 parameter가 먼저 오게 되면, Composable 함수를 사용할 때 굳이 정의하지 않아도 되는 parameter값의 초기화를 강제받게 된다.
- 부모 Composable 함수에서 정의한 Modifier를 자식 Composable 함수에서 추가로 설정하기 위해 Modifier를 반드시 받는 구조이기 때문에 NonNull인 Modifier를 맨 앞으로 가져오는 것이 경제적이라고 보면 되겠다.
- 그리고 잘 보면 변수의 naming도 modifier 이거나 modifier를 사용할 Composable 함수의 이름을 접두사로 사용하면서 camelCase형식으로 적혀있기를 원한다.
- 그리고 Modifier는 **Modifier 요소의 순서가 있고** 한번 정의하면 재정의할 수 없는 **변경 불가능한 컬렉션**이다.

#### Modifier.then()

```kotlin
// Background.kt
fun Modifier.background(
    color: Color,
    shape: Shape = RectangleShape
) = this.then(	// other parameter에 Background 인스턴스가 들어감
    Background(
        color = color,
        shape = shape,
        inspectorInfo = debugInspectorInfo {
            name = "background"
            value = color
            properties["color"] = color
            properties["shape"] = shape
        }
    )
)
```

```kotlin
// Modifier.kt
infix fun then(other: Modifier): Modifier =
    if (other === Modifier) this else CombinedModifier(this, other)
```

- Background 클래스를 내부적으로 사용하고 있고, 최종적으로 Modifier.Element라는 인터페이스를 implement하고 있다.
- 이 Modifier.Element 구현체로 UI요소의 공간에 그림을 그릴 수 있다.

```kotlin
// Background.kt
private class Background constructor(
    private val color: Color? = null,
    private val brush: Brush? = null,
    private val alpha: Float = 1.0f,
    private val shape: Shape,
    inspectorInfo: InspectorInfo.() -> Unit
) : DrawModifier, InspectorValueInfo(inspectorInfo) {
	// ...
}

// DrawModifier.kt
@JvmDefaultWithCompatibility
interface DrawModifier : Modifier.Element {
    fun ContentDrawScope.draw()
}
```

### 커스텀 Modifier 구현

- kotlin 문법을 사용해 쉽게 Extension을 구현할 수 있었다.
- 위에서 봤던 then() 함수에 DrawModifier만 잘 만들면 된다. (마치 View를 확장해서 만들었던 Custom View와 유사해 보인다.)

```kotlin
fun Modifier.drawWhiteCross() = then(
    object : DrawModifier {
        override fun ContentDrawScope.draw() {
            drawLine(
                color = Color.White,
                start = Offset(0F, 0F),
                end = Offset(size.width - 1, size.height - 1),
                strokeWidth = 10F
            )
            drawLine(
                color = Color.White,
                start = Offset(0F, size.height - 1),
                end = Offset(size.width - 1, 0F),
                strokeWidth = 10F
            )
            drawContent()
        }
    }
)

fun Modifier.drawHiddenCross() = then(
    object : DrawModifier {
        override fun ContentDrawScope.draw() {
            drawContent()
            drawBehind {
                drawLine(
                    color = Color.Blue,
                    start = Offset(0F, 0F),
                    end = Offset(size.width - 1, size.height - 1),
                    strokeWidth = 10F
                )
                drawLine(
                    color = Color.Blue,
                    start = Offset(0F, size.height - 1),
                    end = Offset(size.width - 1, 0F),
                    strokeWidth = 10F
                )
            }
        }
    }
)
```

## 요약

- Composable 함수가 어떻게 작성되었고, UI를 그리고 사용되었는지 배웠다.
- Modifier가 무엇이고 어떻게 사용하는 것인지 알아보았다.