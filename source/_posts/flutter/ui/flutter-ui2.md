---
layout: post
title: "Flutter의 다양한 Widget(2)"
date: 2022-02-09
categories: flutter-ui
tags: [dev, flutter]
toc: true
---

> 자주 사용하는 기본적인 Widget에 대해서만 소개합니다.
> 
> 자세한 사용 방법을 제시하기 보다는 이 상황에선 이것을 사용했었다는 정도의 설명입니다.

- 텍스트 입력/표현, 버튼
- 위젯 여러 개를 표현하는 방법
- 위젯 겹치기
- 상자 만들기

## 텍스트 입력/표현, 버튼

### Text
- 문서: https://docs.flutter.dev/development/ui/widgets/text
- 문자열을 화면에 보여주는 위젯
- Text 위젯을 꾸미려면 style 필드에 TextStyle 객체를 넣어 원하는 옵션을 추가하면 됩니다.
- main()에서 선언한 MaterialApp 위젯 안에서 textTheme을 설정해놓고 사용하는 것도 가능합니다.
- 만약 폰트를 적용하고 싶다면 프로젝트에 폰트 파일을 추가하고 pubspec.yaml에 사용할 폰트파일을 명시해야 합니다. (자세한 사용방법은 pubspec.yaml 사용 방법에서...)

### TextField
- 문서: https://api.flutter.dev/flutter/material/TextField-class.html
- 문자 입력을 받는 위젯
- controller 필드에 TextEditingController 인스턴스를 넣으면, TextField 안에 입력한 값을 제어할 수 있습니다.

### Buttons
- 문서: https://docs.flutter.dev/release/breaking-changes/buttons
- 3가지 종류의 버튼 위젯: TextButton, ElevatedButton, OutlinedButton
- 상황에 맞는 버튼을 골라 쓰면 되는데, 만약 버튼 위젯으로 구현이 어렵다 싶은 디자인이면 GestureDetector 위젯을 사용합니다. (자세한 사용방법은 'GestureDetector' 에서...)
- 이미지 버튼이 따로 없어서 버튼 위젯 안에다 이미지를 넣게 되면 풀사이즈 이미지 버튼이 나오지 않고 테두리에 버튼 위젯 고유 테두리가 입혀집니다. (만들고 싶으면 위의 방법 사용)
- style 필드에 버튼 디자인을 좀더 자세하게 작성합니다.
- child 필드에 버튼 내용을 위젯으로 작성합니다.

## 위젯 여러 개를 표현하는 방식들
- Android LinearLayout과 거의 비슷한 Row, Column 위젯
- ListView, GridView
- 복수의 위젯을 포함하는 위젯들은 children 필드 내에 리스트 형태로 지정합니다.

### Row, Column
- Row 문서: https://api.flutter.dev/flutter/widgets/Row-class.html
- Column 문서: https://api.flutter.dev/flutter/widgets/Column-class.html
- Row는 가로 방향으로, Column은 세로 방향으로 위젯들을 나열합니다.
- mainAxisAlignment, crossAxisAlignment 필드를 통해 여러 개의 위젯들을 상황마다 정렬이 가능합니다.

### ListView, GridView
- ListView 문서: https://api.flutter.dev/flutter/widgets/ListView-class.html
- GridView 문서: https://api.flutter.dev/flutter/widgets/GridView-class.html
- 수평이나 수직으로 스크롤이 가능하도록 위젯들을 포함합니다.
- ListView/GridView에 담기는 모든 위젯들이 같은 UI 형태를 띄고 있다면 ListView.builder/GridView.builder 위젯으로 사용하는 것이 깔끔합니다.
- ListView 내부에 Card 위젯과 ListTile 위젯을 사용해 기본적인 ListView 위젯의 항목들을 그릴 수도 있습니다. (리스트 항목 내에 그려야 할 요구사항이 많아지면 위젯을 직접 그리는 것이 편할 것입니다.)

## 위젯 겹치기

### Stack
- Stack 문서: https://api.flutter.dev/flutter/widgets/Stack-class.html
- Stack: 위젯을 독립적인 위치에 고정시키거나 어떤 위젯 위에 겹쳐서 위젯을 보이고 싶을 때 사용합니다.
- 원하는 위치에 위젯을 그리기 위해 Align 위젯을 사용해 처음 위치를 지정하고 그리기 시작하면 편리합니다.

### Padding
- Padding 문서: https://api.flutter.dev/flutter/widgets/Padding-class.html
- 어떤 위젯에 padding을 넣을 때 사용합니다.
- 몇몇 위젯의 경우에는 위젯 내에 padding 필드가 존재하기도 합니다.
  (Container 위젯의 경우는 decoration을 추가했을 때 padding 영역까지 decoration이 적용되면서 Padding 위젯을 직접 사용할 때와 결과가 달라집니다.)

## 상자 만들기

### SizedBox
- SizedBox 문서: https://api.flutter.dev/flutter/widgets/SizedBox-class.html
- 위젯의 크기를 특정 크기로 제한할 때 사용합니다.
- 아무것도 표현하지 않는 위젯을 그릴 때는 'const SizedBox.shrink()' 로 보통 사용합니다.

### Container
- Container 문서: https://api.flutter.dev/flutter/widgets/Container-class.html
- 위젯의 크기를 특정 크기로 제한하거나, 특정 영역에 사각형, 원 등의 모양을 낼 때 사용합니다.
- Container 위젯을 꾸밀 때는 **color** 필드와 **decoration** 필드 둘 중 **하나만 사용**합니다.
- decoration 필드에 들어갈 객체는 BoxDecoration 클래스입니다. [문서](https://api.flutter.dev/flutter/painting/BoxDecoration-class.html)
- decoration 값을 통해 사각형과 원 모양을 다양하게 꾸밀 수 있습니다.
- color나 decoration을 사용하지 않은 채 크기만 제한하려면 SizedBox를 사용하는 것이 좋습니다.
