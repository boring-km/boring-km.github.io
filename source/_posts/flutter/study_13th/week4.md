---
layout: post
title: "Flutter 중급반 스터디 13기 4주차"
categories: flutter-study-13th
tags: [dev, flutter]
toc: true
---

# 진행도

- [x]  1주차: 프로젝트 및 자기 소개
- [x]  2주차: 프로필 소개 화면 Flutter Web으로 만들어보기
- [x]  3주차: 링크풀 테스트 코드 작성 및 리팩토링
- [x]  4주차: 링크풀 테스트 코드 작성 및 리팩토링
- [ ]  5주차: 링크풀 테스트 코드 작성 및 리팩토링
- [ ]  6주차: 이미지 Custom Crop Plugin 개발
- [ ]  7주차: 이미지 Custom Crop Plugin 개발
- [ ]  8주차: 이미지 Custom Crop Plugin 개발

# 테스트 커버리지 측정 및 올리기

- 테스트/코드 커버리지 비율이 무조건 높다고 좋은건 아님.
- 그래도 테스트 코드가 없는 것보다는 있는 게 좋다.
- **유의미한 테스트를 잘 작성하는 것이 중요하다.**

## 테스트 커버리지 확인 방법

1. `flutter test —coverage` → coverage/lcov.info 파일 생성됨
2. `brew install lcov` → lcov.info 파일을 쉽게 보기 위한 도구 설치
3. `genhtml coverage/lcov.info -o coverage/html` → lcov.info 파일을 html 형태로 변환
4. `open ./coverage/html/index.html` → 생성된 html 파일 열어보기

![img.png](/images/flutter/study_13th/week4/1.png)

## 지난 주에 작성한 테스트 코드의 테스트 커버리지 확인해보기

- 자동 생성되는 파일들은 예외처리한다.
- 거의 테스트를 작성하지 않아 3.6% 밖에 없음을 확인했다.
- 여기서부터는 정말 노력으로밖에 채울 수 없다.
- 특히나 UI 코드가 많아 테스트 작성하는데 시간과 비용이 많이 들어갈 것 같다.

![img.png](/images/flutter/study_13th/week4/2.png)

## 참고 글

테스트 커버리지 확인 방법 안내: [https://dev-yakuza.posstree.com/ko/flutter/test-coverage/](https://dev-yakuza.posstree.com/ko/flutter/test-coverage/)

How to get your "actual" test coverage of your Flutter applications?: [https://medium.com/flutter-community/how-to-actually-get-test-coverage-for-your-flutter-applications-f881c0ae8155](https://medium.com/flutter-community/how-to-actually-get-test-coverage-for-your-flutter-applications-f881c0ae8155)

# 통합 테스트 해보기

- 저번주에 `WidgetTester.tap()` 함수가 ElevatedButton에서는 동작했음.
- 실제 코드로 테스트하는 과정

## 로그인/로그아웃 통합 테스트 해보기 (지훈님 발표 중에 테스트 성공!)

- 소셜 로그인은 flutter 안에서만 테스트하기가 어려움.
- id/password 로그인을 안보이게 추가하여 진행하기

```dart
void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  // email: signed@test.com
  // pw: 123456
  testWidgets('이미 가입된 유저 계정의 로그인/로그아웃 테스트', (WidgetTester tester) async {
    // 1. 앱 실행
    await app.main();
    await tester.pumpAndSettle(const Duration(seconds: 2));

    // 2. 튜토리얼 시작 버튼 tap
    final startButton = find.byKey(const Key('StartAppButton'));
    if (startButton.evaluate().isNotEmpty) {
      await tester.tap(startButton);
      await tester.pump(const Duration(milliseconds: 500));
    }

    // 3. 테스트 로그인 버튼 tap
    const testLoginButtonKey = Key('SignedUserLoginButton');
    if (find.byKey(testLoginButtonKey).evaluate().isNotEmpty) {
      InkWellButton(testLoginButtonKey).onTap!.call();
      await tester.pump(const Duration(seconds: 1));
    }

    // 4. 홈화면 BottomNavigation에서 마이페이지로 이동
    await tester.pumpAndSettle();
    await tester.pumpAndSettle(const Duration(seconds: 2));
    final navigation = find.byKey(const Key('MainBottomNavigationBar'));
    final navigationBar = navigation
        .evaluate()
        .first
        .widget as BottomNavigationBar;
    navigationBar.onTap!.call(3);
    await tester.pump(const Duration(seconds: 1));

    // 5. 로그아웃 버튼 클릭
    const logoutKey = Key('menu:로그아웃');
    InkWellButton(logoutKey).onTap!.call();
    await tester.pump(const Duration(seconds: 1));

    // 6. 로그아웃 다이얼로그에서 로그아웃 수행
    const logoutButtonKey = Key('MyPageDialogRightButtonKey');
    await tester.tap(find.byKey(logoutButtonKey));
    await tester.pump(const Duration(seconds: 1));

    // 7. 다시 로그인 화면으로 돌아왔는지 확인
    final actual = find.byKey(testLoginButtonKey);
    expect(actual, findsWidgets);
  });
}
```

# 추가적으로…

Android에서 선언형 UI 그리는 Jetpack Compose 사용한 사이드 프로젝트 잠시 소개…?

- 달력 들어간 TODO 앱
- 금융습관들을 루틴으로 관리할 수 있도록 고도화 예정

![img.png](/images/flutter/study_13th/week4/3.png)![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cc0577dc-7edf-4dd9-9d79-e49cdb780b71/Untitled.png)

- 개인적으로 Preview 제공하는 기능이 UI 격리해서 테스트하는데 굉장히 좋은 것 같다.

[Jetpack Compose UI App Development Toolkit - Android Developers](https://developer.android.com/jetpack/compose?gclid=CjwKCAjwxr2iBhBJEiwAdXECw4CPihTvFILzy9XTURynpGjgt81VDCsg0L4kQ_LTz6ujJBJ-v-fC0xoCkuwQAvD_BwE&gclsrc=aw.ds)

# 이런 것도 있어요

테스트 코드 작성과 관련해서 좀 더 편하게 작성하기 위한 추가적인 패키지가 있지 않을까해서 봤더니 역시 있었습니다.

[bdd_widget_test | Flutter Package](https://pub.dev/packages/bdd_widget_test)

[convenient_test | Flutter Package](https://pub.dev/packages/convenient_test)
