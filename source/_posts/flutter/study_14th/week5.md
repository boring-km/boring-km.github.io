---
layout: post
title: "Flutter 중급반 스터디 14기 5주차"
categories: flutter-study-14th
date: 2023-07-24
tags: [dev, flutter]
toc: true
---

# 닷지 게임 길어지는 선 Enemy 구현하다가 실패

![img.png](/images/flutter/study_14th/week5/img.png)

- 선이 닿으면 충돌을 감지해야하는데 Hitbox 설정을 제대로 못함
- 선이 좌측에서 우측 하단으로 움직일 때만 그려짐

→  선 전체를 Hitbox로 정하기 보다는 이동하는 구체에만 히트박스를 달고, 뒤에 꼬리가 있는 느낌으로 수정해야 할 것 같음

# 길이가 자기 맘대로인 PageView의 Indicator 만들기

- 사실 길이가 디바이스 화면보다 커지면 PageView를 사용할 수가 없다. ㅠㅠ
- CustomScrollView와 SliverList로 각각 다른 페이지의 길이를 기억시키고 Indicator가 눌렸을 때 그 길이 만큼 이동하게 해준다.

![img_1.png](/images/flutter/study_14th/week5/img_1.png)

```dart
final widthsNotifier = ValueNotifier([0, 0, 0]);

	WidgetsBinding.instance.addPostFrameCallback((timeStamp) async {
      scrollController.addListener(() {
        final currentWidth = scrollController.position.pixels;
        var totalWidth = 0;

        for (var i = 0; i < widthsNotifier.value.length; i++) {
          totalWidth += widthsNotifier.value[i];
          if (currentWidth < totalWidth) {
            pageNotifier.value = i;
            break;
          }
        }
      });
			// 끝까지 스크롤해야 모든 페이지의 가로 길이를 얻을 수 있음
			await scrollController.animateTo(
			  scrollController.position.maxScrollExtent,
			  duration: const Duration(milliseconds: 400),
			  curve: Curves.easeInOut,
			);
			await scrollController.animateTo(
			  0,
			  duration: const Duration(milliseconds: 400),
			  curve: Curves.easeInOut,
			);
			// ... 화면 왔다갔다 하는 동작 로딩으로 감추기
	});

// Pages
Expanded BodyPages() {
  return Expanded(
    child: CustomScrollView(
      controller: scrollController,
      scrollDirection: Axis.horizontal,
      slivers: <Widget>[
        SliverList(
          delegate: SliverChildBuilderDelegate(
            (context, index) {
              switch (PageIndex.from(index)) {
                case PageIndex.first:
                  return FirstPage(
                    widthCallback: (width) {
                      widthsNotifier.value[0] = width.toInt();
                    },
                  );
                case PageIndex.second:
                  return SecondPage(
                    widthCallback: (width) {
                      widthsNotifier.value[1] = width.toInt();
                    },
                  );
                case PageIndex.third:
                  return ThirdPage(
                    widthCallback: (width) {
                      widthsNotifier.value[2] = width.toInt();
                    },
                  );
              }
            },
            childCount: 3,
          ),
        ),
      ],
    ),
  );
}
```

→ widthsNotifier

### First Page 예시: GlobalKey를 포함한 Container가 그려지고 나서 해당 위젯의 가로 길이를 반환

```dart
class FirstPage extends StatefulWidget {
  const FirstPage({required this.widthCallback, super.key});

  final void Function(double) widthCallback;

  @override
  State<FirstPage> createState() => _FirstPageState();
}

class _FirstPageState extends State<FirstPage> {
  final key = GlobalKey();

  @override
  void initState() {
    sendWidgetWidth(key, widget.widthCallback);
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      key: key,
      width: 2000.w,
      color: Colors.lightGreen,
      child: Stack(
        children: [
          Align(
            alignment: Alignment.centerLeft,
            child: Text('left', style: TextStyle(fontSize: 30.sp),),
          ),
          Center(
            child: Text('center', style: TextStyle(fontSize: 30.sp),),
          ),
          Align(
            alignment: Alignment.centerRight,
            child: Text('right', style: TextStyle(fontSize: 30.sp),),
          ),
        ],
      ),
    );
  }
}

// 가로 길이 콜백 전달
void sendWidgetWidth(GlobalKey<State<StatefulWidget>> key, void Function(double width) widthCallback) {
  WidgetsBinding.instance.addPostFrameCallback((timeStamp) {
    if (key.currentContext != null) {
      final r = key.currentContext!.findRenderObject() as RenderBox?;
      if (r != null) {
        widthCallback(r.size.width);
      }
    }
  });
}
```

### Indicator

- 선택한 index 이전까지의 가로길이를 합친만큼 이동하면 된다.
- 마지막 index의 1/4 가로길이 만큼 더 이동해주면 중앙으로 갈 수 있다. (화면 중앙으로 와야하니깐)

```dart
SizedBox TopTabBar() {
  return SizedBox(
    height: 100,
    child: ListView.builder(
      shrinkWrap: true,
      scrollDirection: Axis.horizontal,
      itemCount: 3,
      itemBuilder: (context, index) {
        return ValueListenableBuilder(
          valueListenable: pageNotifier,
          builder: (context, page, child) {
            final isSelected = page == index;
            return GestureDetector(
              onTap: () {
                var selectedPagePosition = 0;
                for (var i = 0; i < index; i++) {
                  selectedPagePosition += widthsNotifier.value[i];
                }
                // move to each center
                // selectedPagePosition +=
                //     widthsNotifier.value[index] ~/ 4;
                scrollController.animateTo(
                  selectedPagePosition.toDouble(),
                  duration: const Duration(milliseconds: 500),
                  curve: Curves.easeInOut,
                );
              },
              child: switch (PageIndex.from(index)) {
                PageIndex.first => FirstTab(isSelected),
                PageIndex.second => SecondTab(isSelected),
                PageIndex.third => ThirdTab(isSelected),
              },
            );
          },
        );
      },
    ),
  );
}
```

# 커스텀 달력 좌우 넘기기

![img_2.png](/images/flutter/study_14th/week5/img_2.png)

→ 아직 다음달/이전달로 이동할 때, 달력에서 현재 달의 데이터를 중복해서 그리고 있음 → 구조 개선 필요함

![img_3.png](/images/flutter/study_14th/week5/img_3.png)

# Shorebird Android 확인해보기

- docs: https://docs.shorebird.dev/
- flutter flavor: [https://velog.io/@udong85/Flutter-flavor-를-이용한-개발-운영-환경-설정](https://velog.io/@udong85/Flutter-flavor-%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EA%B0%9C%EB%B0%9C-%EC%9A%B4%EC%98%81-%ED%99%98%EA%B2%BD-%EC%84%A4%EC%A0%95)

![img_4.png](/images/flutter/study_14th/week5/img_4.png)

→ 1.0.2+2 버전에서 patch만 진행해보기