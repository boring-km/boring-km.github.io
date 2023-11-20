---
layout: post
title: "Flutter 중급반 스터디 16기 1주차"
categories: flutter-study-16th
tags: [dev, flutter]
toc: true
---

- 잠시 Flutter에서 눈을 돌리고…

  Compose에 빠져서 Android만 개발하다가 왔습니다… 하하


### 간단 소개

1. 회사에서 앱을 개발하고 있는 **주니어** 개발자입니다.
2. 남양주에 살고 있습니다.
3. 업무 외에 이런저런 사이드프로젝트 팀에 속해서 앱을 담당중입니다.

# 프로젝트 소개

## 1.  조 편성 프로그램 만들기 (12월 초까지)

- 교회에서 매달 한번씩 3~4명씩 랜덤으로 조를 만들어서 밥먹는 모임 있음
- 현재는 번호표 뽑아서 그 번호들로 조 편성하는 프로그램 사용 중 → 누가 누군지 일일이 일어나서 확인

→ 현장에 모여있는 사람들만 조 편성 가능하게 해야함

- 좀 더 편하고 재밌게 조 편성하는 프로그램을 `Flutter Web`으로 만들어보는 중

## 2. 그린라이프 앱 개발

![img.png](/images/flutter/study_16th/img.png)

![img_1.png](/images/flutter/study_16th/img_1.png)

- MVP 기능만 우선으로 개발 어느정도 완료하고 곧 출시 예정 `Flutter(Android, iOS)`

## 3. 10일 사이드 프로젝트 지원

https://bside.best/potenday

- 312 지원했음
- 어떤 프로젝트 할지는 12월에 알게 될 듯
- 아무튼 Flutter 사용해서 모바일 앱 개발할 듯

### 그 외에…

- 전에 쓱님이 추천해주신 Five Lines of Code 스터디 중 → ts 말고 Compose Multiplatform으로 구현해보는 중
- 초등학교 4학년 코딩 멘토링..? 중입니다.
    - C#으로 간단한 알고리즘 문제풀기
    - 유니티로 체스 게임 만들기
- 사이드 프로젝트 링크풀 앱 파트에서 개발 중… [iOS](https://apps.apple.com/us/app/%EB%A7%81%ED%81%AC%ED%92%80-%EC%B2%B4%EA%B3%84%EC%A0%81%EC%9D%B8-%EB%A7%81%ED%81%AC-%EA%B4%80%EB%A6%AC%EC%9D%98-%EC%8B%9C%EC%9E%91/id1644108674), [Android](https://play.google.com/store/apps/details?id=com.mr.ac_project_app)
- 부업으로 텍스트 입력하면 그에 맞게 점자판 올라오는 기기가 있는데, Flutter Windows와 C++ 개발로 연동함

(https://www.dotincorp.com/kr/)

![img_2.png](/images/flutter/study_16th/img_2.png)

![img_3.png](/images/flutter/study_16th/img_3.png)

- DotCell에서 지원해준 라이브러리가 32비트 전용이었는데 flutter 프로젝트 안에 넣으려니까 64비트 밖에 안되어서 MethodChannel로 못하고
- 단독 콘솔프로그램을 실행시키고 커맨드 실행하는 방식으로 구현함.
