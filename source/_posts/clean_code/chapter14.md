---
layout: post
title: "Clean Code 14장 - 점진적인 개선"
categories: Clean_Code
date: 2022-01-26
tags: [java, dev]
toc: true
---

- (소개)
- Args 구현
- Args 1차 초안
- String 인수
- 결론

### (소개)
- 춞발은 좋았으나 확장성이 부족했던 모듈 소개
- 개선하고 정리하는 단계를 거쳐보자
- 명령행 인수의 구문을 분석하는 새로운 유틸리티 *Args*

#### 간단한 예시

```java
public class Main {
  public static void main(String[] args) {
    try {
        Args arg = new Args("l,p#,d*", args);
        boolean logging = arg.getBoolean('l');
        int port = arg.getInt('p');
        String directory = arg.getString('d');
        executeApplication(logging, port, directory);
    } catch (ArgsException e) {
        System.out.println("Argument error: %s\n", e.errorMessage());
    }
  }
}
```

- l은 부울 인수, p는 정수 인수, d는 문자열 인수
- 생성자에서 ArgsException이 발생하지 않는다면 명령행 인수의 구문을 성공적으로 분석했으며 Args 인스턴스에 질의를 던져도 좋다는 말이다.
- 인수 값을 가져오려면 getBoolean, getInteger, getString 등과 같은 메서드를 사용한다.
- 형식 문자열이나 명령행 인수 자체에 문제가 있다면 ArgsException이 발생한다.
- 구체적인 오류를 알아내려면 예외가 제공하는 errorMessage 메서드를 사용한다.

## Args 구현
- chapter14 package
- 깨끗한 코드를 짜려면 먼저 지저분한 코드를 짠 **뒤에 정리해야 한다**는 의미다.
- 먼저 1차 초안을 쓰고, 그 초안을 고쳐 2차 초안을 만ㄷ르고, 계속 고쳐 최종안을 만들자
- '돌아가는' 프로그램에서 멈추지 말라
  - **경험이 풍부한 전문 프로그래머라면 이런 행동이 전문가로서 자살 행위라는 사실을 잘 안다.**

## Args 1차 초안
- 코드는 '돌아가지만' 엉망인 상태
- 아마 실제 코드가 이 상태에 머물러 있는 프로젝트들이 아주 많을 것이다.

## String 인수
- 기능 추가

## 결론
- 그저 돌아가는 코드만으로는 부족하다. 돌아가는 코드가 심하게 망가지는 사례는 흔하다.
- 단순히 돌아가는 코드에 만족하는 프로그래머는 전문가 정신이 부족하다.
- 설계와 구조를 개선할 시간이 없다는 변명은 하지를 말자.
- 나쁜 코드보다 더 오랫동안 더 심각하게 개발 프로젝트에 악영향을 미치는 요인도 없다.

> 나쁜 일정은 다시 짜면 된다.
> 
> 나쁜 요구사항은 다시 정의하면 된다.
> 
> 나쁜 팀 역학은 복구하면 된다.
> 
> **하지만 나쁜 코드는 썩어 문드러진다.**

- 점점 무게가 늘어나 팀의 발목을 잡는다.
- 너무 서두르다가 이후로 영원히 자신들의 운명을 지배할 악성 코드라는 굴레를 짊어진다.
- 물론 나쁜 코드를 깨끗한 코드로 개선할 수 있지만 비용이 엄청나게 많이 든다.
- 오래된 의존성을 찾아내 깨려면 상당한 시간과 인내심이 필요하다.
- 반면 처음부터 코드를 깨끗하게 유지하기란 상대적으로 쉽다.
  - 아침에 엉망으로 만든 코드는 오후에 정리하기에 어렵지 않다.
  - 5분 전이면 더더욱 쉽다.
- 그러므로 **코드는 언제나 최대한 깔끔하고 단순하게 정리하자.** 절대로 썩어가게 방치하면 안 된다.