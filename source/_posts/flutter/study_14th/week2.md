---
layout: post
title: "Flutter 중급반 스터디 14기 2주차"
categories: flutter-study-14th
date: 2023-07-03
tags: [dev, flutter]
toc: true
---

## Flutter Flame으로 간단한 게임 만들어보기

- Dodge Game → 총알 피하기 게임..?
- 2022년 8월 정도쯤 개발하다가 딴거하느라 바빠서 손 떼었던 프로젝트 살리기
- 간단한 상태관리는 Getx로 진행 → 이유는 딱히 없고 빨리 만들어보고 싶어서 선택했던걸로 기억
- Firebase Auth랑 Cloud Firestore 사용해 사용자 정보 저장함.

## 자바 초보 시절 만들었던 게임 (군대 전역 직후…)

- Swing으로 만들었음
- 그림은 전부 파워포인트 + 그림판
- 배경음악은 대충 저작권 없는 음원 찾아서 넣음
- https://github.com/boring-km/Java-Dodge-Game

## 주요 게임 화면

- 카카오 로그인도 넣었다가 귀찮아서 뺌
- 나중에 출시하려면 애플로그인도 하나 끼워넣어야 할듯.

![img.png](/images/flutter/study_14th/week2/img.png)

![img_1.png](/images/flutter/study_14th/week2/img_1.png)

![img_2.png](/images/flutter/study_14th/week2/img_2.png)

- 스마트폰 기기를 기울여서 움직임 https://pub.dev/packages/sensors_plus 사용
    - 학생 때 재밌게 했던 게임이 이 방식이었는데 아무리 검색해봐도 안나오는데 그리워서 채용
- 조이스틱으로 하는 방법도 구현은 해봤는데 개성이 별로 없는거 같음 + 조이스틱에 가려져서 보기 흉해짐

![img_3.png](/images/flutter/study_14th/week2/img_3.png)

## 개선안?

- 스테이지 부여해서 점점 어려워지도록 하기
- 다양한 유형의 총알? 만들기
    - 멀리서 일직선으로 선 그리면서 오는 Enemy
    - n초 동안 느린 속도로 따라오다가 사라지는 Enemy
    - etc…
- 광고 넣어보기 → 보통 Flutter 앱에서 광고 넣으면 어떤거 사용하나요?
    - GPT의 답변

  ![img.png](/images/flutter/study_14th/week2/img_4.png)

- 결론: https://pub.dev/packages/functional_google_mobile_ads 사용해보기 
