---
layout: post
title: "Flutter 중급반 스터디 13기 5주차"
categories: flutter-study-13th
tags: [dev, flutter]
toc: true
---

# 진행도

- [x]  1주차: 프로젝트 및 자기 소개
- [x]  2주차: 프로필 소개 화면 Flutter Web으로 만들어보기
- [x]  3주차: 링크풀 테스트 코드 작성 및 리팩토링
- [x]  4주차: 링크풀 테스트 코드 작성 및 리팩토링
- [x]  5주차: 링크풀 테스트 코드 작성 및 리팩토링
- [ ]  6주차: 이미지 Custom Crop Plugin 개발
- [ ]  7주차: 이미지 Custom Crop Plugin 개발
- [ ]  8주차: 이미지 Custom Crop Plugin 개발

# 소프트웨어 개발의 꽃은 자동화 적용?

## CI에 테스트 커버리지 적용하기

- 테스트 커버리지를 측정하고 나서 자동화 시스템에 반영하지 못하면 불편한 업무 프로세스가 오히려 늘어날 수도 있으니 CI에 반영할 수 있는 방법을 찾아본다.
- 다양한 CI 도구들: [https://katalon.com/resources-center/blog/ci-cd-tools](https://katalon.com/resources-center/blog/ci-cd-tools)
- LINKPOOL 앱은 GitHub에 소스코드를 올리고 있으니 최대한 GitHub Action을 통해 무료로 적용해본다.

## Codecov

여기서는 Codecov 라는 도구를 사용해 테스트 커버리지를 더 쉽게 보고, CI에 반영할 수 있도록 해본다.

[Quick Start](https://docs.codecov.com/docs)

### codecov 동작을 정의하는 codecov.yml 작성

```yaml
codecov:
  require_ci_to_pass: true

comment:
  layout: "reach,diff,flags,files"
  behavior: default
  require_changes: false
  require_base: false
  require_head: false
  branches:
    - develop

coverage:
  range: 10..100
  round: down
  precision: 2
  status:
    project:
      default:
        target: 10%
        threshold: 0%
    patch:
      target: 10%
      threshold: 0%

ignore:
  - 'lib/models/result.freezed.dart'
  - 'lib/models/**/*.freezed.dart'
  - 'lib/models/*/*.freezed.dart'
  - 'lib/models/*/*.g.dart'
  - 'lib/gen/*.gen.dart'
  - 'lib/firebase_options.dart'
  - 'lib/util/logger.dart'
```

### Pull Request 작성 시 github action에서 감지하여 테스트 커버리지 계산하기

```yaml
name: Measure Code Coverage
on:
  pull_request:
    branches:
      - develop
jobs:
  test-coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: subosito/flutter-action@v2.8.0
        with:
          flutter-version: '3.10.0'
      - name: Install packages
        run: flutter pub get
      - run: echo '${{ secrets.DOTENV }}' | base64 -d > .env
      - run: echo "${{ secrets.FIREBASE_OPTIONS }}" | base64 -d > ./lib/firebase_options.dart
      - name: run flutter test
        run: flutter test --coverage
      - name: Upload coverage reports to Codecov
        uses: codecov/codecov-action@v3
```

## 웹페이지에서 확인

![Untitled](/images/flutter/study_13th/week5/1.png)

### github에서 확인

- 자동으로 PR에 Report 작성됨

![Untitled](/images/flutter/study_13th/week5/2.png)

![Untitled](/images/flutter/study_13th/week5/3.png)

# 주간 회고

- 다른 사이드 프로젝트 앱 개발하느라 정신이 없음 (곧 출시!)
- 매일 잠을 거의 못 자는 상태
- Kotlin의 suspend 함수가 비동기 함수들을 확실히 개발단계에서부터 제한하니까 실수가 적은 것 같다.

## 질문?

- flutter 3.10 업데이트 다들 해보고 계신가요? ㅎㅎ