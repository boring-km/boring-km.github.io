---
layout: post
title: "Flutter 단위 테스트 작성"
categories: flutter
tags: [dev, flutter]
toc: true
---

- Flutter 앱에서 단위 테스트를 하는 방법에 대해 간단하게 소개합니다.
- UI 테스트에 대해서는 좀 더 익숙해진 다음에 따로 글을 작성해야겠습니다.

## 테스트 코드 작성 및 실행
- /test 경로에 dart 파일을 생성합니다.

```dart
import 'package:flutter_test/flutter_test.dart';

void main() {
  
  // 초기화 코드 작성
  
  test('테스트 이름', () {
    // 테스트 할 코드 작성
  });
}
```

- 기본적인 코드 형태는 이렇습니다.
- main() 에서 'async-await' 문법을 사용해 초기화 코드가 비동기 코드여도 동기적으로 실행이 가능합니다.
- test() 메서드 내 코드 작성할 때도 마찬가지로 'async-await' 사용이 가능합니다.
- 테스트를 IDE에서 실행하게 되면 'flutter test' 로 시작하는 flutter test 명령이 실행됩니다.
- **'flutter test' 명령만 입력하면 현재 작성된 모든 테스트 코드를 한번에 실행할 수 있습니다.**

### 테스트 코드 예시
- 다음은 파일을 저장할 수 있는 CDN 서버에 파일을 업로드 시키는 단위 테스트 코드 예시입니다.

```dart
// import 생략...

void main() {

  CdnApi api = CdnApi();

  /// 앱에서 테스트 실행 시 명령어: flutter run .\test\api\cdn_api_test.dart
  /// android path: '/data/user/0/{앱 패키지 이름}/app_flutter/'
  test('CDN Data 업로드 테스트', () async {

    final cdnData = CdnData(
      cdnPath: 'CDN 서버 내 저장할 경로',
      filePath: '현재 파일이 저장 되어 있는 경로',
      fileName: '파일 이름.확장자',
    );

    final Result<bool> result = await api.uploadFiles([cdnData]);

    expect(result, const Result.success(true));
  });
}
```

- 단위 테스트의 GWT(Given-When-Then)에 따라 업로드하는 행위에 맞춰 작성했습니다.
- 해당 테스트는 테스트 코드를 로컬 환경에서 실행하는 **PC와 모바일 App 두 가지 환경에서 모두 테스트가 가능**합니다.
- 모바일 App에서 테스트 코드를 실행하고 검증하고 싶다면, 터미널에 'flutter run 테스트_할_파일_경로'를 입력하여 실행이 가능합니다.

### Mocking
- 사용할 라이브러리 문서: https://pub.dev/packages/mockito
- 위의 문서에서 자세하게 나와 있지만, 간단한 설명과 예시를 덧붙이겠습니다.
- mockito를 사용하기 위해 'build_runner' 라는 라이브러리가 추가로 필요합니다. [링크](https://pub.dev/packages/build_runner)
- 2가지 라이브러리를 모두 pubspec.yaml에 추가했다면 코드를 작성하면 됩니다.
- 해당 코드는 오준석 님의 ['Flutter 중급 - 클린 아키텍처'](https://www.inflearn.com/course/플러터-중급) 강의에서 따라해본 예제입니다.

```dart
// import 생략...

@GenerateMocks([http.Client])
void main() {
  test('Pixabay Api에서 iphone 데이터를 잘 가져와야 한다.', () async {
    const query = 'iphone';
    final client = MockClient();
    final api = PhotoApiRepositoryImpl(PixabayApi(client));

    when(client.get(Uri.parse(
        '${PixabayApi.baseUrl}?key=${PixabayApi.key}&q=$query&image_type=photo&pretty=true')))
        .thenAnswer((_) async => http.Response(fakeJsonBody, 200));

    final Result<List<Photo>> result = await api.fetch('iphone');

    expect((result as Success<List<Photo>>).data.length, 20);

    verify(client.get(Uri.parse(
      '${PixabayApi.baseUrl}?key=${PixabayApi.key}&q=$query&image_type=photo&pretty=true',)));
  });
}

String fakeJsonBody = """
  api 데이터가 너무 많아서 생략...
""";
```

1. @GenerateMocks([http.Client]) annotation을 이용해 mocking할 객체를 먼저 선언합니다.
2. 터미널에 'flutter pub run build_runner build' 명령을 실행하면 http.Client 객체에 대한 Mock 객체를 생성할 수 있는 코드가 같은 파일 경로에 자동생성 됩니다.
3. MockClient 객체를 선언하게 되면 해당 객체를 http.Client 객체와 같이 코드 작성이 가능합니다.
4. http.Client 객체가 수행해야 했을 작업을 **when()** 안에서 수행시킵니다.
5. **thenAnswer()** 안에서 테스트 코드가 Mock 객체에 기대하는 결과 값을 반환하도록 합니다.
6. 이제 api를 테스트하면 실제 http 통신이 아닌 Mock 객체에서 전달하는 결과를 받게 됩니다.

- verify() 메서드는 해당 테스트를 수행하면서 verify() 내에 선언한 함수가 정말 실행이 되었는지 확인합니다.
