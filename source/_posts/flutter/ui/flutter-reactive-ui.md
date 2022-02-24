---
layout: post
title: "Flutter 반응형 UI"
date: 2022-02-18 16:35:05
categories: flutter-ui
tags: [dev, flutter]
toc: true
---

> 반응형 UI에 대해서 많은 것을 다루기는 어려울 것 같고, 구현하면서 몇가지 신경써줘야 했던 부분 위주로 소개하겠습니다.
> 
> 개발을 하면서 해당 포스트는 여러번 수정이 생길 수도 있을 것 같습니다.

### 위젯의 크기를 값으로 지정하지 않기
- 특정 위젯에 대해서는 고정된 크기의 위젯을 사용해야 할 수도 있겠지만,
  반응형으로 그리고 싶은 위젯은 width, height 값을 정하기 보다는 Padding 위젯을 이용하는게 좋겠습니다.
- 어쩔 수 없이 특정 값을 입력해서 위젯을 그려야겠다면, context 값을 통해 현재 Device의 width, height, devicePixelRatio 값을 얻을 수 있습니다.
  - 문서: https://api.flutter.dev/flutter/widgets/MediaQueryData-class.html

```dart
MediaQueryData queryData = MediaQuery.of(context);
double width = queryData.size.width;
double height = queryData.size.height;
double devicePixelRatio = queryData.devicePixelRatio;
```

- 개발자에 따라서 UI를 그리는 자기만의 노하우가 있을 것 같습니다.
- 저의 경우엔 devicePixelRatio 값을 이용해 이미지가 들어가는 위젯의 크기를 scaling 하기도 했고,
  화면 회전을 할 때 가로화면과 세로화면을 구분하는 방법도 있었으며,
  화면 크기의 몇 퍼센트를 값으로 지정할 것인가를 값으로 주고 싶을 때 쓸 수도 있겠습니다.
- 폰트 크기를 화면 크기에 따라 조절하기도 했었습니다.

### Flexible
- 문서: https://api.flutter.dev/flutter/widgets/Flexible-class.html
- Row, Column, Flex 위젯과 함께 Flexible 위젯의 flex 필드 값을 이용해 내부에 들어갈 위젯의 크기를 비율로 정할 수 있습니다.

### Expanded
- 문서: https://api.flutter.dev/flutter/widgets/Expanded-class.html
- Row, Column, Flex 위젯에서 화면에 가득차지 않아 남아있는 부분을 늘려줄 수 있습니다.
- flex 필드도 사용이 가능하니 Flexible 위젯을 늘리고 싶을 때 사용하면 되겠습니다.
