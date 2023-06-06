---
layout: post
title: "Flutter 중급반 스터디 13기 6주차"
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
- [ ]  7주차: 이미지 Custom Crop Package 개발
- [ ]  8주차: 이미지 Custom Crop Package 개발

# 프로젝트 소개

- 목적: Flutter에서 다양한 모형으로 이미지를 crop 하는 라이브러리

## 기존 이미지 Crop 관련 패키지 조사

### image_cropper

[image_cropper | Flutter Package](https://pub.dev/packages/image_cropper)

- Native 영역에서 편집을 하고 있어 Flutter 소스로 커스터마이징 하기가 어려움

### crop

[crop | Flutter Package](https://pub.dev/packages/crop)

- crop할 특정 영역을 직접 정하지 못함

![img.png](/images/flutter/study_13th/week6/img.png)


### crop_your_image

[crop_your_image | Flutter Package](https://pub.dev/packages/crop_your_image)

- 위의 crop 패키지와 마찬가지로 이미지의 특정 영역을 선택해서 crop 하는 방식이 아닌 뒤에 있는 이미지를 움직여서 crop 영역에 맞추는 느낌

![img_1.png](/images/flutter/study_13th/week6/img_1.png)

### crop_image

[crop_image | Flutter Package](https://pub.dev/packages/crop_image)

![img_2.png](/images/flutter/study_13th/week6/img_2.png)

- 영역을 지정해서 사각형 crop까지는 가능
- 카카오톡에 있는 이미지 자르기랑 비슷

![img_3.png](/images/flutter/study_13th/week6/img_3.png)

### custom_image_crop

[custom_image_crop | Flutter Package](https://pub.dev/packages/custom_image_crop)

![img_4.png](/images/flutter/study_13th/week6/img_4.png)

## 새로 개발하려는 패키지의 차이점

![img_5.png](/images/flutter/study_13th/week6/img_5.png)

![img_6.png](/images/flutter/study_13th/week6/img_6.png)

![img_7.png](/images/flutter/study_13th/week6/img_7.png)

![img_8.png](/images/flutter/study_13th/week6/img_8.png)

1. 직접 crop 할 부분을 그려서 그 감싸지는 영역만 꺼내기
2. 사각형/원형 crop 사이즈 조절하면서 원하는 영역만 crop 할 수 있게 하기

## 최종 목표

- pub.dev에 배포해보기
- 핵심 기능만 압축해서 패키지화하기
- ui test 적용하기 (픽셀 비교)

# 주간 회고

- 사이드 프로젝트로 개발한 프로젝트 PlayStore 검토 중
- 이제 슬슬 이력서 작성할 때