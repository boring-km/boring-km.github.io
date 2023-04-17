---
layout: post
title: "8장 애니메이션 적용"
categories: jetpack-compose-study
date: 2023-03-19
tags: [dev, android]
toc: true
---

[Source Code](https://github.com/Just-Android-Compose-Study/JetpackComposeStudy/tree/master/Chapter08)

## 애니메이션을 사용한 상태 변화 시각화

버튼을 클릭하면 상태를 변화시켜 박스의 색상을 빨간색과 파란색으로 전환하는 예제

```kotlin
@Preview
@Composable
fun StateChangeDemo() {
    var toggled by remember {
        mutableStateOf(false)
    }
    val color = if (toggled)
        Color.Blue
    else
        Color.Red
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Button(onClick = {
            toggled = !toggled
        }) {
            Text(
                stringResource(R.string.toggle)
            )
        }
        Box(
            modifier = Modifier
                .padding(top = 32.dp)
                .background(color = color)
                .size(128.dp)
        )
    }
}
```

### 한 가지 값을 변경하는 애니메이션

```kotlin
val color by animateColorAsState(
    targetValue = if (toggled)
        Color.Blue
    else
        Color.Red,
    animationSpec = tween(durationMillis = 500)
)
```

- ```animateColorAsState()```을 통해 색 변경 부분을 애니메이션으로 부드럽게 전환한다.
- tween 애니메이션을 사용해봤다.

### 여러 값을 변경하는 애니메이션

상태가 변경됐을 때 한번에 여러 값에 애니메이션 적용하기

```kotlin
@Composable
@Preview
fun MultipleValuesAnimationDemo() {
    var toggled by remember {
        mutableStateOf(false)
    }
    val transition = updateTransition(
        targetState = toggled,  // 이 상태값이 변하면 transition 작동
        label = "toggledTransition"
    )
    val borderWidth by transition.animateDp(label = "borderWidthTransition") { state ->
        if (state)
            10.dp
        else
            1.dp
    }

    // 상태 값에 따라 animate
    val degrees by transition.animateFloat(label = "degreesTransition") { state ->
        if (state) -360F
        else
            0F
    }
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Button(onClick = {
            toggled = !toggled  // 상태 변경
        }) {
            Text(
                stringResource(R.string.toggle)
            )
        }
        Box(
            contentAlignment = Alignment.Center,
            modifier = Modifier
                .padding(top = 32.dp)
                .border(
                    width = borderWidth,
                    color = Color.Black
                )
                .size(128.dp)
        ) {
            Text(
                text = stringResource(id = R.string.app_name),
                modifier = Modifier.rotate(degrees = degrees)
            )
        }
    }
}
```

## 애니메이션을 사용해 UI 요소를 노출하거나 숨기기

- 필요한 데이터만 보여주고 가리기

### AnimatedVisibility()의 이해

- 좌측에서 ```slideInHorizontally()```으로 부드럽게 이동하면서 나오다가 ```fadeOut()```으로 투명도 올리면서 사라지기

```kotlin
@Composable
@Preview
fun AnimatedVisibilityDemo() {
    var visible by remember {
        mutableStateOf(false)
    }
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Button(onClick = {
            visible = !visible
        }) {
            Text(
                stringResource(
                    id = if (visible)
                        R.string.hide
                    else
                        R.string.show
                )
            )
        }
        AnimatedVisibility(
            visible = visible,
            enter = slideInHorizontally(initialOffsetX = { -it }),
            exit = fadeOut(animationSpec = tween(durationMillis = 300))
        ) {
            Box(
                modifier = Modifier
                    .padding(top = 32.dp)
                    .background(color = Color.Red)
                    .size(128.dp)
            )
        }
    }
}
```

### 크기 변경 애니메이션

- ```Slider()``` 값에 따라 ```Text()```의 maxLine 값과 fontSize가 동적으로 변하는 예제 (교재보다 좀더 부드럽게 해봄)

```kotlin
@Preview
@Composable
fun SizeChangeAnimationDemo() {
    var size by remember { mutableStateOf(1F) }

    val transition = updateTransition(
        targetState = size,  // 이 상태값이 변하면 transition 작동
        label = "sizeTransition"
    )

    val fontSize by transition.animateFloat(label = "text") { state ->
        state
    }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        Slider(
            value = size,
            valueRange = (1F..4F),
            steps = 3,
            onValueChange = {
                size = it
            },
            modifier = Modifier.padding(bottom = 8.dp)
        )
        Text(
            text = stringResource(id = R.string.lines),
            modifier = Modifier
                .fillMaxWidth()
                .background(Color.White)
                .animateContentSize(tween(durationMillis = 300)),
            maxLines = fontSize.toInt(), color = Color.Blue
        )
        Text(
            text = stringResource(id = R.string.app_name),
            fontSize = (fontSize * 8).sp
        )
    }
}
```

## 시각 효과를 통한 트랜지션 향상

- UI 일부를 전환하고 싶을 때는 ```Crossfade()```를 사용하자

### Crossfade Composable function

```kotlin
@Preview
@Composable
fun CrossfadeAnimationDemo() {
    var isFirstScreen by remember { mutableStateOf(true) }
    Column(
        modifier = Modifier
            .fillMaxSize(),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Switch(
            checked = isFirstScreen,
            onCheckedChange = {
                isFirstScreen = !isFirstScreen
            },
            modifier = Modifier.padding(top = 16.dp, bottom = 16.dp)
        )
        Crossfade(targetState = isFirstScreen, animationSpec = spring(stiffness = Spring.StiffnessVeryLow)) {
            if (it) {
                Screen(
                    text = stringResource(id = R.string.letter_w),
                    backgroundColor = Color.Gray
                )
            } else {
                Screen(
                    text = stringResource(id = R.string.letter_i),
                    backgroundColor = Color.LightGray
                )
            }
        }
    }
}

@Composable
fun Screen(
    text: String,
    backgroundColor: Color = Color.White
) {
    Box(
        modifier = Modifier
            .fillMaxSize()
            .background(color = backgroundColor),
        contentAlignment = Alignment.Center
    ) {
        Text(
            text = text,
            style = MaterialTheme.typography.displayLarge
        )
    }
}
```

교재에서는 ```Crossfade()``` 내부에서 사용된 ```tween()``` 애니메이션에 대해 설명해주고 있지만 패스


### AnimationSpec 이해

- 애니메이션 사양을 정의하기 위한 기본 인터페이스
- 애니메이션을 수행할 데이터 타입과 애니메이션 환경설정을 저장한다.
- 애니메이션 시스템은 AnimatorVector 인스턴스에서 동작한다.
- 여러가지 AnimationSpec의 확장 인터페이스들이 있으니 필요에 따라 사용하면 되겠다.
- 무한으로 실행되는 애니메이션 보여주고 끝내기

```kotlin
@Composable
@Preview
fun InfiniteRepeatableDemo() {
    val infiniteTransition = rememberInfiniteTransition()
    val degrees by infiniteTransition.animateFloat(
        initialValue = 0F,
        targetValue = 0F,
        animationSpec = infiniteRepeatable(animation = keyframes {
            durationMillis = 3000
            0F at 0
            180F at 750
            359F at 750 + 750
            180F at 750 + 750 + 750
            0F at 750 + 750 + 750 + 750
        })
    )
    Box(
        modifier = Modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        Text(
            text = stringResource(id = R.string.app_name),
            modifier = Modifier.rotate(degrees = degrees),
            fontSize = (degrees / 6).sp
        )
    }
}
```
