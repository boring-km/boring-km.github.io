---
layout: post
title: "Flutter 중급반 스터디 13기 2주차"
categories: flutter-study-13th
tags: [dev, flutter]
toc: true
---

# 13기 2주차 강민

- [x]  1주차: 프로젝트 및 자기 소개
- [x]  2주차: 프로필 소개 화면 Flutter Web으로 만들어보기
- [ ]  3주차: 링크풀 테스트 코드 작성 및 리팩토링 ← 프로필 소개 화면 Flutter Web으로 만들어보기
- [ ]  4주차: 링크풀 테스트 코드 작성 및 리팩토링
- [ ]  5주차: 링크풀 테스트 코드 작성 및 리팩토링
- [ ]  6주차: 이미지 Custom Crop Plugin 개발
- [ ]  7주차: 이미지 Custom Crop Plugin 개발
- [ ]  8주차: 이미지 Custom Crop Plugin 개발

[13기 1주차 강민](https://boring-km.github.io/2023/04/10/flutter/study_13th/week1/)

## 일정 변경사항

2주 연속으로 개발 기간으로 잡기에 아깝기도 해서 프로필 내용은 천천히 검토하면서 채우도록 하고, 다음주부터 링크풀 리팩토링 시작하기로 함.

## 소스코드

[https://github.com/boring-km/profile](https://github.com/boring-km/profile)

- Flutter Web 프로젝트
- 상세 내역은 markdown으로 작성해서 파일 내용만 바꿔도 업데이트 될 수 있도록 구현
- github page로 배포 (아래는 github workflow)

[profile/workflow.yml at master · boring-km/profile](https://github.com/boring-km/profile/blob/master/.github/workflows/workflow.yml)

```yaml
name: Profile Web
on:
  push:
    branches:
      - master
jobs:
  build:
    name: Build Web
    env:
      my_secret: $
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.7.11'
          channel: 'stable'
      - run: flutter --version
      - run: flutter config --enable-web
      - run: flutter pub get
      - run: flutter build web --release
      - run: |
          # 2. change path to [existed lib/main.dart path]/build/web
          cd ./build/web
          pwd
          git init
          git config --global user.email kms0644804@naver.com
          git config --global user.name boring-km
          git status
          git remote add origin https://${{secrets.commit_secret}}@github.com/boring-km/profile.git 
          git checkout -b gh-pages
          git add --all
          git commit -m "update"
          git push origin gh-pages -f
```

## known issue? - Flutter Web & SEO (Accessibility)

- flutter web에서 SEO(검색 엔진 최적화) 이슈가 가장 흔한 이슈로 알고 있는데, flutter도 많이 발전한 만큼 방법이 생겼지 않을까 해서 찾아봤다.
- 이전 Flutter에서는 Semantics 위젯으로 감싸야만 접근이 가능했던 것 같은데, **지금은 굳이 Semantics 위젯을 작성하지 않아도 가능했다.**

[Going deeper with Flutter’s web support](https://medium.com/flutter/going-deeper-with-flutters-web-support-66d7ad95eb52)

- 참고 글 - html처럼 inspecting 가능했음

![Untitled](/images/flutter/study_13th/week2/example.png)

### How to? (2가지 방법)

- 우선 flutter web 빌드 할 때 web-render 옵션을 html로 설정해야 한다!
- 명령어로 실행/빌드할 때 설정하는 옵션과 Runtime에서 설정하는 옵션이 있다.
- 실행 옵션을 간편하게 하기 위해 런타임 옵션으로 설정하고 진행한다. (index.html에서 아래 내용처럼 수정하면 된다.)

```jsx
let useHtml = // ...
_flutter.loader.loadEntrypoint({
  onEntrypointLoaded: async function(engineInitializer) {
    let config = {
      renderer: "html",
    }
    let appRunner = await engineInitializer.initializeEngine(config);
    await appRunner.runApp();
  }
});
```

[Web renderers](https://docs.flutter.dev/development/platform-integration/web/renderers)

- 그리고 여기서 상황에 맞게 변경하면 될 것 같다.
- 이왕이면 공식문서에서 제공하는 2번의 방식을 사용하는 게 낫겠다.

1. JavaScript 함수 호출, Console에서 입력해서 활성화 가능

```jsx
function setDebuggable() {
    document.querySelector('flt-glass-pane').shadowRoot.querySelector('flt-semantics-placeholder').click({force: true});
  }
```

2. flutter에서 semantic enable 하기 ([관련문서](https://docs.flutter.dev/development/accessibility-and-localization/accessibility?tab=browsers#screen-readers))

```dart
RendererBinding.instance.setSemanticsEnabled(true);
```

## 사소한 이슈

Flutter Web으로 로컬에서 Chrome으로 Run 하면 문제없이 화면이 보이는데, github page로 배포하면 화면이 안보이는 현상 있음

- Inspector를 통해 오류를 확인하면 _flutter 객체를 javascript에서 찾지 못해서 발생.

[Getting _flutter is undefined in flutter web, only in production](https://stackoverflow.com/questions/72833719/getting-flutter-is-undefined-in-flutter-web-only-in-production)

[https://github.com/flutter/flutter/issues/107448](https://github.com/flutter/flutter/issues/107448)

<base *href*="$FLUTTER_BASE_HREF"> → <base *href*="/profile/">

## 배포된 Web Url

[boring-km profile](https://boring-km.github.io/profile)

## 회고

- 지난 과거를 되돌아보면서 이력서 비스무레한 내용을 적다보니 후회가 많이 남았습니다. 더 열심히 살아야겠어요.
- Web으로 화면 크기를 큰 간격으로 조절하다보니 앱으로 개발하면서는 생각지못한 영역에서 Overflow가 발생하는 걸 볼 수 있었어요.
- Flutter Web에서 모바일 웹인지 데스크탑 웹인지 구별하는 방법도 있을까 해서 찾아보니까 방법이 있네요

[Detect whether flutter web app is running on desktop or mobile · Issue #80505 · flutter/flutter](https://github.com/flutter/flutter/issues/80505#issuecomment-821817704)

```dart
import 'package:flutter/foundation.dart';
final isWebMobile = kIsWeb &&
 (defaultTargetPlatform == TargetPlatform.iOS ||
	 defaultTargetPlatform == TargetPlatform.android);
```

## 질문

- 처음에 화면 로딩할 때 Text 값이 잠깐동안 X 모양으로 보였다가 글자가 나타나는데 이유를 아시는 분 계실까요?
- flutter web으로 개발하시면서 모바일 웹 따로 데스크탑 따로 UI 구성하시나요..?