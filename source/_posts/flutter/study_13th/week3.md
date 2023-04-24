---
layout: post
title: "Flutter 중급반 스터디 13기 3주차"
categories: flutter-study-13th
tags: [dev, flutter]
toc: true
---

# 진행도

- [x]  1주차: 프로젝트 및 자기 소개
- [x]  2주차: 프로필 소개 화면 Flutter Web으로 만들어보기
- [x]  3주차: 링크풀 테스트 코드 작성 및 리팩토링
- [ ]  4주차: 링크풀 테스트 코드 작성 및 리팩토링
- [ ]  5주차: 링크풀 테스트 코드 작성 및 리팩토링
- [ ]  6주차: 이미지 Custom Crop Plugin 개발
- [ ]  7주차: 이미지 Custom Crop Plugin 개발
- [ ]  8주차: 이미지 Custom Crop Plugin 개발

# 프로젝트 LINKPOOL 소개

다른 앱에서 url 링크를 쉽게 저장하고 공유할 수 있는 앱

## Store Link

- 링크풀 - 체계적인 링크 관리의 시작
- PlayStore: [https://play.google.com/store/apps/details?id=com.mr.ac_project_app](https://play.google.com/store/apps/details?id=com.mr.ac_project_app)
- AppStore: [https://apps.apple.com/us/app/링크풀-체계적인-링크-관리의-시작/id1644108674](https://apps.apple.com/us/app/%EB%A7%81%ED%81%AC%ED%92%80-%EC%B2%B4%EA%B3%84%EC%A0%81%EC%9D%B8-%EB%A7%81%ED%81%AC-%EA%B4%80%EB%A6%AC%EC%9D%98-%EC%8B%9C%EC%9E%91/id1644108674)


## 사용 기술

![tutorial](/images/flutter/study_13th/week3/tutorial.png)

- App 외부에서 url 저장 기능
- 해당 url의 meta 태그 정보 저장함
- 나머지는 거의 게시판 기능


### html meta 태그 정보 활용 (OGP)

- Facebook에서 개발한 OGP(Open Graph Protocol)
- 앱 내부에선 flutter 패키지 metadata_fetch 사용

[metadata_fetch | Dart Package](https://pub.dev/packages/metadata_fetch)

- Android에선 jsoup 라이브러리 활용해 html의 메타태그 정보 가져왔음
- iOS에선 OGP Swift wrapper 라이브러리 사용

[https://github.com/satoshi-takano/OpenGraph](https://github.com/satoshi-takano/OpenGraph)

### App 내부 DB 활용

- flutter: [https://pub.dev/packages/sqflite](https://pub.dev/packages/sqflite)
- AOS, iOS 네이티브 화면에서도 동일 데이터에 접근하기 위해 SQLite 사용, db 파일경로를 공유하여 양방향으로 기록하는 구조 사용

### GitHub Source
- https://github.com/Monday-Rocket/ac_project_app

# 테스트 코드 작성 및 리팩토링

![flow](/images/flutter/study_13th/week3/flow.png)

## 프로필 이미지 변경 화면 테스트

![test1](/images/flutter/study_13th/week3/test1.png)

- 마이페이지 화면에서 프로필 이미지를 누르면 프로필 변경 화면으로 이동함

![test2](/images/flutter/study_13th/week3/test2.png)

- 지정된 이미지 9개에서 선택하는 기능
- 서버에 저장할 때는 이미지 번호만 저장함 → 추후 사용자 이미지로 저장하기 위해 String으로 저장
- 사용자 이미지를 업로드하는 기능으로 만들면 비용 나올까봐 현재는 구현 안되어 있음

### UX Flow

1. 변경할 프로필 이미지를 9개 이미지에서 하나를 골라 누르기
2. 하단의 변경하기 버튼이나, 완료 버튼 누르기
3. 프로필 정보 변경 API 호출
4. 이전 화면(마이페이지)으로 돌아가면서, 변경된 프로필 정보를 전달함
5. 전달 받은 데이터로 닉네임과 프로필 이미지 영역만 다시 그리기

### Widget 테스트 작성

1. 프로필 이미지가 불러와지는 지 테스트
2. 프로필 닉네임이 불러와지는 지 테스트
3. 특정 프로필 이미지를 선택했을 때 화면이 바뀌는 지 테스트

### test/widget_tap_helper.dart: GestureDetector 위젯을 key에서 찾아오는 위젯 (아래의 테스트에서 이걸 구현한 이유 있음)

```dart
GestureDetector GestureDetectorButton(Key key) =>
    find.byKey(key).evaluate().first.widget as GestureDetector;
```

### test/ui/view/user/change_profile_view_test.dart

```dart
import '../../widget_tap_helper.dart';

void main() {
  final testWidget = MaterialApp(
    home: ScreenUtilInit(
      designSize: const Size(393, 852),
      builder: (_, __) {
        return ProfileSelector(
          profile: Profile(
            nickname: '오키',
            profileImage: '02',
          ),
        );
      },
    ),
  );

  testWidgets('프로필 이미지가 불러와지는 지 테스트', (tester) async {
    await tester.pumpWidget(testWidget);

    const imagePath = 'assets/images/profile/img_02_on.png';
    final actual = find.image(Image.asset(imagePath).image);

    expect(actual, findsWidgets);
  });

  testWidgets('프로필 닉네임이 불러와지는 지 테스트', (tester) async {
    await tester.pumpWidget(testWidget);

    const nickname = '오키';
    final actual = find.text(nickname);

    expect(actual, findsWidgets);
  });

  testWidgets('4번 프로필을 선택했을 때 화면이 바뀌는 지 테스트', (tester) async {
    await tester.pumpWidget(testWidget);

    // 4번 이미지 버튼 tap
    const selectIndex = 4;
    const key = Key('select:$selectIndex');
    GestureDetectorButton(key)
        .onTap!
        .call(); // 동작 안 함 -> await tester.tap(found, warnIfMissed: false);
    await tester.pump(); // 안 기다려주면 변경된 UI 감지를 못함

    // 선택된 이미지 위젯 찾기
    const selectedImageKey = Key('selectedImage');
    final selectedWidget = find.byKey(selectedImageKey);
    final actualImage = (selectedWidget.evaluate().first.widget as Image).image;

    // 설정하려고 했던 이미지 가져오기 
    const targetImagePath =
        'assets/images/profile/img_0{$selectIndex+1}_on.png';
    final matcherImage = Image.asset(targetImagePath).image;
    
    // 같은 이미지인지 확인하기
    expect(actualImage, matcherImage);
  });
}
```

### Widget 테스트 우여곡절

- 3번에서 위젯을 선택해도 안 바뀜
- 리팩토링 필요: BlocBuilder로 감싸여진 구조에서 변경된 상태를 바로 전달받을 수가 없었음
- tap() 메소드가 동작을 안해서 다른 방법을 선택했음.

### API Mock 테스트 작성

- 프로필 이미지 변경 API Mock 테스트

```dart
class MockFirebaseAuth extends Mock implements FirebaseAuth {}

void main() {
  test('ProfileApi ChangeApi Success Test', () async {
    // Given: 변경하려는 프로필 이미지 번호는 2번
    const targetProfileImageNumber = '02';
    final expectedResult = ApiResult(
      status: 0,
      data: DetailUser(
        profileImg: targetProfileImageNumber,
      ),
    );

    // When 1: ProfileApi의 MockClient 설정하고
    final mockClient = MockClient((request) async {
      if (request.url.toString() == '$baseUrl/users/me') {
        return http.Response(
          jsonEncode(expectedResult),
          200,
          headers: {
            HttpHeaders.contentTypeHeader: 'application/json; charset=utf-8'
          },
        );
      }
      return http.Response('error', 404);
    });

    final profileApi = ProfileApi(
      client: CustomClient(
        client: mockClient,
        auth: MockFirebaseAuth(),
      ),
    );

    // When 2: ProfileApi의 changeImage() 실행했을 때,
    final result = await profileApi.changeImage(
      profileImg: targetProfileImageNumber,
    );

    // Then: 예상했던 결과와 동일하게 나오는지 확인한다.
    result.when(
      success: (actual) => expect(actual, expectedResult.data),
      error: fail,
    );
  });
}
```

### Mock 테스트 우여곡절

- http mocking이 잘 안되어서 힘들었음.
- mock test를 예상하고 만들지 않아 MockClient, MockFirebaseAuth를 사용하기 위한 구조로 변경이 필요했음.

![error](/images/flutter/study_13th/week3/error.png)

- 원인은 Response 인코딩에 있었음
- mock Response랑 다르다고 하면 될 것을 http patch 함수 날릴 때 부터 에러를 보내서 원인을 빨리 알아내기가 어려웠음.
- 해결에 참고한 링크: [https://stackoverflow.com/questions/52990816/dart-json-encodedata-can-not-accept-other-language](https://stackoverflow.com/questions/52990816/dart-json-encodedata-can-not-accept-other-language)

- **추후에 개선될 기능**
    - 나중에는 사용자가 직접 업로드한 이미지로도 프로필 이미지 바꿀 수 있게도 구현할 예정
    - 이미 선택된 프로필 이미지로는 변경이 안되게 수정

## 리팩토링

1. 프로필 변경 화면 안쪽 부분 Stateful로 변경
2. 프로필 변경 후 나올 때 화면 업데이트 개선 (불필요한 API 재조회 및 화면 전체 로딩하는 등)
3. iOS 공유패널 Open Graph 데이터 불러와질 때 까지 화면 잠시 blocking 하기 (로딩효과?)

# 회고

- 내가 작성했던 코드를 다시 보니 아주 형편없는 코드였다.
- 테스트를 처음부터 고려하면서 코딩하지 않으면 나중에 개고생한다.
- 익숙하지 않은 테스트 코드 작성에 훈련이 많이 필요함을 느꼈다.
- bloc 잘 다루는 법좀 공부해야겠다.

# 질문

1. 테스트 코드 작성에 관련된 선배님들의 훈수 부탁드립니다.
2. widget test에서 GestureDetector/InkWell 위젯의 WidgetTester.tap() 이 동작하지 않는 이유?
3. GridView로 분명 9개를 그렸는데 테스트코드에서 6개밖에 못 그리는 이유가 있을까요

![console](/images/flutter/study_13th/week3/console.png)
