---
layout: post
title: "2장 선언적 패러다임 이해"
categories: jetpack-compose-study
date: 2023-02-08
tags: [dev, android]
toc: true
---

[Source Code](https://github.com/Just-Android-Compose-Study/JetpackComposeStudy/tree/master/Chapter02)

## 안드로이드 뷰 시스템 살펴보기 (xml 방식)

- layout 파일에서 xml 계층 구조를 통해 기본 UI를 작성해왔다.
- ...Layout 태그로 감싸게 되면 자식 노드의 크기와 위치를 지정해야 하는 책임이 생긴다.
- layout과 non-layout 요소를 통틀어서 컴포넌트라고 한다.

### layout file inflating

- layout 파일의 id를 직접 참조하여 UI요소를 정의하는 방식 (*findViewById*)
- ViewBinding: 컴포넌트의 참조를 계속 유지하지 않아도 binding 변수에 참조를 유지시켜 사용하는 방식

> UI를 XML 파일로 정의
>
> UI를 런타임 단계에서 컴포넌트 트리로 인플레이트된다.
>
> UI를 변경하려면 연관된 모든 컴포넌트의 속성을 수정해야만 한다.
>
> UI 요소가 화면에서 보이지 않더라도 컴포넌트 트리의 요소로 남아있다.

- 위의 내용을 이유로 xml 방식의 일반적 UI 프레임워크를 명령적(*imperative*)인 방법이라고 한다.
- 앱에 UI 요소가 많아질수록 변경 사항을 추적하는 부담도 점점 커진다.
- 개발자는 도메인 데이터의 변경이 일어나면, 도메인 데이터의 어느 부분이 어떤 UI 요소와 관련이 있는지 알아야 하며, 컴포넌트 트리를 적절히 수정해야만 한다.

## 컴포넌트에서 컴포저블 함수로 이동

- 컴포넌트는 메시지를 주고받는 방식을 사용해 시스템의 다른 부분과 통신한다.
- 컴포넌트의 모습이나 행위는 일련의 속성이나 프로퍼티로 제어한다.
- layout 파일은 xml 문법을 사용해 Java/Kotlin 파일 외부에 있는 컴포넌트 트리를 서술한다. (현재 상태와 무관하게 UI를 정의함)
- android.view.View가 최상의 Android UI 요소이고, 여기서부터 상속하여 다양한 UI 클래스가 만들어졌다.

### 컴포넌트 계층 구조의 한계

> **이미지와 텍스트를 모두 보여주는 버튼을 만드는 케이스**

- Java의 단일 상속 기반 속성을 가지고 있기 때문이다.
- 하나 이상의 컴포넌트의 개별 기능을 조합하는 것이 불가능 -> 개별 기능을 분리할 수 없기 때문에, 재활용이 컴포넌트 단계에서 발생하기 때문
- 결론적으로 UI를 확장시켜 개발하는 데에 한계가 있다.

### 함수를 통한 UI 구성

```kotlin
@Composable
fun Factorial() {
    var expanded by remember { mutableStateOf(false) }  // 상태 값 -> Composable 함수에서 parameter로 사용되면 이 값이 변경될 때 마다 Recomposition이 발생한다.
    var text by remember { mutableStateOf(factorialAsString(0)) } // 상태 값
    Box(
        modifier = Modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        Text(
            modifier = Modifier.clickable {
                expanded = true
            },
            text = text,
            style = MaterialTheme.typography.h2
        )
        DropdownMenu(
            expanded = expanded,    // expanded 값에 따라 dropdown이 열리고 닫힌다.
            onDismissRequest = {
                expanded = false
            }) {
            for (n in 0 until 10) {
                DropdownMenuItem(onClick = {
                    expanded = false
                    text = factorialAsString(n)
                }) {
                    Text("${n.toString()}!")
                }
            }
        }
    }
}
```

- Compose UI의 진입점은 Composable 함수다.
- Composable 함수는 주로 content parameter를 전달받는데, 이는 다른 Composable 함수다.
- 호출 순서는 다른 UI 요소와 비교해 UI 요소가 위치할 곳을 제어한다.
- **Android layout 파일은 초기 상태로 컴포넌트 트리를 정의하는 반면 Composable UI는 항상 실제 데이터를 사용해 정의된다.**
- recomposition(재구성): Compose UI 업데이트 과정, Composable 함수가 업데이트 되어야 할 때마다 자동으로 발생한다.
- state: 시간이 지나면서 변경되는 값, **mutableStateOf()**를 사용해 상태를 생성했었다. (1장에서)

## 아키텍처 관점에서 설명

- Composable 함수는 공유되는 일련의 property가 존재하지 않는다.
- @Composable Annotation을 함수에 추가하는 것으로 Jetpack Compose의 특정 부분에서 함수를 인지하게 할 수 있다.

### 클릭 동작에 반응

```kotlin
@Composable
@Preview
fun ButtonDemo() {
    Box {
        Button(
            onClick = { println("clicked") },
            enabled = false
        ) {
            Text(
                "Click me!",
                modifier = Modifier.clickable { // 개별 기능만 추가한 사례
                    println("text clicked")
                }
            )
        }
    }
}
```

### UI 요소 크기 조절과 배치

컴포넌트 중심의 UI 프레임워크에서는 크기와 위치를 화면에 나타내는 프로퍼티가 핵심이다.
...Layout으로 이름이 끝나는 노드들은 자식 컴포넌트의 크기와 위치를 조정하는 능력을 갖는 컨테이너이다.

> 그래서 Compose는?

Row(), Column(), Box() 등을 사용하고, Modifier를 통해 위치 재정의를 한다.

## 요약
- 컴포넌트 중심의 UI 프레임워크의 핵심 요소
    - UI를 런타임 단계에서 컴포넌트 트리로 인플레이트된다.
    - UI를 변경하려면 연관된 모든 컴포넌트의 속성을 수정해야만 한다.
    - UI 요소가 화면에서 보이지 않더라도 컴포넌트 트리의 요소로 남아있다.
- 이 핵심 요소들의 한계점과 이를 Compose가 극복하는 방법
    - Modifier 매커니즘을 사용
