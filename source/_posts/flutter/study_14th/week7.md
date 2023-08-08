---
layout: post
title: "Flutter 중급반 스터디 14기 7주차"
categories: flutter-study-14th
date: 2023-08-07
tags: [dev, flutter]
toc: true
---

이번 주에 안드로이드 개발에만 빠져 있었네요. 
Preview 기능이 Flutter에도 도입되었으면 개발이 훨씬 편해질 것 같습니다

# Not Flutter

![img.png](/images/flutter/study_14th/week7/img.png)

- GUI: PyQT로 제작
- 실행파일로 만들기 위해 py2app 사용함 → pyinstaller 라는 것도 있는데 뭔가 번들링하고 나서 실행이 안되어서 포기
- selenium: https://selenium-python.readthedocs.io/index.html
- Web에서 로그인을 하고 출근/퇴근 버튼을 눌러야 하는 동작을 수행

### 소스코드

- Safari에서는 개발자 모드에서 Remote Automation 활성화해야함.

![img_1.png](/images/flutter/study_14th/week7/img_1.png)

```python
def get_browser_driver():
    # macOS면 Safari, Windows면 Chrome
    if platform.system() == 'Darwin':
        driver = webdriver.Safari(keep_alive=False)
    else:
        driver = webdriver.Chrome(keep_alive=False)
    return driver
```

- webdriver 사용 코드

    ```python
    def start_work(driver):
        now_hour = datetime.datetime.now().hour
        if now_hour > 11:
            notice_not_yet(now_hour)
            return
    
        portal_login(driver)
        driver.get(start_work_url)
        time.sleep(1)
        driver.close()
        driver.quit()
        app.exit(0)
    
    def finish_work(driver):
        now_hour = datetime.datetime.now().hour
        if now_hour < 16:
            notice_not_yet(now_hour)
            return
    
        confirm, hour, minute = portal_login(driver)
        driver.get(finish_work_url)
        time.sleep(3)
        driver.execute_script("document.getElementById('input2').value = '" + hour + "'")
        driver.execute_script("document.getElementById('input3').value = '" + minute + "'")
        driver.execute_script("document.getElementsByName('check2')[1].checked = true")
    
        if confirm:
            driver.execute_script("$app.save()")
            time.sleep(1)
            driver.close()
            driver.quit()
        time.sleep(10)
        driver.close()
        driver.quit()
        app.exit(0)
    
    def portal_login(driver):
        driver.get(loginUrl)
        time.sleep(3)
        user_id, password, hour, minute, confirm = get_credentials()
        driver.find_element('id', 'j_username').send_keys(user_id)
        driver.execute_script("document.getElementById('j_username').value = '" + user_id + "'")
        driver.find_element('id', 'password').send_keys(password)
        driver.execute_script("document.getElementById('password').value = '" + password + "'")
        time.sleep(1)
        driver.execute_script("document.getElementById('btnSubmit').click()")
        time.sleep(2)
        return confirm, hour, minute
    ```


# 링크풀 앱에서 Flutter Widget으로 그릴 수 없었던 기능

다른 앱(인터넷 브라우저, 유튜브 등)에서 Flutter 앱 호출하기 전에 중간 UI 그리기

- [링크풀 - 체계적인 링크 관리의 시작(PlayStore)](https://play.google.com/store/apps/details?id=com.mr.ac_project_app)
- [링크풀 - 체계적인 링크 관리의 시작(AppStore)](https://apps.apple.com/us/app/%EB%A7%81%ED%81%AC%ED%92%80-%EC%B2%B4%EA%B3%84%EC%A0%81%EC%9D%B8-%EB%A7%81%ED%81%AC-%EA%B4%80%EB%A6%AC%EC%9D%98-%EC%8B%9C%EC%9E%91/id1644108674)

### iOS UI

iOS에서는 Share Extension 이라는 모듈을 추가해야 해당 기능 구현이 가능합니다.

Xcode에서만 개발이 가능해요

![img_2.png](/images/flutter/study_14th/week7/img_2.png)

![img_3.png](/images/flutter/study_14th/week7/img_3.png)

![img_4.png](/images/flutter/study_14th/week7/img_4.png)

![img_5.png](/images/flutter/study_14th/week7/img_5.png)

![img_6.png](/images/flutter/study_14th/week7/img_6.png)

### Android UI

- iOS 보다는 비교적 구현이 쉬운 편

![img_7.png](/images/flutter/study_14th/week7/img_7.png)

- Activity 추가하고 호출하면 됨 (아래는 layout)

![img_8.png](/images/flutter/study_14th/week7/img_8.png)

- 불러와지기 위한 Activity에 intent-filter 추가

```xml
<activity
		android:name="com.mr.ac_project_app.view.share.ShareActivity"
		android:exported="true"
		android:launchMode="singleTop"
		android:screenOrientation="portrait"
		android:theme="@style/TransparentCompat"
		tools:ignore="LockedOrientationActivity">

		<intent-filter>
				<action android:name="android.intent.action.SEND" />
        <action android:name="android.intent.action.PROCESS_TEXT" />

        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
		</intent-filter>
</activity>
```

## 오프라인 데이터 연동

- Flutter 안에서 Firebase로 계정연동을 하고 있는 서비스라 앱 외부에서 해당 계정과 연동시키는 부분이 난관이었음 → 결론적으로 앱 외부에서는 API 통신을 안하면서 현재 로그인 된 계정의 폴더를 보여줄 수 있어야 함
- 앱 외부와 앱 내부 사이에 데이터를 연동하기 위한 내부 DB를 구현하기로 결정
- Flutter에서 플랫폼 상관없이 각 네이티브에서 호출하는 DB를 제어하고 싶었음 (네이티브 코드 생략)

    ```dart
    class ShareDataProvider {
      static const _platform = MethodChannel('share_data_provider');
    
      static Future<List<Map<String, dynamic>>> getNewLinks() async {
        try {
          final newLinks = await _platform.invokeMethod('getNewLinks')
              as LinkedHashMap<Object?, Object?>;
    
          final links = <Map<String, dynamic>>[];
          for (final url in newLinks.keys) {
            final item =
                jsonDecode(newLinks[url].toString()) as Map<String, dynamic>;
            Log.i(item);
            final decoded = decodeBase64Text(item['title'] as String? ?? '');
            final shortTitle = getShortTitle(decoded);
            links.add({
              'url': url,
              'title': shortTitle,
              'describe': item['comment'],
              'image': item['image_link'],
              'folder_name': item['folder_name'],
              'created_at': item['created_at']
            });
          }
    
          return links;
        } on PlatformException catch (e) {
          Log.e(e.message);
          rethrow;
        }
      }
    
      static Future<List<Map<String, dynamic>>> getNewFolders() async {
        try {
          final newFolders =
              await _platform.invokeMethod('getNewFolders') as List<Object?>? ?? [];
    
          final result = <Map<String, dynamic>>[];
    
          for (final temp in newFolders) {
            final json = jsonDecode(temp!.toString()) as Map<String, dynamic>;
            final folder = {
              'name': json['name'],
              'visible': json['visible'],
              'created_at': json['created_at']
            };
            result.add(folder);
          }
    
          return result;
        } on PlatformException catch (e) {
          Log.e(e.message);
          rethrow;
        }
      }
    	// 생략...
    }
    ```

- 다른 계정으로 로그인하면 DB를 비우고 폴더 리스트를 가져오기 위한 정보를 서버에서 일괄적으로 불러와야함.
- iOS 네이티브 개발이 처음이었어서 DB 연동하고 Share Extension 에서 본앱으로 데이터 넘기는 과정이 구글링해도 잘 나오지 않아서 어려웠음 → **DB 파일 경로를 공유해서 동일한 DB 인스턴스를 참조하는 것이 핵심**

- 기타

  올해 초에 출시한 사이드 프로젝트이지만, 천천히 개선 중인 앱.

  아직 bloc 패턴에 익숙하지 않아 제대로 구현 못한게 많아서 계속 코드 수정도 병행하는 중

  원래 있던 PM이자 기획자이신 분이 식당 운영하느라 너무 바빠서 기획자 한분 더 섭외함


# flutter_tflite 사용해보기

## 숫자 인식시키기 실패

- 이거보고 따라하다가 실패: https://developer.android.com/codelabs/digit-classifier-tflite#0
- 아래 코드에서도 수많은 변화를 주면서 테스트해봤지만 interpreter가 결과를 안줌

```dart
import 'dart:ffi';

import 'dart:typed_data';
import 'package:flutter/material.dart';
import 'package:flutter/rendering.dart';
import 'package:tflite_flutter_examples/drawing_painter.dart';
import 'dart:ui' as ui;
import 'package:tflite_flutter/tflite_flutter.dart' as tfl;

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: MainScreen(),
    );
  }
}

class MainScreen extends StatefulWidget {
  const MainScreen({super.key});

  @override
  State<MainScreen> createState() => _MainScreenState();
}

class _MainScreenState extends State<MainScreen> {
  final _globalKey = GlobalKey();
  List<Offset?> _points = [];

  final outputs = [Float64List(10)];

  String resultValue = '';

  void _onPanUpdate(DragUpdateDetails details) {
    RenderBox? renderBox =
        _globalKey.currentContext?.findRenderObject() as RenderBox?;
    if (renderBox == null) return;
    Offset localPosition = renderBox.globalToLocal(details.globalPosition);
    setState(() {
      _points = List.from(_points)..add(localPosition);
    });
  }

  void _extractImage() async {
    final boundary =
        _globalKey.currentContext?.findRenderObject() as RenderRepaintBoundary?;
    if (boundary == null) return;
    ui.Image image = await boundary.toImage();

    final shape = interpreter.getInputTensor(0).shape;
    final width = shape[1];
    final height = shape[2];

    // image encode
    final pngBytes = await image.toByteData(format: ui.ImageByteFormat.png);
    final imageBytes = pngBytes?.buffer.asUint8List();
    if (imageBytes == null) return;
    // resize image by width and height
    final resizedImage = await resizeImage(imageBytes, width, height);
    if (resizedImage == null) return;

    classify(resizedImage);
  }
  Future<ByteBuffer?> resizeImage(Uint8List imageData, int width, int height) async {
    ui.Image image = await decodeImageFromList(imageData);
    ui.Image resizedImage = (await (await ui.instantiateImageCodec(
      Uint8List.fromList(imageData),
      targetWidth: width,
      targetHeight: height,
    )).getNextFrame()).image;
    final resizedByteData = await resizedImage.toByteData(format: ui.ImageByteFormat.png);
    return resizedByteData?.buffer;
  }

  void classify(ByteBuffer pngBytes) {
    interpreter.runInference(pngBytes.asUint8List());
    final output = interpreter.getOutputTensors();
    final result = outputs[0];
    final maxValue = result.reduce((curr, next) => curr > next ? curr : next);
    setState(() {
      resultValue = '$maxValue';
    });
  }

  late tfl.Interpreter interpreter;

  @override
  void initState() {
    WidgetsBinding.instance.addPostFrameCallback((_) async {
      interpreter = await tfl.Interpreter.fromAsset('assets/mnist.tflite');
    });
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            RepaintBoundary(
              key: _globalKey,
              child: GestureDetector(
                onPanUpdate: _onPanUpdate,
                onPanEnd: (_) => _points.add(null),
                child: Container(
                  width: 300,
                  height: 300,
                  decoration: BoxDecoration(
                    border: Border.all(color: Colors.black),
                  ),
                  child: CustomPaint(
                    painter: MyCustomPainter()..points = _points,
                  ),
                ),
              ),
            ),
            const SizedBox(height: 20),
            Text(resultValue),
            const SizedBox(height: 20),
            ElevatedButton(
              onPressed: _extractImage,
              child: const Text('Extract Image'),
            ),
          ],
        ),
      ),
    );
  }
}
```

## flutter_tflite 패키지에서 제공해준 예제들

https://pub.dev/packages/tflite_flutter

- 재밌게도 tensorflow 팀에서 처음 만든게 아니라 Amish Garg 라는 사람이 만든 패키지에 기반하고 있는게 공식 패키지가 된듯하다.

![img_9.png](/images/flutter/study_14th/week7/img_9.png)

- 이사람 github에 가면 다양한 예제 링크로 연결된다.

https://github.com/am15h/tflite_flutter_plugin#examples