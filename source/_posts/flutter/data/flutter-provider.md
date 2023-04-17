---
layout: post
title: "Flutter 의존성 주입 - provider"
categories: flutter
tags: [dev, flutter]
toc: true
---

- 문서: https://pub.dev/packages/provider
- GetX를 사용해보고 나서 provider가 좀더 형식적인 코드 작성이 많다는 것을 깨달았습니다.
- 그래도 Flutter 앱 개발에 있어서 좀더 라이브러리에 덜 의존적인 원형의 형태를 사용하는 것에 초점을 둔다면 좋을 것 같습니다.
- Google I/O 2019 구글의 공식 추천 픽 -> https://www.youtube.com/watch?v=HrBiNHEqSYU
  - 나름 그때 시점으로는 provider가 좀더 안정적이어서 그런 것 같습니다.

### 구현 순서
- 확립되어 있는 개발 방식이랄게 없는 단계이지만, 제가 구현한 방식 위주로 소개해보겠습니다.
1. main() 안에서 MaterialApp 객체를 리턴하고 있는 App 클래스를 MultiProvider 객체의 child 영역 안에서 선언되도록 합니다.
2. 그리고 providers 필드에 사용하고자 하는 의존성 주입 모델들을 추가하면 됩니다.
3. 의존성이 추가하기 위해 **ChangeNotifier** 클래스를 mixin하는 **ViewModel** 클래스를 생성합니다.
4. **View**에서 필요로 하는 함수들을 **ViewModel** 클래스의 인스턴스를 provider를 통해 제공받습니다.
