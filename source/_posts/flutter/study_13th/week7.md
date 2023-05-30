---
layout: post
title: "Flutter 중급반 스터디 13기 7주차"
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
- [x]  6주차: 이미지 Custom Crop Plugin 개발
- [x]  7주차: 이미지 Custom Crop Plugin 개발
- [ ]  8주차: 이미지 Custom Crop Plugin 개발

# 지난주에 빠트렸던 crop 패키지

[image_crop | Flutter Package](https://pub.dev/packages/image_crop)

제일 내가 만들고 싶은 기능과 흡사함

예제가 확대된 채 움직여서 crop을 해야하는게 조금 불편했음

이 패키지 안에 있는 함수 기능이 꼭 필요해서 dart 코드로만 작성할 수 있을지 검토 중

```dart
  static Future<File> cropImage({
    required File file,
    required Rect area,
    double? scale,
  }) =>
      _channel.invokeMethod('cropImage', {
        'path': file.path,
        'left': area.left,
        'top': area.top,
        'right': area.right,
        'bottom': area.bottom,
        'scale': scale ?? 1.0,
      }).then<File>((result) => File(result));
```

# 어려움

- 외부 패키지 없이 기능을 완성하고 싶은데 위에 적혀있는 저 함수를 dart로 작성하는 방법을 아직 찾지 못함 → 다른 패키지 기능 더 살펴보고 다음주까지 알아내보기
    - crop_image에서 가능한 것 같아서 좀 더 살펴보기
    - (https://pub.dev/packages/crop_image)

### crop_image
- https://github.com/deakjahn/crop_image/blob/master/lib/src/crop_controller.dart
- 아래의 함수를 참고해서 dart 코드로 작성해보기

```dart
import 'dart:ui' as ui;

static Future<ui.Image> getCropImage({
  required Rect crop,
  required ui.Image image,
  required ShapeType shapeType,
  required CropArea area,
}) async {
  final pictureRecorder = ui.PictureRecorder();
  final canvas = Canvas(pictureRecorder);
  
  final cropWidth = crop.width * image.width; // 실제로 crop할 이미지의 width
  final cropHeight = crop.height * image.height; // 실제로 crop할 이미지의 height
  
  final cropCenter = Offset(    // 실제로 crop할 이미지의 중심좌표
    image.width.toDouble() * crop.center.dx,
    image.height.toDouble() * crop.center.dy,
  );
  
  // ShapeType에 따라서 다른 Painter를 사용
  if (shapeType == ShapeType.rectangle) {
    RectanglePainterForCrop(
      Rect.fromLTWH(area.left, area.top, cropWidth, cropHeight),
      cropCenter,
      image,
    ).paint(canvas, Size(cropWidth, cropHeight));
  } else if (shapeType == ShapeType.circle) {
    CirclePainterForCrop(
      Rect.fromLTWH(area.left, area.top, cropWidth, cropHeight),
      cropCenter,
      image,
    ).paint(canvas, Size(cropWidth, cropHeight));
  } else if (shapeType == ShapeType.triangle) {
    TrianglePainterForCrop(
      Rect.fromLTWH(area.left, area.top, cropWidth, cropHeight),
      cropCenter,
      image,
    ).paint(canvas, Size(cropWidth, cropHeight));
  } else {
    throw Exception('Unknown shape type');
  }
}

```

- 패키지 사용이 최대한 단순하도록 계속 수정중

# 주간 회고

- 앱 출시하고 조금 휴식..?
- 자유형식 경력 이력서 처음 작성해 봄
- flutter_inappwebview 이슈에 적은 글이 반영됨
    - https://github.com/pichillilorenzo/flutter_inappwebview/issues/1629