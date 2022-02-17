---
layout: post
title: "Flutter Hot Reload, Hot Restart"
categories: flutter
tags: [dev, flutter]
toc: true
---

- **실행 중인 Flutter 앱을 빠르게 재실행 해주는 기능입니다.**
- Flutter에는 Android 개발할 때에도 잠깐 있었던 Hot Reload 기능을 제공합니다.

## Hot Reload vs Hot Restart

### Hot Reload
- IntelliJ(Android Studio)는 번개 그림의 아이콘 버튼으로도 제공하고 있습니다. (단축키 Windows: Ctrl+s, Mac: Cmd+s)
- Hot Reload는 현재 화면에 보이는 위젯을 기준으로 변경된 코드를 반영합니다.
  - 현재 화면에 있는 Widget의 build() 메서드가 실행됩니다.
  - 현재 화면에 있지 않은 소스 코드가 런타임에서 반영이 됩니다.

### Hot Restart
- IntelliJ(Android Studio)는 실행 상태에서 다시 Run 실행하면 Hot Restart 동작
- Hot Restart는 처음 코드가 동작하는 main 함수부터 다시 실행됩니다.

### 자주 실수했던 예외 상황
- Native Plugin을 사용하거나 새로운 Resource(image, font...)를 추가하는 등 몇가지 상황에서는 실행 상태의 앱을 종료 시키고 Run 해야 적용되는 경우가 있습니다.
- Hot Reload 사용 시 현재 변경한 위젯에서 아직 초기화되지 않은 값을 쓰려고 하면서 에러가 발생할 수 있습니다.
  - 이전 화면이 존재한다면 이전 화면에서 Hot Reload 후 변경된 화면으로 이동으로 해결하거나 Hot Restart 사용

### 편리했던 상황
- UI 코드를 작성하면서 변경된 코드에 대한 피드백을 바로 확인이 되어서 개발이 편리했습니다.
- API를 변경하면서 실행할 때도 앱 종료 없이 재실행이 가능해서 편리했습니다.
