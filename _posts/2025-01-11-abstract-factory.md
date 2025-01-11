---
title: GoF Design Pattern - Abstract Factory
date: 2025-01-11 02:34:00
categories: [Design pattern, Creational pattern]
tags: [design pattern, software design, creational pattern, abstract factory]
render_with_liquid: false
---

## 정의

추상 팩토리는 관련 객체들의 구상 클래스들을 지정하지 않고도 해당 객체들의 제품 패밀리들을 생성할 수 있도록 한다. 연관성이 있는 객체 군이 여러개 있을 경우 이들을 묶어 추상화하고, 어떤 구체적인 상황이 주어지면 팩토리 객체에서 집합으로 묶은 객체 군을 구현화 하는 생성 패턴이다. 클라이언트에서 특정 객체을 사용할때 팩토리 클래스만을 참조하여 특정 객체에 대한 구현부를 감추어 역할과 구현을 분리시킬 수 있다.

## 구성 요소

![image1](https://refactoring.guru/images/patterns/diagrams/abstract-factory/structure-2x.png)

+ Abstract Factory: 제품군을 생성하는 인터페이스를 정의.

+ Concrete Factory: 추상 팩토리를 구현하여 특정 제품군을 생성.

+ Abstract Product: 제품군 내 각 객체의 공통 인터페이스를 정의.

+ Concrete Product: 추상 제품을 구현한 실제 객체.

+ Client: 추상 팩토리를 사용해 객체를 생성하며, 구체적인 클래스에 의존하지 않음.

![image2](https://reactiveprogramming.io/_next/image?url=%2Fbooks%2Fpatterns%2Fimg%2Fpatterns-articles%2Fabstract-factory-diagram.png&w=3840&q=75)

## 예제

예제 코드: 

```java
// Abstract Products
interface Keyboard {
    void type();
}

interface Mouse {
    void click();
}

// Concrete Products
class LogiKeyboard implements Keyboard {
    public void type() {
        System.out.println("Typing with Logitech Keyboard");
    }
}

class AppleKeyboard implements Keyboard {
    public void type() {
        System.out.println("Typing with Apple Keyboard");
    }
}

class LogiMouse implements Mouse {
    public void click() {
        System.out.println("Clicking with Logitech Mouse");
    }
}

class AppleMouse implements Mouse {
    public void click() {
        System.out.println("Clicking with Apple Mouse");
    }
}

// Abstract Factory
interface ComputerFactory {
    Keyboard createKeyboard();
    Mouse createMouse();
}

// Concrete Factories
class LogiFactory implements ComputerFactory {
    public Keyboard createKeyboard() {
        return new LogiKeyboard();
    }

    public Mouse createMouse() {
        return new LogiMouse();
    }
}

class AppleFactory implements ComputerFactory {
    public Keyboard createKeyboard() {
        return new AppleKeyboard();
    }

    public Mouse createMouse() {
        return new AppleMouse();
    }
}

// Client Code
public class Client {
    public static void main(String[] args) {
        ComputerFactory factory = new LogiFactory(); // Change to AppleFactory for Apple products
        Keyboard keyboard = factory.createKeyboard();
        Mouse mouse = factory.createMouse();

        keyboard.type();
        mouse.click();
    }
}   
```

출력 결과: 

```text
Typing with Logitech Keyboard
Clicking with Logitech Mouse
```

## 특징

적용 시기

1. 추상 팩토리는 당신의 코드가 관련된 제품군의 다양한 패밀리들과 작동해야 하지만 해당 제품들의 구상 클래스들에 의존하고 싶지 않을 때 사용한다. 왜냐하면 이러한 클래스들은 당신에게 미리 알려지지 않았을 수 있으며, 그 때문에 향후 확장성​(extensibility)​을 허용하기를 원할 수 있기 때문이다.

2. 코드에 클래스가 있고, 이 클래스의 팩토리 메서드들의 집합의 기본 책임이 뚜렷하지 않을 때 추상 팩토리 구현을 고려하라.

장점

+ 팩토리에서 생성되는 제품들의 상호 호환을 보장할 수 있다.

+ 구상 제품들과 클라이언트 코드 사이의 단단한 결합을 피할 수 있다.

+ 단일 책임 원칙. 제품 생성 코드를 한 곳으로 추출하여 코드를 더 쉽게 유지보수할 수 있다.

+ 개방/폐쇄 원칙. 기존 클라이언트 코드를 훼손하지 않고 제품의 새로운 변형들을 생성할 수 있다.

단점

+ 패턴과 함께 새로운 인터페이스들과 클래스들이 많이 도입되기 때문에 코드가 필요 이상으로 복잡해질 수 있다.

## vs Factory Method

공통점

+ 객체 생성 과정을 추상화한 인터페이스를 제공

+ 객체 생성을 캡슐화함으로써 구체적인 타입을 감추고 느슨한 결합 구조를 표방

차이점

1. Factory Method는 구체적인 객체 생성과정을 하위 또는 구체적인 클래스로 옮기는 것이 목적. 반면, Abstract Factory는 관련 있는 여러 객체를 구체적인 클래스에 의존하지 않고 만들 수 있게 해주는 것이 목적

2. Factory Method는 한 Factory당 한 종류의 객체 생성 지원. 반면, Abstract Factory는 한 Factory에서 서로 연관된 여러 종류의 객체 생성을 지원. (제품군 생성 지원)

3. Factory Method는 메소드 레벨에서 포커스를 맞춤으로써, 클라이언트의 ConcreteProduct 인스턴스의 생성 및 구성에 대한 의존을 감소.
반면, Abstract Factory는 클래스 레벨에서 포커스를 맞춤.

주의해야 할 점은 **Abstract Factory 패턴이 Factory Method의 상위 호환이 아니라는 점**이다.

Factory Method는 **객체 생성 이후 해야 할 일**의 공통점을 정의하는 데 초점을 맞추는 반면, Abstract Factory는 생성해야 할 **객체 집합 군**의
공통점에 초점을 맞춘다.
