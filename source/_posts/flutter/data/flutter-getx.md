---
layout: post
title: "Flutter 의존성 주입 - GetX"
categories: flutter-data
tags: [dev, flutter]
toc: true
---

- 문서: https://pub.dev/packages/get
- 의존성 주입 플러그인이고, 다음에 설명할 'provider' 플러그인보다 간편한 코드 작성이 장점입니다.
  - 예를 들어, 프로젝트 전역에서 현재 context를 손쉽게 호출할 수도 있습니다.
- 왜 Getx를 사용해야 하는가에 대해 문서에도 잘 나와있습니다. [링크](https://pub.dev/packages/get#why-getx)

### Why Getx? 요약
1. Flutter 업데이트에 따른 패키지 종속성/호환성 이슈 해결
2. 개발자들이 싫어하는 상용구를 간소화
3. 상태 값이 바뀔 때 편리하게(변수 뒤에 .obs 붙여서) 화면을 자동 업데이트
4. 최소한의 리소스를 사용하게 하여 성능 향상
5. 뷰의 비즈니스 로직에 의존하지 않도록 분리

### 구현 순서
- 확립되어 있는 개발 방식이랄게 없는 단계이지만, 제가 구현한 방식 위주로 소개해보겠습니다.
1. main() 안에 엡 화면 라우팅을 위한 페이지 선언을 하면서, 바인딩할 객체를 지정합니다. (Bindings)
2. 바인딩 클래스들을 한 파일에 넣어놓고, 나중에 화면 내부에서 필요한 의존성이 생길 때마다 이 파일에 'Get.put()'을 이용해 GetxController 및 기타 세부 구현체에 대한 의존성을 추가해줍니다.
3. 기본적으로 1개의 화면 당 하나의 **GetxController** 상속 클래스를 매칭하여 UI 클래스를 **GetView<GetxController 상속 클래스>**로 정의합니다.
4. 데이터 로직에 대한 부분은 최대한 **GetxController** 상속 클래스에 ViewModel 형태로 구현합니다.
5. 이 후로는 자유롭게 본인의 스타일로 프로젝트를 만들어나가면서 자신만의 노하우가 생길 것입니다.

### 사용 팁...?
- UI를 그릴 위젯을 StatelessWidget/StatefulWidget이 아닌 **GetView**를 상속하면, 상태관리가 훨씬 쉽습니다.
  - **GetView**는 내부적으로 각 ViewModel 영역을 담당할 **GetxController** 클래스의 구현 클래스를 **Generic Type**으로 받습니다.
  - 내부에서 **StatelessWidget** 형태를 띄고 있기 때문에 **StatefulWidget**을 더이상 쓰지 않아도 된다고 문서에 써있긴 한데,
    **StatefulWidget**으로 구현하면 더 쉽게 그려질 UI를 굳이 번거롭게 **GetView**로 구현하지는 않아도 될 것 같습니다.
- 화면 업데이트를 .obs 값으로 관리하기가 불편하다면, update() 함수를 호출해서 원하는 시점에 build() 메서드를 실행해줍니다.
  - update()는 GetxController 클래스에 선언된 메서드입니다.
  - update() 사용 시 controller에서 View를 따로 지정하여 업데이트 한 것이 아니므로, UI 코드에서 **GetxController**의 update() 실행 시 업데이트 될 영역을 GetBuilder 위젯으로 wrapping하면 됩니다.
- [이 링크](https://pub.dev/packages/get#other-advanced-apis)에 자주 쓰일만한 메서드가 모여 있어서 참고하면 좋겠습니다.
- Unit Test에서도 getx 플러그인을 자유롭게 사용할 수 있습니다.
- GetConnect 클래스를 통해 HTTP 메서드 통신이 가능합니다. 다만, http 통신에 대해 좀더 직접적으로 개입하고 싶다면 [http 플러그인](https://pub.dev/packages/http)을 사용하세요.
