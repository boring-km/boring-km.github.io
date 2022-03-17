---
layout: post
title: "Clean Code 6장 - 객체와 자료 구조"
categories: Clean_Code
tags: [java, dev]
toc: true
---

- 자료 추상화
- 자료/객체 비대칭
- 디미터 법칙
- 자료 전달 객체
- 결론

### 자료 추상화
- 구체적인 클래스와 추상적인 클래스

```java
// 구체적인 클래스
public class Point {
	public double x;
	public double y;
} 
// 추상적인 클래스
public interface Point {
	double getX();
	double getY();
	void setCartesian(double x, double y);
	double getR();
	double getTheta();
	void setPolar(double r, double theta);
}
```

- Point 인터페이스는 클래스 메서드가 접근 정책을 강제한다.
    - 좌표를 읽을 때는 각 값을 개별적으로 읽어야 한다.
    - 하지만 값을 설정할 때는 두 값을 한꺼번에 설정해야 한다.
- Point 클래스는 구현을 노출한다.
    - 역시 개별적으로 좌표값을 읽고 설정하도록 강제한다.
    - 변수를 private으로 설정한다고 하더라도 get, set 함수를 제공한다면 결국 구현을 외부로 노출하는 셈이다.
- 변수 사이에 함수라는 계층을 넣는다고 구현이 저절로 감춰지지 않는다.
- 형식에 치우쳐서 조회 함수와 설정 함수로 변수를 다룬다고 클래스가 되는 것이 아니다.
- **추상 인터페이스를 제공해 사용자가 구현을 모른 채 자료의 핵심을 조작할 수 있어야 진정한 의미의 클래스다.**

- 구체적인 클래스와 추상적인 클래스 또다른 예시

```java
// 구체적인 클래스
public interface Vehicle {
	double getFuelTankCapacityInGallons();
	double getGallonsOfGasoline();
}
// 추상적인 클래스
public interface Vehicle {
	double getPercentFuelRemaining();
}
```

- 구체적인 클래스는 연료 상태를 구체적인 숫자 값으로 알려준다.
- 추상적인 클래스는 자동차 연료 상태를 백분율이라는 추상적인 개념으로 알려준다.
    - **백분율 값으로 주기 때문에 클래스 내부에 어떤 데이터를 갖고 결과를 내보내는 지 감출 수 있다.**

### 자료/객체 비대칭
- **객체는 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개한다.**
- 자료 구조는 자료를 그대로 공개하며 별다른 함수는 제공하지 않는다.

***절차적인 도형***

```java
public class Square {
	public Point topLeft;
	public double side;
}

public class Rectangle {
	public Point topLeft;
	public double height;
	public double width;
}

public class Circle {
	public Point center;
	public double radius;
}

public class Geometry {
	public final double PI = 3.14159265;
	
	public double area(Object shape) throws NoSuchShapeException {
		if (shape instanceof Square) {
			Square s = (Square)shape;
			return s.side * s.side;
		}
		else if (shape instanceof Rectangle) {
			Rectangle r = (Rectangle)shape;
			return r.height * r.width;
		}
		else if (shape instanceof Circle) {
			Circle c = (Circle)shape;
			return PI * c.radius * c.radius;
		}
		throw new NoSuchShapeException();
	}
}
```

- Geometry 클래스에 둘레 길이를 구하는 perimeter() 함수를 추가한다고 할 때는 도형 클래스에 아무 영향이 없다. 도형 클래스에 의존하는 다른 클래스도 마찬가지다.
- 하지만 새 도형을 추가하게 된다면 Geometry 클래스에 속한 함수를 모두 수정해야 한다. (area() 함수에서 새로운 도형에 대한 로직을 작성하고, perimeter() 함수 역시 추가가 된다.)

***객체 지향적인 도형***

```java
// 오버라이드 어노테이션 추가함
public class Square implements Shape {
	private Point topLeft;
	private double side;

	@Override
	public double area() {
		return side*side;
	}
}

public class Rectangle implements Shape {
	private Point topLeft;
	private double height;
	private double width;

	@Override
	public double area() {
		return height * width;
	}
}

public class Circle implements Shape {
	private Point center;
	private double radius;
	public final double PI = 3.14159265;

	public double area() {
		return PI * radius * radius;
	}
}
```

- 절차적인 도형과 달리 Geometry 클래스가 없어도 각각의 도형 클래스에서 area() 메서드를 갖고 있다.(polymorphic 메서드)
- 그러므로 새로운 도형을 추가해도 기존 함수에 아무런 영향을 미치지 않는다.
- 하지만, 새로운 함수를 추가하고 싶다면 모든 도형 클래스를 고쳐야 한다.

#### 정리
- 객체 지향 코드에서 어려운 변경은 절차적인 코드에서 쉽고, 절차적인 코드에서 어려운 변경은 객체 지향 코드에서 쉽다.
- 상황에 따라서 적합한 방법을 선택해야 한다.
- **모든 것이 객체라는 생각은 미신이다.**
- **때로는 단순한 자료 구조와 절차적인 코드가 가장 적합한 상황도 있다.**

### 디미터 법칙
- **모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다.**
- **객체는 조회 함수로 내부 구조를 공개하면 안 된다.**
- 디미터 법칙에서 말하는 메서드 호출이 가능한 객체

```text
“클래스 C의 메서드 f는 아래의 객체의 메서드만 호출해야 한다”
- 클래스 C
- f가 생성한 객체
- f 인수로 넘어온 객체
- C 인스턴스 변수에 저장된 객체
```

***기차 충돌***

```java
// 이렇게 작성하지 말자
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();

// 이렇게 나누자
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```

- ctxt, Options, ScratchDir이 객체라면 내부 구조를 숨겨야 하므로 확실히 디미터 법칙을 위반한다.
- 하지만 자료 구조라면 당연히 내부 구조를 노출하므로 디미터 법칙이 적용되지 않는다.

#### 잡종 구조
- 이런 혼란으로 말미암아 때때로 절반은 객체, 절반은 자료 구조인 잡종 구조가 나온다.
- 잡종 구조는 중요한 기능을 수행하는 함수도 있고, 공개 변수나 공개 조회/설정 함수도 있다.
- 공개 조회/설정 함수는 비공개 변수를 그대로 노출한다.
- 그래서 다른 함수가 절차적인 프로그래밍의 자료 구조 접근 방식처럼 비공개 변수를 사용하고픈 유혹에 빠지기 십상이다.
- **새로운 함수는 물론이고 새로운 자료 구조도 추가하기 어렵다.**
- **양쪽 세상(객체, 자료 구조)에서 단점만 모아놓은 구조다**
- 되도록 피하자.
- 프로그래머가 함수나 타입을 보호할지 공개할지 확신하지 못해 어중간하게 내놓은 설계에 불과하다.

#### 구조체 감추기
- 만약에 ctxt가 자료구조가 아닌 진짜 객체라면 위에서처럼 내부 구조를 드러내면 안된다.
- 객체에게 **뭔가를 하라고** 말해야지 속을 드러내라고 말하면 안 된다.
- 절대 경로(absolutePath)가 왜 필요할지 모듈 내부를 찾아서 임시 파일을 생성하기 위한 목적이라는 사실을 발견한다. (클린코드 교재의 상황에서)
- ctxt 객체에게 임시 파일을 생성하라고 직접 시켜본다.

```java
BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
```

- 더 이상 ctxt는 내부 구조를 드러내지 않으며, 모듈에서 해당 함수는 자신이 몰라야 하는 여러 객체를 탐색할 필요가 없다. (**디미터 법칙을 위반하지 않는다.**)

### 자료 전달 객체
- 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스다.
- 흔히 데이터베이스에 저장된 가공되지 않은 정보를 애플리케이션 코드에서 사용할 객체로 변환하는 일련의 단계에서 가장 처음으로 사용하는 구조체다.
- (교재 예시는 private 인스턴스 변수와, 생성자, getter() 함수가 적혀있다.)

#### 활성 레코드
- DTO의 특수한 형태다. (**자료 구조**)
- 공개 변수가 있거나 비공개 변수에 조회/설정 함수가 있는 자료 구조지만, 대개 save나 find와 같은 탐색 함수도 제공한다.
- 데이터베이스 테이블이나 다른 소스에서 자료를 직접 변환한 결과다.
- 활성 레코드에 비즈니스 규칙 메서드를 추가해 이런 자료 구조를 객체로 취급하지 말아야 한다. (**잡종 구조**)
- **비즈니스 규칙을 담으면서 내부 자료를 숨기는 객체는 따로 생성한다.**
    - 내부 자료가 활성 레코드의 인스턴스일 가능성이 높다!

### 결론
- 객체는 동작을 공개하고 자료를 숨긴다.
    - 그래서 기본 동작을 변경하지 않으면서 새 객체 타입을 추가하기는 쉬운 반면, 기존 객체에 새 동작을 추가하기는 어렵다.
- 자료 구조는 별다른 동작 없이 자료를 노출한다.
    - 그래서 기존 자료 구조에 새 동작을 추가하기는 쉬우나, 기존 함수에 새 자료 구조를 추가하기는 어렵다.
- **시스템을 구현할 때, 새로운 자료 타입을 추가하는 유연성이 필요하면 객체가 더 적합하다.**
- **새로운 동작을 추가하는 유연성이 필요하면 자료 구조와 절차적인 코드가 더 적합하다.**
- **편견 없이 이 사실을 이해해 최적의 해결책을 선택하자**
