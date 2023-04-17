---
layout: post
title: "Flutter의 다양한 Widget(3)"
categories: flutter
tags: [dev, flutter]
toc: true
---

> 자주 사용하는 기본적인 Widget에 대해서만 소개합니다.
>
> 자세한 사용 방법을 제시하기 보다는 이 상황에선 이것을 사용했었다는 정도의 설명입니다.

- GestureDetector
- 다이얼로그

## GestureDetector
- 문서: https://api.flutter.dev/flutter/widgets/GestureDetector-class.html
- 모든 위젯에 제스처(터치) 동작을 추가해주는 위젯
- 보통 터치 영역을 특정 위젯으로 잡아줄 때 많이 사용했습니다. (대표적으로 버튼 동작)
- onTap, onDoubleTap, onLongPress 등 다양한 터치 동작에 대해 인식할 때 동작하는 코드를 작성할 수 있습니다.

## 다이얼로그
- 문서: https://api.flutter.dev/flutter/material/Dialog-class.html
- 팝업 알림창을 띄울 때 대표적으로 사용하는 위젯
- 다이얼로그를 띄우기 위해선 context 인스턴스가 무조건 필요하다.
- [showDialog](https://api.flutter.dev/flutter/material/showDialog.html) 메서드를 기본으로 사용하여 Dialog 위젯을 화면에 띄웁니다.

### 투명 배경 다이얼로그
- 개발하면서 유용하게 사용한 코드를 가져와봤다.

```dart
void showTransparentDialog(BuildContext context, Widget child, {FutureOr Function(Object? value)? onValue}) {
  showGeneralDialog(
    context: context,
    barrierLabel: '',
    barrierDismissible: true,
    barrierColor: Colors.white.withOpacity(0),  // Colors.transparent로 입력하면 완벽한 투명이 되지 않는다.
    pageBuilder: (context, _, __) {
      return Material(
        type: MaterialType.transparency,
        child: child,
      );
    },
  ).then(onValue ?? (_) => {});
}
```

- onValue 함수는 다이얼로그가 닫힌 이후의 동작을 작성하는 콜백함수이다.
