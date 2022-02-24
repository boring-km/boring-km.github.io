---
layout: post
title: "StatefulWidget vs StatelessWidget"
categories: flutter-ui
tags: [dev, flutter]
toc: true
---

- 각각이 어떤 위젯이었는지 까먹었으면 [이쪽](https://boring-km.github.io/2022/02/08/flutter/ui/flutter-ui1/#StatefulWidget-StatelessWidget-Scaffold)
- 구글링하면서 찾은 다른 포스팅
  - https://here4you.tistory.com/220
  - https://velog.io/@dosilv/Flutter-StatelessWidget-StatefulWidget
  - https://security-nanglam.tistory.com/478

### 개인적인 생각
- 외부에 의해서만 변경되는 위젯이면 StatelessWidget이 깔끔하다고 생각합니다.
- 내부에서도 스스로 UI 변경이 자주 일어나는 위젯이면 StatefulWidget이 적합합니다.
- 의존성 주입 플러그인을 사용하다보면 StatefulWidget을 사용할 경우가 거의 없었습니다.
  - 웬만하면 StatelessWidget으로 거의 가능한 것 같습니다.
- 다만 모든 위젯을 StatelessWidget으로 만들려고만 하지 않으면 되겠습니다. (Stateful이 적합한 위젯은 분명히 있습니다.)
- UI를 그리다가 너무 코드가 길어져서 코드를 다시 찾아가기가 힘들어질때 UI를 별도의 StatelessWidget으로 분리시키는 리팩토링을 고려해보면 좋을 것 같습니다.