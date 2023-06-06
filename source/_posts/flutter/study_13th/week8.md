---
layout: post
title: "Flutter 중급반 스터디 13기 8주차"
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
- [x]  6주차: 이미지 Custom Crop Package 개발
- [x]  7주차: 이미지 Custom Crop Package 개발
- [x]  8주차: 이미지 Custom Crop Package 개발

# [pub.dev](http://pub.dev) 배포

[dynamic_image_crop | Flutter Package](https://pub.dev/packages/dynamic_image_crop)

### 외부 패키지 의존성 없이 하기

- 이왕 패키지를 직접 만드는 기회이니만큼, 다른 패키지와의 의존성을 없애고 싶었음
- Native Code 의존성 없이 Dart 언어만으로 구현하고 싶었음

### PUB POINTS

- 처음에 120점 나왔음
    1. (-10점) Description 설명이 60자 ~ 120자로 적어야 하는데 너무 짧게 적었음
    2. (-10점) dartdoc comments를 20% 이상 적어야 점수 줌
- 해결하니 다시 만점으로 채점해줌

### verified publisher

[Authentication required](https://pub.dev/create-publisher)

등록하려면 Domain Name 소유하고 있어야 함.

→ [boring-km.dev](http://boring-km.dev) 로 Google Domains에서 도메인 구매

### dart pub publish

- 명령어 하나만으로 배포되는게 정말 편리했음
- publish 하면 [pub.dev](http://pub.dev) 에서 자체적으로 analysis 한 뒤 평가됨 [PENDING ANALYSIS]
- Github Actions 사용해서 Automated publishing 해볼 예정

![Untitled](/images/flutter/study_13th/week8/img.png)

# dynamic_image_crop Demo

[https://boring-km.dev/dynamic_image_crop](https://boring-km.dev/dynamic_image_crop/)

# 남은 작업들

- 테스트 코드 작성 - 실제로 사용자가 지정한 영역을 잘 자르고 있는지 확인
- [README.md](http://README.md) 에 뱃지 이것저것 달아보기
- Github Actions로 publish 자동화할 브랜치 정하고 workflow 만들기
- CustomPaint 상속 객체 받아서 입력받은 도형대로 crop 할 수 있는 기능 추가

# 13기 스터디 회고

해야지 하고 미뤘던 작업들을 발표를 해야한다는 부담감을 갖고 하니까 목표대로 수행할 수 있었습니다.

여태 혼자서만 flutter 개발 해오다가 다른 분들 개발하시는 것들도 구경해보고 좋은 기회였습니다. ㅎㅎ
