---
layout: post
title: "7장 팁, 트릭, 모범 사례"
categories: jetpack-compose-study
date: 2023-03-13
tags: [dev, android]
toc: true
---

[Source Code](https://github.com/Just-Android-Compose-Study/JetpackComposeStudy/tree/master/Chapter07)

## 상태 유지와 검색

- 상태(state): 시간의 흐름에 따라 변하는 앱 데이터

### ViewModel에 객체 주입

ViewModel은 배후에서 데이터를 어떻게 읽고 쓰는지 관련이 없어야 한다.

```kotlin
class ViewModelFactory(private val repository: Repository) :
    ViewModelProvider.NewInstanceFactory() {
    // ViewModel 인스턴스 생성
    // modelClass 새로 생성할 ViewModel
    override fun <T : ViewModel> create(modelClass: Class<T>): T =
        if (modelClass.isAssignableFrom(TemperatureViewModel::class.java))
            TemperatureViewModel(repository) as T
        else
            DistancesViewModel(repository) as T
}
```

### Factory 사용

- 이전 챕터 6장에서 이미 살펴봤다.

```kotlin
@Composable
fun ComposeUnitConverterNavHost(
    navController: NavHostController, modifier: Modifier
) {
    val context = LocalContext.current
    val factory = ViewModelFactory(Repository(context))
    NavHost(
        navController = navController,
        startDestination = Screens.temperature,
        modifier = modifier
    ) {
        composable(Screens.temperature) {
            TemperatureConverter(
                viewModel = viewModel(factory = factory)
            )
        }
        composable(Screens.distances) {
            DistancesConverter(
                viewModel = viewModel(factory = factory)
            )
        }
    }
}
```

- 생성자를 호출하는 방식으로 Repository 객체를 ViewModel에 주입
- **의존성 주입** 프레임워크에 의존하고 있다면 방식이 많이 바뀐다.

## 컴포저블을 반응성 있게 유지

**컴포저블 함수**의 목적이 UI를 선언하고 사용자 인터랙션을 다루는 것임을 항상 명심해야 한다.
데이터가 ViewModel 내부에서 유지되고 있다면 Composable은 반드시 ViewModel과 상호작용해야 한다.

### ViewModel 인스턴스와 소통

- ViewModel에 있는 데이터는 observable이어야 한다. (LiveData, MutableLiveData)

```kotlin
private val _temperature: MutableLiveData<String> = MutableLiveData(
    repository.getString("temperature", "")
)

val temperature: LiveData<String>
    get() = _temperature

fun setTemperature(value: String) {
    _temperature.value = value
    repository.putString("temperature", value)
}
```

ViewModel 인스턴스는 아래와 같은 방법으로 데이터를 나타낸다.

- public 접근자를 갖는 읽기 전용 property(temperature)
- private 접근자를 갖는 쓰기 가능한 backing variable(_temperature)

ViewModel 사용 방법에 대해서는 6장에서 이미 해봤다.

- ```viewModel.temperature.observeAsState()```
- ViewModel을 전달 받아 상태 값을 구하면 된다.

### 장기간 동작하는 작업 처리

섭씨와 화씨를 전환하는 경우는 빠르게 자주 호출되는 경우이고, 입력값에 따라 점점 더 많은 시간을 소모하는 상황도 있다.

연산에 너무 많은 시간이 소요되어 앱이 응답하지 않는 상황을 막기 위해 **연산**을 **결과를 전달하는 동작**과 분리해야 한다.

1. 결과를 observable property로 제공한다.
2. coroutine, kotlin flow 사용
3. 연산을 끝내면 result property 갱신한다.

코루틴을 사용해 observable property를 바꾸는 예제가 있었다.

```kotlin
fun convert() {
    getDistanceAsFloat().let {
        viewModelScope.launch {
            _convertedDistance.value = if (!it.isNaN())
                if (_unit.value == R.string.meter)
                    it * 0.00062137F
                else
                    it / 0.00062137F
            else
                Float.NaN
        }
    }
}
```

반대로 convertedDistance 상태 값을 가져오고 싶을 땐
```val convertedValue by viewModel.convertedDistance.observeAsState()```

**장시간 동작하는 함수**는 **ViewModel**에서 호출되게 해라 -> 그냥 비동기로 동작하면 다 ViewModel

## 부수 효과의 이해

### suspend 함수 호출

- 6장에서 사용한 코드에서는 이 코루틴 scope 안에서만 동작하는 중단 함수가 있었다.

```kotlin
val snackbarCoroutineScope = rememberCoroutineScope()
snackbarCoroutineScope.launch {
    snackbarHostState.showSnackbar(s)
}
```

### LaunchedEffect(), DisposableEffect()

```kotlin
@Composable
@Preview
fun LaunchedEffectDemo() {
    var clickCount by rememberSaveable { mutableStateOf(0) }
    var counter by rememberSaveable { mutableStateOf(0) }
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Row {
            Button(onClick = {
                // 상태 값 갱신 -> LaunchedEffect 동작
                clickCount += 1
            }) {
                Text(
                    text = if (clickCount == 0)
                        stringResource(id = R.string.start)
                    else
                        stringResource(id = R.string.restart)
                )
            }
            Spacer(modifier = Modifier.width(8.dp))
            Button(enabled = clickCount > 0,
                onClick = {
                    clickCount = 0  // 상태 값 초기화 -> DisposableEffect 동작
                }) {
                Text(text = stringResource(id = R.string.stop))
            }
            // 상태값에 따라 변하는 View가 suspend 함수에 의해 변하고 있다면...!
            if (clickCount > 0) {
                DisposableEffect(clickCount) {  // 키가 변경 되었을 때 함수 실행
                    println("init: clickCount is $clickCount")
                    onDispose {
                        println("dispose: clickCount is $clickCount")
                    }
                }
                LaunchedEffect(clickCount) {
                    counter = 0
                    while (isActive) {
                        counter += 1
                        delay(1000)
                    }
                }
            }
        }
        Text(
            text = "$counter",
            style = MaterialTheme.typography.displaySmall
        )
    }
}
```