---
layout: post
title: "Flutter InAppWebView iOS 16.4 Inspecting 안되는 이슈"
categories: flutter
tags: [dev, flutter]
toc: true
---

- 원문 링크: https://github.com/pichillilorenzo/flutter_inappwebview/issues/1629

iOS 버전이 16.4로 업데이트 되면서 WKWebView.isInspectable 값을 true로 설정해야만 Safari를 통해 웹뷰 디버깅이 가능하다.

근데 아직 InAppWebView 베타버전 라이브러리에서도 작업이 안되어 있어서 이슈를 적어봤다.

일단은 임시로 Pod 폴더에서 InAppWebView.swift 파일을 직접 수정해서 옵션을 넣어주는 수 밖에 없을 것 같다.

