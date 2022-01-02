---
layout: post
title: "PhotoSchool App"
categories: flutter
tags: [dev, flutter]
author:
- Kangmin
---

### 포토스쿨 SNS App
- 웅진씽크빅 인턴 과제로 진행한 프로젝트
- 진행 기간: 2달 이내
- TIL에 남긴 이력
  - 기획: https://boring-km.github.io/TIL/#/./TIL/2021/210610
  - 구조: https://boring-km.github.io/TIL/#/./TIL/2021/210907
- 실제로 구현하면서 회사 서비스와 연동하는 부분이 좀 생겼고, 기능도 많이 추가가 된 편이다.

### 관련 코드
- 현재는 내부서버 닫혀있어서 사용 못함 (띄울 수 있는 ec2 인스턴스가 없음 ㅠㅠ)
- 애초에 회사 검증 서버라서 외부에서 이용도 못함
- 서버: https://github.com/boring-km/photoschool-server
- 앱: https://github.com/boring-km/photoschool-app
  - Web 부분은 dart2js를 사용해서 별도로 배포했기 때문에 프로젝트는 앱 쪽에 있다. 

### 후기
- Flutter로 프로젝트를 본격적으로 만든 첫번째 앱이라 봐도 무방한 느낌
- UI적으로 다양한 시도(?)를 해본 것 같음
- 아직 Flutter 프로젝트의 구조에 대한 이해가 부족해서 코드가 엉망인 부분이 많았다
- GCP에 있는 무료 서비스들이 많아 생각보다 손쉽게 해결한 기능들도 많았던 것 같다.
- 개발하면서 Flutter Web이 본격적으로 서비스를 시작했는데, 생각보다 앱에서 쓰이는 부분과 웹은 많이 다른 느낌이다.