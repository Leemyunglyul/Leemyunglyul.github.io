---
title: Factory Method
date: 2025-01-11 02:31:00
categories: [Design pattern, Creational pattern]
tags: [design pattern, software design, creational pattern, factory method]
render_with_liquid: false
---

## 정의

팩토리 메서드 패턴은 객체 생성에 관한 디자인 패턴으로, 객체 생성의 책임을 서브클래스에 위임하여 유연성과 확장성을 높이는 데 중점을 둔다. 이 패턴은 객체 생성 코드를 캡슐화하여 클라이언트 코드가 구체적인 클래스에 의존하지 않도록 설계한다. 부모 클래스에서 객체들을 생성할 수 있는 인터페이스를 제공하지만, 자식 클래스들이 생성될 객체들의 유형을 변경할 수 있도록 하는 생성 패턴이다.

## 구성 요소

<img src='../assets/img/post/design pattern/factory_method_structure.png' alt = "image">

다음의 사진이 팩토리 메서드를 잘 설명하고 있다.

+ Product: 생성할 객체의 공통 인터페이스 또는 추상 클래스

+ ConcreteProduct: Product 인터페이스를 구현할 구체적인 클래스

+ Creator/Factory: Product 객체를 반환하는 팩토리 메서드를 선언한 클래스.

+ ConcreteCreator/ConcreteFactory: 팩토리 메서드를 오버라이드하여 구체적인 Product 객체를 생성하는 클래스.

<img src='../assets/img/post/design pattern/factory-method-diagram.webp' alt = "image">

실제 흐름은 위의 이미지처럼 ConcreteFactory에서 ConcreteProduct을 생성한다.

## 예제

예제 코드:

```java
// Product 인터페이스
interface Shape {
    void draw();
}

// ConcreteProduct 클래스들
class Circle implements Shape {
    public void draw() {
        System.out.println("Drawing a Circle");
    }
}

class Rectangle implements Shape {
    public void draw() {
        System.out.println("Drawing a Rectangle");
    }
}

// Creator 클래스
abstract class ShapeFactory {
    public abstract Shape createShape();
}

// ConcreteCreator 클래스들
class CircleFactory extends ShapeFactory {
    public Shape createShape() {
        return new Circle();
    }
}

class RectangleFactory extends ShapeFactory {
    public Shape createShape() {
        return new Rectangle();
    }
}

// Client 코드
public class FactoryMethodExample {
    public static void main(String[] args) {
        ShapeFactory circleFactory = new CircleFactory();
        Shape circle = circleFactory.createShape();
        circle.draw();

        ShapeFactory rectangleFactory = new RectangleFactory();
        Shape rectangle = rectangleFactory.createShape();
        rectangle.draw();
    }
}
```

실행 결과: 

```text
Drawing a Circle
Drawing a Rectangle
```

## 특징

적용 시기

1. 팩토리 메서드는 당신의 코드가 함께 작동해야 하는 객체들의 정확한 유형들과 의존관계들을 미리 모르는 경우 사용한다.

2. 팩토리 메서드는 당신의 라이브러리 또는 프레임워크의 사용자들에게 내부 컴포넌트들을 확장하는 방법을 제공하고 싶을 때 사용한다.

3. 팩토리 메서드는 기존 객체들을 매번 재구축하는 대신 이들을 재사용하여 시스템 리소스를 절약하고 싶을 때 사용한다.

장점

+ 크리에이터와 구상 제품들이 단단하게 결합되지 않도록 할 수 있다.

+ 단일 책임 원칙(SRP). 제품 생성 코드를 프로그램의 한 위치로 이동하여 코드를 더 쉽게 유지관리할 수 있다.

+ 개방/폐쇄 원칙(OCP). 당신은 기존 클라이언트 코드를 훼손하지 않고 새로운 유형의 제품들을 프로그램에 도입할 수 있다.

단점

+ 패턴을 구현하기 위해 많은 새로운 자식 클래스들을 도입해야 하므로 코드가 더 복잡해질 수 있다. 가장 좋은 방법은 크리에이터 클래스들의 기존 계층구조에 패턴을 도입하는 것이다.

<https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%ED%8C%A9%ED%86%A0%EB%A6%AC-%EB%A9%94%EC%84%9C%EB%93%9CFactory-Method-%ED%8C%A8%ED%84%B4-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EC%9E%90>

자료는 다음 블로그를 참고해서 작성했다. 워낙 잘 설명되어 있어 여기를 참고하는 게 좋다.


