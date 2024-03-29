---
layout: post
title: "Flutter의 다양한 Widget(1)"
date: 2022-02-08
categories: flutter
tags: [dev, flutter]
toc: true
---

> 자주 사용하는 기본적인 Widget에 대해서만 소개합니다.
> 
> 자세한 사용 방법을 제시하기 보다는 이 상황에선 이것을 사용했었다는 정도의 설명입니다.

- MaterialApp
- StatefulWidget, StatelessWidget, Scaffold

## MaterialApp

- 앱이 시작될 때 무조건 사용하는 위젯
- 문서: https://api.flutter.dev/flutter/material/MaterialApp-class.html
- MaterialApp 위젯을 main()에서 호출하여 처음 앱이 실행될 수 있도록 합니다.

```dart
void main() {
  runApp(MaterialApp(
    title: '앱 이름',
    initialRoute: '/',  // 시작할 화면 위젯의 String Path
    routes: {
      HomeScreen.routeName: (context) => HomeScreen(),
      // '/': (context) => const HomeScreen(),
    },
    theme: ThemeData(
      // 다양한 기본 위젯 옵션들을 정의
    ),
    home: HomeScreen(), // initialRoute 대신 클래스 직접 호출하는 방식
  ));
}
```

- routes 영역에 API URI 명세하듯이 여러 화면들을 선언해놓고 사용하는 것이 나중을 위해 좋다고 생각합니다. (의존성 주입 관련)

## StatefulWidget, StatelessWidget, Scaffold
- **화면을 그릴 때 제일 먼저 사용하는 위젯**
- Widget 클래스를 상속하는 추상 클래스 **StatefulWidget**과 **StatelessWidget**이 있습니다. (두 위젯에 대한 비교 내용은 다음 포스팅에서 이어집니다.)

### StatefulWidget
- 문서: https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html
- 문서에도 나와있는 대로 동적으로 변하는 UI를 표현하고 싶을 때 구현하기 좋습니다.
- 회사 프로젝트에서는 동영상 플레이어와 상태에 따라 변하는 이미지 버튼을 구현하기 위해 사용했습니다.
- IntelliJ에서 제공하는 'Flutter Code Template' 에서는 'stful' 이라고 입력하면 기본 코드 형태가 자동완성이 됩니다.

```dart
// TestView 라는 위젯을 만들었을 때 생성된 기본 코드
import 'package:flutter/material.dart'; // 코드 생성 후에 import 에러가 나면 material로 import 해주세요.

class TestView extends StatefulWidget {
  const TestView({Key? key}) : super(key: key);

  @override
  _TestViewState createState() => _TestViewState();
}

class _TestViewState extends State<TestView> {
  @override
  Widget build(BuildContext context) {
    return Container();
  }
}

```

- _TestViewState 클래스의 build() 메서드의 리턴 값에서 UI가 그려집니다.
- _TestViewState 클래스 내에 변수를 선언해놓고 상태를 변경하고 싶을 때 아래의 코드를 호출합니다.

```dart
setState(() {

});
```

- _TestViewState에서 어떤 위젯과 상호작용을 하면서 TestView 위젯의 UI를 업데이트 할 때 사용됩니다.

### StatelessWidget
- 문서: https://api.flutter.dev/flutter/widgets/StatelessWidget-class.html
- StatefulWidget으로 모든 화면을 구현할 수 있지만 어떤 상호작용으로 인해 UI가 변화하는 위젯이 아닐 때 사용합니다.

```dart
import 'package:flutter/material.dart'; // 코드 생성 후에 import 에러가 나면 material로 import 해주세요.
class TestView extends StatelessWidget {
  const TestView({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Container();
  }
}
```

- TestView 클래스에서 전역 변수를 생성하려고 하면 경고가 뜹니다. [관련링크](https://dart.dev/tools/diagnostic-messages?utm_source=dartdev&utm_medium=redir&utm_id=diagcode&utm_content=must_be_immutable#must_be_immutable)
- StatefulWidget 처럼 UI가 바뀌는 동작에 대한 수행을 할 수도 없습니다.
- Flutter에서 위젯을 다시 빌드하는 동작을 최소화하려고 한다는 내용을 문서에서도 확인할 수 있는데, 이를 위해서라도 꼭 StatefulWidget으로 사용하지 않아도 될 부분이면 StatelessWidget으로 구현하면 되겠습니다.
    - const로 선언 가능한 위젯은 IDE에서 제안해주고 있는것도 위와 같은 이유입니다.

### Scaffold
- 문서: https://api.flutter.dev/flutter/material/Scaffold-class.html
- 앱 화면 하나를 작성할 때 build() 메서드에서 반환할 위젯 중 가장 최상위에 Scaffold 위젯을 사용합니다.
- appBar 필드에서 화면 최상단에 표시할 위젯들을 정의합니다. (굳이 필요 없으면 선언하지 않으면 됩니다.)
- body 필드에서 화면을 표현합니다.