---
layout: post
title: "Flutter 중급반 스터디 14기 4주차"
categories: flutter-study-14th
date: 2023-07-17
tags: [dev, flutter]
toc: true
---

# Test Coverage 100% 만들기

[dynamic_image_crop | Flutter Package](https://pub.dev/packages/dynamic_image_crop)

[Codecov](https://app.codecov.io/gh/boring-km/dynamic_image_crop)

## 테스트 폴더

```
test/
├── assets/
│   ├── sample_image.png
│   └── sample_image_vertical.png
├── controller/
│   ├── crop_controller_test.dart
│   └── image_change_notifier_test.dart
├── crop_area_moving_test.dart
├── drawing_view_crop_test.dart
├── figure_shape_view_crop_test.dart
├── none_crop_type_test.dart
├── resize_crop_area_test.dart
└── test_utils.dart

```

- 가로가 긴 PNG 이미지 하나와 세로가 긴 PNG 이미지로 테스트
    - 다른 이미지 확장자에 대해서는 아직 테스트 못함
- 자세한건 코드로…
    - https://github.com/boring-km/dynamic_image_crop/tree/master/test

# CI에 적용하기

```yaml
name: check test coverage

on:
  pull_request:
    branches:
      - publish
      - example_page

jobs:
  coverage:
    runs-on: macos-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - uses: subosito/flutter-action@v2.10.0
        with:
          flutter-version: '3.10.5'
      - name: Install packages
        run: flutter pub get
      - name: run flutter test
        run: flutter test --coverage
      - name: Upload coverage reports to Codecov
        uses: codecov/codecov-action@v3
        env:
          CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

- 패키지 배포하거나 example 페이지 pull request 할 때 동작
- 예외적인 상황 발견 (PC에서는 통과하는 테스트가 github 서버에서는 이미지가 동일한 바이트 크기로 crop 되지 않고 오차 있음)
    - https://github.com/boring-km/dynamic_image_crop/actions/runs/5568985397/jobs/10171990974

## publish workflow

```yaml
name: Publish to pub.dev

on:
  push:
    branches:
      - publish

jobs:
  publish:
    runs-on: macos-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - uses: subosito/flutter-action@v2.10.0
        with:
          flutter-version: '3.10.5'
      - name: Install packages
        run: flutter pub get
      - name: Setup Pub Credentials
        shell: bash
        env:
          INPUT_CREDENTIAL: ${{ secrets.CREDENTIAL_JSON }}
        run: |
          sh ./pub_login.sh
      - name: Check Publish Warnings
        run: dart pub publish --dry-run
      - name: Publish Package
        run: dart pub publish -f
```

- dry-run은 필수는 아님

### 과외 요청 받음

- 직장 동료 아들이 유니티로 게임 만드는 걸 재밌게 하고 있다는데, 좀 진지하게 공부 시키고 싶어서 요청 받음
- 토요일에 잠깐 화상으로 만났는데 너무 어려보여서 당황 (초4)

### 노션 캡쳐

![Untitled](/images/flutter/study_14th/week4/img.png)
