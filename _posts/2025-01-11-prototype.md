---
title: GoF Design Pattern - Prototype
date: 2025-01-11 03:00:00
categories: [Design pattern, Creational pattern]
tags: [design pattern, software design, creational pattern, prototype]
render_with_liquid: false
---

## 정의

프로토타입은 코드를 그들의 클래스들에 의존시키지 않고 기존 객체들을 복사할 수 있도록 하는 생성 디자인 패턴이다.

## 구성 요소

![image](https://refactoring.guru/images/patterns/diagrams/prototype/structure-2x.png)

+ Prototype 인터페이스: 복제를 위한 `clone()` method 정의.

+ ConcretePrototype 클래스: `Prototype` 인터페이스를 구현하고, 자신의 복제 로직을 제공.

+ Client: `clone()` method를 호출하여 새로운 객체 생성.

## 예제

예제 코드:

```java
// Prototype 인터페이스
interface Shape {
    Shape clone(); // 복제를 위한 메서드
    void draw();   // 도형을 그리는 메서드
}

// 구체적인 프로토타입 클래스: Circle
class Circle implements Shape {
    private String color;

    // 생성자
    public Circle(String color) {
        this.color = color;
    }

    // 복제 메서드 구현
    @Override
    public Shape clone() {
        return new Circle(this.color); // 현재 객체의 색상을 복사하여 새 객체 생성
    }

    // 도형 그리기 메서드 구현
    @Override
    public void draw() {
        System.out.println("Drawing a " + color + " circle.");
    }
}

// 구체적인 프로토타입 클래스: Rectangle
class Rectangle implements Shape {
    private String color;

    // 생성자
    public Rectangle(String color) {
        this.color = color;
    }

    // 복제 메서드 구현
    @Override
    public Shape clone() {
        return new Rectangle(this.color); // 현재 객체의 색상을 복사하여 새 객체 생성
    }

    // 도형 그리기 메서드 구현
    @Override
    public void draw() {
        System.out.println("Drawing a " + color + " rectangle.");
    }
}

// 실행 클래스 (Main)
public class PrototypeExample {
    public static void main(String[] args) {
        // 원본 프로토타입 생성 (빨간색 원과 파란색 사각형)
        Shape redCircle = new Circle("red");
        Shape blueRectangle = new Rectangle("blue");

        // 프로토타입을 이용해 새로운 객체 생성 (복제)
        Shape clonedCircle = redCircle.clone();
        Shape clonedRectangle = blueRectangle.clone();

        // 원본 및 복제된 객체 사용
        System.out.println("원본 객체:");
        redCircle.draw();       // 출력: Drawing a red circle.
        blueRectangle.draw();   // 출력: Drawing a blue rectangle.

        System.out.println("\n복제된 객체:");
        clonedCircle.draw();     // 출력: Drawing a red circle.
        clonedRectangle.draw();  // 출력: Drawing a blue rectangle.
    }
}
```
출력 결과:

```text
원본 객체:
Drawing a red circle.
Drawing a blue rectangle.

복제된 객체:
Drawing a red circle.
Drawing a blue rectangle.
```

## 특징

적용 시기

1. 프로토타입 패턴은 복사해야 하는 객체들의 구상 클래스들에 코드가 의존하면 안 될 때 사용한다.

2. 프로토타입 패턴은 각자의 객체를 초기화하는 방식만 다른 자식 클래스들의 수를 줄이고 싶을 때 사용한다.

장점

+ 객체들을 그 구상 클래스들에 결합하지 않고 복제할 수 있다.

+ 반복되는 초기화 코드를 제거한 후 그 대신 미리 만들어진 프로토타입들을 복제하는 방법을 사용할 수 있다.

+ 복잡한 객체들을 더 쉽게 생성할 수 있다.

+ 복잡한 객체들에 대한 사전 설정들을 처리할 때 상속 대신 사용할 수 있는 방법이다.

단점

+ 순환 참조가 있는 복잡한 객체들을 복제하는 것은 매우 까다로울 수 있습니다.