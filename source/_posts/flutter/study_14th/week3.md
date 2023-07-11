---
layout: post
title: "Flutter 중급반 스터디 14기 3주차"
categories: flutter-study-14th
date: 2023-07-10
tags: [dev, flutter]
toc: true
---

# 14기 3주차 강민

## 이번주 한 업무 외 개발들

- 사이드 프로젝트 UI 부분만 개발
- [dynamic_image_crop](https://pub.dev/packages/dynamic_image_crop) 패키지 widget test 작성하다가 실패함

## 위젯 테스트 실패… -> 겨우 성공

위젯이 처음에 한번 빌드되고 나서 GlobalKey가 할당되기 전에

currentState 값을 사용하려고 해서 에러가 났다.

```dart
/// Crop the image as you can see on the screen.
void cropImage() {
  final cropType = cropTypeNotifier.cropType;
  if (cropType == CropType.none) {
    _callback(
      imageNotifier.image,
      painterSize.width.floor(),
      painterSize.height.floor(),
    );
  } else {
    final area = cropType == CropType.drawing
        ? _drawingKey.currentState!.getDrawingArea()
        : _painterKey.currentState!.getPainterArea(); // 에러가 발생하는 구간
    _callbackToParentWidget(area, cropType);
  }
}
```

```bash
══╡ EXCEPTION CAUGHT BY FLUTTER TEST FRAMEWORK ╞════════════════════════════════════════════════════
The following _TypeError was thrown running a test:
Null check operator used on a null value

When the exception was thrown, this was the stack:
#0      CropController.cropImage (package:dynamic_image_crop/src/controller/crop_controller.dart:60:37)
#1      main.<anonymous closure> (file:///Users/kangmin/dev/dynamic_image_crop/test/dynamic_crop_image_test.dart:47:20)
<asynchronous suspension>
<asynchronous suspension>
(elided one frame from package:stack_trace)
```

### 원인

- 이 위젯은 시작할 때 initState() 함수 안에서 비동기로 imageSize 값을 초기화한다.
  (현재 화면 크기에 맞춰 이미지 사이즈를 조절해야 한다.)
- imageSize 값이 초기화 되기 전까지 Container() 위젯이었다가, imageSize 값이 초기화되면 setState(() {}) 을 실행해 다시 렌더링한다.
- 그 때 _painterKey를 key로 사용하는 위젯이 빌드된다.
- 근데 테스트 코드에서 이 _painterKey가 Widget의 key로 할당되기 전에 currentState를 호출해서 null 에러가 발생했다.


### 시도한 방법

- 테스트 코드에서 빌드가 될 때까지 좀 기다려주기 → 실패
    - `await Future.delayed(const Duration(seconds: 1));` 이런 방식으로 await 걸면 테스트 함수 안에서 다음 라인의 코드가 동작하지 않음
    - `await tester.pumpWidget()` `await tester.pumpAndSettle()` 이런 테스트 함수 내에 Duration을 설정할 수 있지만 기다려주지 않음
- ~~HELP!!!!!!!!~~

- 테스트 성공 -> tester.runAsync() 사용해서 기다려주면 된다.

```dart
import 'package:flutter_test/flutter_test.dart';

Future<void> waitAndPumpAndSettle(WidgetTester tester, Duration duration) async {
  await tester.runAsync(() async {
    await Future<void>.delayed(duration);
  });

  await tester.pumpAndSettle();
}
```

- 테스트 코드 전체

```dart

import 'dart:io';
import 'dart:typed_data';

import 'package:dynamic_image_crop/src/controller/crop_controller.dart';
import 'package:dynamic_image_crop/src/crop/crop_type.dart';
import 'package:dynamic_image_crop/src/dynamic_image_crop.dart';
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

import 'test_utils.dart';

void main() {
  TestWidgetsFlutterBinding.ensureInitialized();

  group('FigureShapeView cropImage success test', () {
    late File file;
    late Uint8List image;
    late CropController cropController;

    const expectedCroppedWidth = 300;
    const expectedCroppedHeight = 300;
    const defaultPhysicalSize = Size(1920, 1000);

    setUp(() {
      file = File('test/assets/sample_image.png'); // 1920 x 880
      image = file.readAsBytesSync();
      cropController = CropController();
    });

    testWidgets('circle CropType success test', (tester) async {
      tester.view.physicalSize = defaultPhysicalSize;
      cropController.cropTypeNotifier.value = CropType.circle;
      const expectedImageBytesLength = 81129;

      final testWidget = MaterialApp(
        home: DynamicImageCrop(
          image: image,
          controller: cropController,
          onResult: (image, width, height) {
            debugPrint(
                'image length: ${image.length}, width: $width, height: $height');
            expect(expectedImageBytesLength, image.length);
            expect(expectedCroppedWidth, width);
            expect(expectedCroppedHeight, height);
          },
        ),
      );
      await tester.pumpWidget(testWidget);

      await waitAndPumpAndSettle(tester, const Duration(seconds: 1));
      await waitAndPumpAndSettle(tester, const Duration(seconds: 1));

      cropController.cropImage();

      await waitAndPumpAndSettle(tester, const Duration(seconds: 1));
      await waitAndPumpAndSettle(tester, const Duration(seconds: 1));
    });

    testWidgets('rectangle CropType success test', (tester) async {
      tester.view.physicalSize = defaultPhysicalSize;
      cropController.cropTypeNotifier.value = CropType.rectangle;
      const expectedImageBytesLength = 98868;

      final testWidget = MaterialApp(
        home: DynamicImageCrop(
          image: image,
          controller: cropController,
          onResult: (image, width, height) {
            debugPrint(
                'image length: ${image.length}, width: $width, height: $height');
            expect(expectedImageBytesLength, image.length);
            expect(expectedCroppedWidth, width);
            expect(expectedCroppedHeight, height);
          },
        ),
      );
      await tester.pumpWidget(testWidget);

      await waitAndPumpAndSettle(tester, const Duration(seconds: 1));
      await waitAndPumpAndSettle(tester, const Duration(seconds: 1));

      cropController.cropImage();

      await waitAndPumpAndSettle(tester, const Duration(seconds: 1));
      await waitAndPumpAndSettle(tester, const Duration(seconds: 1));
    });

    testWidgets('triangle CropType success test', (tester) async {
      tester.view.physicalSize = defaultPhysicalSize;
      cropController.cropTypeNotifier.value = CropType.triangle;
      const expectedImageBytesLength = 56144;

      final testWidget = MaterialApp(
        home: DynamicImageCrop(
          image: image,
          controller: cropController,
          onResult: (image, width, height) {
            debugPrint(
                'image length: ${image.length}, width: $width, height: $height');
            expect(expectedImageBytesLength, image.length);
            expect(expectedCroppedWidth, width);
            expect(expectedCroppedHeight, height);
          },
        ),
      );
      await tester.pumpWidget(testWidget);

      await waitAndPumpAndSettle(tester, const Duration(seconds: 1));
      await waitAndPumpAndSettle(tester, const Duration(seconds: 1));

      cropController.cropImage();

      await waitAndPumpAndSettle(tester, const Duration(seconds: 1));
      await waitAndPumpAndSettle(tester, const Duration(seconds: 1));
    });
  });
}


```

## 사이드 프로젝트 UI 작업 짧게

- AnimatedSwitcher, AnimatedOpacity, AnimatedContainer

## EventChannel을 어쩔 수 없이 쓰게 된 사례
- **원인: Native 코드에서 callback으로밖에 구현이 안되는 상황**

### 상세내용

- 모든 앱을 관리하는 중앙집중형(?) 부모 앱(Android) → 자식 앱 (flutter)
1. 앱 권한을 계속 거부해서 더이상 팝업이 안뜨면 부모 앱 호출
2. 부모 앱은 권한을 받아야 한다는 팝업 노출 후 설정 화면으로 이동 (삼성 기기의 Knox 연동으로 네비게이션 바를 강제로 감춤)
3. 결과를 다시 받으면 권한 여부 확인

```dart
class NativeFunctions {
  static const INTENT_CHANNEL = MethodChannel('smart_korn_writing/intent');
  static const PERMISSION_EVENT_CHANNEL = EventChannel('smart_korn_writing/permission');

  static Future<bool> requestPermissionToBookclub() async {
    unawaited(INTENT_CHANNEL.invokeMethod('메소드 채널 method name'));
    final result = await PERMISSION_EVENT_CHANNEL.receiveBroadcastStream().elementAt(0);
    if (result is bool) {
      return result;
    }
    return false;
  }

	// others...
}
```

```kotlin
// 1
if (call.method == "메소드 채널 method name") {
	requestPermissionToBookclub()
}

// 2
private fun requestPermissionToBookclub() { //  부모앱 연동
	val intent = Intent("액션명")
	intent.putExtra("PACKAGE", BuildConfig.APPLICATION_ID) // 요청 패키지
	intent.putExtra("PERMISSION", "APP")
	startActivityForResult(intent, PERMISSION_REQUEST_CODE)
}

// 3
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    if (requestCode != PERMISSION_REQUEST_CODE) return
    // 마지막 권한 체크
    if (ActivityCompat.checkSelfPermission(
            this,
            Manifest.permission.RECORD_AUDIO
        ) != PackageManager.PERMISSION_GRANTED
        || ActivityCompat.checkSelfPermission(
            this,
            Manifest.permission.CAMERA
        ) != PackageManager.PERMISSION_GRANTED
    ) {
        finishApp()
    } else {
        permissionEventSink?.success(true)
    }
}
```

- FlutterActivity는 onActivityResult()가 deprecated X → 일반적으로(?) 사용하는 Activity 상속 클래스들은 deprecated 였지만!
- 권한이 그래도 없다면 앱을 종료시켜 버림 → 부모앱 화면으로 돌아감
