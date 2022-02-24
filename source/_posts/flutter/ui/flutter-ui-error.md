---
layout: post
title: "Flutter 위젯 에러"
categories: flutter-ui
tags: [dev, flutter]
toc: true
---

- 에러 상황이야 엄청 다양하게 있겠지만, 기존 Native에서 UI를 그리는 방식이랑 많이 달라서 겪었던 상황들을 정리해보았습니다.
- Flutter에서 대표적으로 당황하게 되는 에러화면은 2가지가 있습니다.
  - 에러 메시지와 함께 화면 전체가 빨간 배경으로 덮이는 현상 발생
  - 화면에서 Overflow 된 모서리에 사선으로 노란색/검정색이 그어진 그림 발생
- 관련 문서
  - https://docs.flutter.dev/testing/common-errors
  - https://docs.flutter.dev/testing/errors

## red screen of death
- 관련 포스트: https://medium.com/nonstopio/flutter-kill-the-red-screen-of-death-f5e0601d1cdc
- UI를 런타임에서 그리다가 생긴 에러로 인해 화면 전체가 빨간색으로 덮일 수 있습니다. (아예 화면에 그릴 수 없는 치명적 상태)
- 에러를 해결하는 게 먼저이고, 빨간 화면 대신 다른 방식으로 표현하도록 Error Handling도 가능합니다. [링크](https://stackoverflow.com/questions/53903928/disable-flutters-red-screen-of-death)

## RenderFlex Overflow 에러
- 제한된 영역보다 크게 위젯을 그리려고 할 때 주로 나타납니다.

![예시](https://i.stack.imgur.com/0slE0.png)

- [그림 출처](https://stackoverflow.com/questions/54178816/row-renderflex-overflowed-by-76-pixels-on-the-right)
- 화면 상에서 에러가 발생한 상하좌우 방향의 모서리에 따라 몇 pixel이 넘쳤는지 에러가 표시됩니다.
- 해당 에러를 발생시키지 않기 위한 방법은 매우 많겠지만 생각나는 것들을 나열해보겠습니다.

### Text
- Text 위젯을 사용하면서 길이가 가로나 세로로 계속 길어지면서 overflow 발생
  - **해결:** maxLines 값을 정하거나 일부만 보여주기 위해 TextStyle 내 overflow 필드를 이용하는 방법이 있습니다.
- TextField 위젯과 같이 키보드가 아래로 올라오며 화면에 표현될 때, 예상치 않게 구현한 UI가 화면 밖 위로 당겨지면서 bottom overflow
  - **해결:** 구현되어 있는 화면의 위젯에 따라 다양하겠지만, 보통의 경우는 움직이게 되는 위젯을 ListView 안에 넣어서 스크롤 될 수 있게 하여 에러를 방지했습니다.

### 화면 밖으로 나가는 위젯
- 화면 사이즈보다 가로나 세로 길이가 긴 위젯이 그려지면 overflow 발생

> 물론 화면 사이즈보다 작게 그리면 해결됩니다.
하지만 반응형으로 UI를 그리는 상황이 많기 때문에 제한된 크기의 위젯을 사용하기에도 애매하고,
크기를 제한하지 않았다가 overflow가 발생할 수도 있습니다.

- **해결:** Padding 위젯 안에 크기가 동적인 위젯을 넣으면 overflow를 방지하는데 유용합니다.

## setState 에러
- 주로 화면 시작될 때 UI 업데이트를 시도하다가 많이 발생합니다.
- 자체 UI 업데이트가 가능한 **StatefulWidget**의 경우에서 예시를 들겠습니다.
- 위젯이 시작하는 StatefulWidget 내 State 클래스에서 **initState()** 메서드를 오버라이드해서 코드를 작성하게 됩니다.
- 아직 위젯이 그려지기 전 상황이기 때문에 setState() 메서드를 사용하게 되면 에러가 발생합니다.
- **해결:** 일반적인 Flutter 개발자들의 편법과도 같은 해결방법은 initState 내 동작할 UI 업데이트 코드를 비동기로 실행합니다.

```dart
  @override
  void initState() {
    Future.microtask(() {
      // 코드 작성
      // UI 업데이트
      setState(() {});
    });
    super.initState();
  }
```

- 제가 선호하는 방법은 Future를 이용한 방식입니다. 상황에 따라서 딜레이 시간을 줄 수도 있어서 좋은 것 같습니다.
- build() 메서드가 실행되는 도중에 setState() 메서드가 실행되어서 build() 메서드가 또다시 실행되지 않게 하는 것이 핵심입니다.
- 

### 주의할 점
- 예를 들어 서버에서 초기값을 가져와 UI에 바로 보여줘야하는 경우에 값을 가져오지 않은 초기 화면이 나왔다가 이후에 UI가 업데이트 되는 상황이 나올 수 있습니다.
- 당연한 얘기이지만, 데이터가 없을 때의 UI를 잘 고려해서 기본값을 설정해주어야 자연스럽습니다.
- 초기값을 가져오기 전에는 나중에 그릴 UI를 아예 build() 메서드 내에서 런타임으로 실행되지 않도록 **분기를 하는 방법**도 있습니다.

