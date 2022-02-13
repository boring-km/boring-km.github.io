---
layout: post
title: "Flutter에서 안드로이드 함수 사용하기"
date: 2021-12-21
categories: flutter
tags: [dev, flutter]
toc: true
---

- android 폴더 안에 있는 MainActivity 클래스가 FlutterActivity라는 걸 상속하고 있는데 이걸 통해 구현을 할 것이다.

### 예제 작성해보기
- 연습해 볼 코드는 simpletodo 라는 이름으로 개발 중인 단순한 앱에 넣어본다.

> 먼저 안드로이드에서 flutter와 같은 CHANNEL 값으로 맞춰준다.

> method 이름이 같지만 기능이 달라야 할 다른 채널을 위한 게 아닐까 싶다.

```kotlin
import io.flutter.embedding.android.FlutterActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugin.common.MethodChannel

class MainActivity: FlutterActivity() {

    companion object {
        private const val CHANNEL = "flutter.android.channel"
    }

    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)
        MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL)
            .setMethodCallHandler { call, result ->
                run {
                    if (call.method == "getMyCacheDirectory") {
                        // 이 부분은 아예 call.method 값과 동일하게 함수로 묶어주면 많아졌을 때 보기 깔끔하겠다.
                        val path = applicationContext.cacheDir.absolutePath
                        val data = HashMap<String, Any?>()
                        data.put("path", path)
                        result.success(data)
                    } else {
                        result.error("errorCode", "errorMessage", "errorDetail")
                    }
                }
            }
    }
}
```

> flutter에서는 원하는 코드에서 선언 되어 있는 method를 불러오면 되겠다.

```dart
import 'package:flutter/services.dart';
import 'package:get/get.dart';
import 'package:simpletodo/utils/Log.dart';

class MainController extends GetxController {
  // ...

  final platform = MethodChannel('flutter.android.channel');

  getMyCacheDirectory() async {
    final result = await platform.invokeMethod("getMyCacheDirectory");
    Log.d('myCacheDir: $result');
  }
}
```

- 이런 식으로 구현하면 응답값을 얻을 수 있겠다.

#### 관련 문서
- https://docs.flutter.dev/development/platform-integration/platform-channels