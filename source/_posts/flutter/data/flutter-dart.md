---
layout: post
title: "Dart 문법 사용 시 주의사항"
categories: flutter-data
tags: [dev, flutter]
toc: true
---

- Flutter 앱을 개발하면서 Dart 언어를 사용할 때 몇가지 주의해야 할 내용들에 대해 다루겠습니다.
- Dart 문법 관련 문서: https://dart.dev/guides/language/language-tour#a-basic-dart-program

### dynamic
- Dart에서 method 선언 시 리턴 타입을 입력하지 않으면 기본적으로 dynamic 타입으로 리턴하게 됩니다.
- dynamic은 null 여부를 확인해야 할 뿐만 아니라 어떤 데이터 타입이 반환될 지 알 수 없는 상황에서는 사용을 피하는 것이 좋겠습니다.
  (개인적으로 여러 데이터 타입으로 반환해야 하는 경우라면 설계를 변경하거나 객체화하는 것이 좋겠습니다.)
- IDE의 code snippet 기능을 활용하기에도 타입이 지정되어 있는 편이 개발이 편합니다.

### interface
- 문서: https://dart.dev/guides/language/language-tour#implicit-interfaces
- class를 interface처럼 사용할 수 있습니다.
- abstract class를 사용해도 좋습니다.

### null-safety
- 문서: https://dart.dev/null-safety/understanding-null-safety
- 간혹 사용하고자 하는 라이브러리가 오래되어 null-safety 지원이 안 될 수 있습니다.
- flutter 실행 시 '--no-sound-null-safety' 옵션을 추가해주세요.

### late
- late 키워드로 선언한 변수는 적어도 객체 생성이 끝나기 전에 초기화 해야 합니다.
- 만약 비동기로 초기화 될 데이터가 있다면 'Future<타입>' 으로 선언하거나 '?' 키워드를 사용해 먼저 null로 초기화 해주세요.

### new
- 과거 flutter 코드를 찾아보면 new 키워드를 사용하는 코드가 있는데, 현재는 사용할 필요 없이 객체 생성을 해도 아무 문제가 없습니다.

### 문자열
- dart linter에서는 기본적으로 문자열 초기화 시 작은 따옴표를 권장합니다.

### try-catch stacktrace
- 어쩔 수 없이 try-catch를 사용해야 하는 상황에서 에러 코드의 stacktrace를 얻으려면 'catch(error, stacktrace)'를 통해 두 번째 파라미터에 오는 값을 사용하면 됩니다.
