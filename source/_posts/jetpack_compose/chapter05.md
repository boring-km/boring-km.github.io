---
layout: post
title: "5장 Composable 함수 상태 관리"
categories: jetpack-compose-study
date: 2023-02-27
tags: [dev, android]
toc: true
---

[Source Code](https://github.com/Just-Android-Compose-Study/JetpackComposeStudy/tree/master/Chapter05)

## 상태를 갖거나 갖지 않는 Composable 함수 이해

**UI는 항상 현재 데이터를 보여줘야만 한다는 것이 중요하다. 따라서 값이 변경되면 반드시 UI에 알려야 한다.**
이를 위해 ```observable``` 타입을 사용한다.

```kotlin
// kotlin에서 사용되는 예시
var counter by observable(-1) { _, oldValue, newValue ->
	println("$oldValue -> $newValue")
}

for (i in 0..3) counter = i
```

Jetpack Compose에서는 이러한 콜백 함수 없이도 상태가 변경되면 자동으로 관련된 UI 요소를 재구성하도록 동작한다.

### Composable 함수에서 상태 사용

stateful: Composable 함수가 값을 유지(remember)하고 있으면 stateful 함수다

```kotlin
@Composable
@Preview
fun SimpleStateDemo1() {
    val num = remember { mutableStateOf(Random.nextInt(0, 10)) }
    Text(text = "값: ${num.value}")

    LaunchedEffect(true) {
        delay(3000)
        num.value = 9999
    }
}

@Composable
@Preview
fun SimpleStateDemo2() {
    val num by remember { mutableStateOf(Random.nextInt(0, 10)) }
    // num 값을 직접 변경할 수는 없어졌다.
    // num 자체에서는 state 가지지 않음
    Text(text = num.toString())
}
```

```kotlin
// caculation은 기억할 값을 생성하는 lambda 표현식
// 구성되는 동안 단 한 번만 평가되고 다시는 평가되지 않는다.
@Composable
inline fun <T> remember(crossinline calculation: @DisallowComposableCalls () -> T): T =
    currentComposer.cache(false, calculation)

@Composable
inline fun <T> remember(
    key1: Any?,
    crossinline calculation: @DisallowComposableCalls () -> T
): T {
	// key1으로 들어오는 값이 변경되었는지 체크
    return currentComposer.cache(currentComposer.changed(key1), calculation)
}
```

- changed 내용

```kotlin
/**
 * A Compose compiler plugin API. DO NOT call directly.
 *
 * Check [value] is different than the value used in the previous composition. This is used,
 * for example, to check parameter values to determine if they have changed.
 *
 * @param value the value to check
 * @return `true` if the value if [equals] of the previous value returns `false` when passed
 * [value].
 */
@ComposeCompilerApi
fun changed(value: Any?): Boolean
```

- Composer.cache()

```kotlin
// invalid 값이 위에 있는 check에서 값이 변경되었을 때 true로 들어오면서 값이 변경된다. (block 람다식 재실행)
@ComposeCompilerApi
inline fun <T> Composer.cache(invalid: Boolean, block: @DisallowComposableCalls () -> T): T {
    @Suppress("UNCHECKED_CAST")
    return rememberedValue().let {
        if (invalid || it === Composer.Empty) {
            val value = block()
            updateRememberedValue(value)
            value
        } else it
    } as T
}
```

```kotlin
@Composable
fun RememberWithKeyDemo() {
    var key by remember { mutableStateOf(false) }
    // key의 상태를 지켜보는 date, 변경시 람다식이 재실행 될 것임
    val date by remember(key) { mutableStateOf(Date()) }
    Column(
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center,
        modifier = Modifier.fillMaxSize()
    ) {
        Text(text = date.toString())
        /*
        remember로 설정한 값이 이전 구성과 같다면 재평가되지 않고,
        동일하지 않은 경우에 새로운 값으로 계산하고, 이 값을 기억하고 반환한다.
        */
        Button(onClick = { key = !key }) {
            Text(text = stringResource(id = R.string.click))
        }
    }
}
```

### 상태를 갖지 않는 Composable 함수 작성

```kotlin
@Composable
fun SimpleStatelessComposable2(text: State<String>) {
    Text(text = text.value)
}
```

이 함수는 파라미터로 상태를 받지만 저장하지 않고 다른 상태를 기억하지도 않는다.
**멱등성(idempotent)**: 연산을 여러 번 적용하더라도 결과가 달라지지 않는 성질을 의미

#### Composable 함수의 준수사항
- 빠름: Composable은 무거운 연산을 하지 말아야 한다. 웹 서비스나 어떠한 I/O도 호출해서는 안 된다. Composable에서 사용하는 데이터는 전달받는 형식이 돼야 한다.

- 부수 효과에서 자유로움: 전역 프로퍼티를 수정하거나 의도치 않은 observable 효과를 생산하지 말아야 한다.

- 멱등성: ```remember { }```를 사용하지 않고(1) 전역 프로퍼티에도 접근하지 않으며(2) **예측 불가능한 코드를 호출하지 말아야 한다.(3)**

#### 상태를 갖는 Composable, 상태를 갖지 않는 Composable

**상태를 갖는 Composable에서 상태를 갖지 않는 Composable을 호출하자.**

```kotlin
// stateless
@ExperimentalMaterial3Api
@Composable
fun TextFieldDemo(state: MutableState<TextFieldValue>) {
    TextField(
        value = state.value,
        onValueChange = { state.value = it },
        placeholder = { Text("Hello") },
        modifier = Modifier.fillMaxWidth()
    )
}

// stateful
@ExperimentalMaterial3Api
@Composable
@Preview
fun TextFieldDemo() {
    val state = remember {
        mutableStateOf(TextFieldValue(""))
    }
    TextFieldDemo(state)
}
```

## 상태 호이스팅과 이벤트 전달


> - 상태를 갖지 않는 Composable로 만들기 위해 상태를 상위로 이동시키는 패턴이다.
> - Composable을 좀 더 재사용하기 쉽고 테스트에 용이하게 하는 것 외에도 하나 이상의 Composable 함수에서 상태를 사용하려면 상태를 상위로 올리게 할 필요가 있다.

### 섭씨/화씨 변경 예제

```kotlin
@Composable
fun TemperatureTextField(
    temperature: MutableState<String>,  // 파라미터로 상태를 전달 받으며
    modifier: Modifier = Modifier,
    callback: () -> Unit
) {
    TextField(
        value = temperature.value,
        onValueChange = {
            temperature.value = it  // 변경 사항을 상태에 다시 저장한다.
        },
        placeholder = {
            Text(text = stringResource(id = R.string.placeholder))
        },
        modifier = modifier,
        keyboardActions = KeyboardActions(onAny = {
            callback()
        }),
        keyboardOptions = KeyboardOptions(
            keyboardType = KeyboardType.Number,
            imeAction = ImeAction.Done
        ),
        singleLine = true
    )
}

@Composable
fun TemperatureRadioButton(
    selected: Boolean,
    resId: Int,
    onClick: (Int) -> Unit, // 파라미터를 받는 콜백함수를 파라미터로 설정
    modifier: Modifier = Modifier
) {
    Row(
        verticalAlignment = Alignment.CenterVertically,
        modifier = modifier
    ) {
        RadioButton(
            selected = selected,
            onClick = {
                onClick(resId)  // onClick 콜백함수 호출
            }
        )
        Text(
            text = stringResource(resId),
            modifier = Modifier
                .padding(start = 8.dp)
        )
    }
}

@Composable
fun TemperatureScaleButtonGroup(
    selected: MutableState<Int>,    // 상태를 받음
    modifier: Modifier = Modifier
) {
    val sel = selected.value
    val onClick = { resId: Int -> selected.value = resId }  // 3: 새로운 상태 값으로 지정한다.
    Row(modifier = modifier) {
        TemperatureRadioButton(
            selected = sel == R.string.celsius,
            resId = R.string.celsius,   // 2: redId 값을
            onClick = onClick   // 1: 라디오 버튼을 클릭하면, (버블업, bubble up)
        )
        TemperatureRadioButton(
            selected = sel == R.string.fahrenheit,
            resId = R.string.fahrenheit,
            onClick = onClick,
            modifier = Modifier.padding(start = 16.dp)
        )
    }
}

// Convert
// 부모(stateful)에서 상태를 생성하고,
// 자식(stateless)은 부모의 상태를 받아서만 동작하는 수동적 Composable 함수다. 
@Composable
@Preview
fun FlowOfEventsDemo() {
    val strCelsius = stringResource(id = R.string.celsius)
    val strFahrenheit = stringResource(id = R.string.fahrenheit)
    val temperature = remember { mutableStateOf("") }
    val scale = remember { mutableStateOf(R.string.celsius) }
    var convertedTemperature by remember { mutableStateOf(Float.NaN) }
    val calc = {
        val temp = temperature.value.toFloat()
        convertedTemperature = if (scale.value == R.string.celsius)
            (temp * 1.8F) + 32F // 섭씨 * 1.8 + 32
        else
            (temp - 32F) / 1.8F // (화씨 - 32) / 1.8
    }
    // 섭씨면 화씨로, 화씨면 섭씨로 표현
    val result = remember(convertedTemperature) {
        if (convertedTemperature.isNaN())
            ""
        else
            "${convertedTemperature}${
                if (scale.value == R.string.celsius)
                    strFahrenheit
                else strCelsius
            }"
    }
    val enabled = temperature.value.isNotBlank()    //  비어 있지 않으면 모두 활성화
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        TemperatureTextField(
            temperature = temperature,
            modifier = Modifier.padding(bottom = 16.dp),
            callback = calc     // 키보드 액션에서 완료 눌렀을 때
        )
        TemperatureScaleButtonGroup(
            selected = scale,
            modifier = Modifier.padding(bottom = 16.dp)
        )
        Button(
            onClick = calc,     // 직접 변환
            enabled = enabled
        ) {
            Text(text = stringResource(id = R.string.convert))
        }
        if (result.isNotEmpty()) {  // 결과가 있을 때만 보이는 Text
            Text(
                text = result,
                style = MaterialTheme.typography.bodyMedium
            )
        }
    }
}
```

- 전환 후 표시되는 텍스트는 기억되었다가 result에 할당된다.
- convertedTemperature가 변경되면 result는 재평가된다. (정말?)

```kotlin
// ...
	// 섭씨면 화씨로, 화씨면 섭씨로 표현
    val result = remember(convertedTemperature) {
    	// 여기서 출력을 찍어봤다.
        println("converted?: $convertedTemperature")
        if (convertedTemperature.isNaN())
            ""
        else
            "${convertedTemperature}${
                if (scale.value == R.string.celsius)
                    strFahrenheit
                else strCelsius
            }"
    }
// ...
```

![](/images/jetpack_compose/chapter05/log.png)

예상대로 remember 대상 상태값인 convertedTemperature 바뀌지 않으면 재평가(Recomposition/재구성)되지 않음을 알 수 있었다.

## 환경설정 변경에도 데이터 유지

Jetpack Compose는 임시로 상태를 저장하기 위해 ```rememberSaveable { }``` 사용한다.

### ViewModel 사용해보기

```kotlin
class MyViewModel : ViewModel() {

    // observable 변수
    private val _text: MutableLiveData<String> =
        MutableLiveData<String>("Hello #3")

    val text: LiveData<String>
        get() = _text

    fun setText(value: String) {
        _text.value = value
    }
}

class ViewModelDemoActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            ViewModelDemo()
        }
    }
}

@Composable
@Preview
fun ViewModelDemo() {
    val viewModel: MyViewModel = viewModel()
    // 상태를 임시로 기억함
    val state1 = remember {
        mutableStateOf("Hello #1")
    }
    /*
    It behaves similarly to remember,
     but the stored value will survive the activity or process recreation
      using the saved instance state mechanism
     (for example it happens when the screen is rotated in the Android application).
     */
    val state2 = rememberSaveable {
        mutableStateOf("Hello #2")
    }

    val testState = rememberSaveable(saver = Saver(
        save = {
            println("save")
            mutableStateOf("Hello #tester2")
        },
        restore = {
            println("restore")
            mutableStateOf("Restored")
        }
    )) {
        println("initial")
        mutableStateOf("initial")
    }


    val state3 = viewModel.text.observeAsState()    // 변경 가능한 상태를 리턴함, Nullable State를 리턴함
    state3.value?.let {
        Column(modifier = Modifier.fillMaxWidth()) {
            val context = LocalContext.current as Activity

            MyTextField(state1) { state1.value = it }
            MyTextField(state2) { state2.value = it }
            MyTextField(state3) {
                viewModel.setText(it)
            }
            MyTextField(testState) {
                testState.value = it
            }
            Button(onClick = {
                context.startActivity(Intent(context, ViewModelDemoActivity::class.java))
                context.finish()

            }) {
                Text(text = "재시작")
            }
        }
    }
}

@Composable
fun MyTextField(
    value: State<String?>,  // Nullable State
    onValueChange: (String) -> Unit
) {
    value.value?.let {
        TextField(
            value = it,
            onValueChange = onValueChange,
            modifier = Modifier.fillMaxWidth()
        )
    }
}
```

```rememberSaveable { }``` 으로도 간단하게 상태를 저장하고 불러올 수 있겠지만, 하나의 화면 안에서 모든 로직이 끝나는 것이 아니라 데이터를 외부에서 의존적인 형태로 받아올 때는 Saver 구현체나 ViewModel 클래스를 사용하는 것이 좋겠다.

## 요약
- 상태 호이스팅은 상태를 갖지 않는 Composable 함수를 만들기 위한 일종의 도구다.
- ```remember { }```, ```rememberSaveable { }```, ```ViewModel```에 대해 알아보았다.
