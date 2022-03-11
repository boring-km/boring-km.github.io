---
layout: post
title: "Flutter pubspec.yaml"
categories: flutter-data
tags: [dev, flutter]
toc: true
---

- 문서: https://docs.flutter.dev/development/tools/pubspec
- 기본적인 작성법은 문서만 보아도 거의 이해할 수 있습니다.
- 몇 가지 추가 팁에 대해서만 작성하겠습니다.

### version
- **pubspec.yaml**에서 변경한 버전은 Android, iOS 앱 버전에도 함께 반영되어 release 빌드 시 변경된 버전으로 배포가 가능합니다.

### assets 추가 팁
- Flutter에서 참조할 파일들을 assets에 명시할 때 폴더 전체를 명시할 수 있습니다.

```yaml
flutter:
  assets:
    - images/
    - videos/
```

### plugin 관련 팁
- **dependencies**와 **dev_dependencies**에서 외부 모듈에 대한 의존성 추가를 합니다.
  - **dev_dependencies** 의존성은 앱 실행에 직접적인 도움을 주지는 않고, 테스트 코드 작성, 코드 자동 생성 등 부가적인 역할에 도움을 줍니다.
- 플러그인의 버전을 작성하지 않으면 **'pub get'** 명령 수행 시, 가장 최근 버전을 가져옵니다.
- **'pub upgrade'** 명령을 통해 작성되어 있는 버전이더라도 최신 버전을 찾아와 업그레이드 할 수 있습니다.
- 플러그인에 대한 더 자세한 내용은 문서를 참고해보세요.
  - https://docs.flutter.dev/development/packages-and-plugins/using-packages
  - https://dart.dev/tools/pub/dependencies
  - git repository에 있는 dart 프로젝트를 plugin처럼 사용하는 것도 가능합니다.
