---
title: GoF Design Pattern - Singleton
date: 2025-01-11 04:00:00
categories: [Design pattern, Creational pattern]
tags: [design pattern, software design, creational pattern, singleton]
render_with_liquid: false
---

## 정의

싱글톤은 클래스에 인스턴스가 하나만 있도록 하면서 이 인스턴스에 대한 전역 접근​(액세스) 지점을 제공하는 생성 디자인 패턴이다.
가장 간단한 패턴이며, **단 하나의 유일한 객체**를 만들기 위한 코드 패턴이다.

메모리 절약을 위해 똑같은 인스턴스를 새로 만들지 않고 기존 인스턴스를 가져와 활용하는 기법이다. 전역 변수를 클래스에 대입한 것이라
보면 편하다. 싱글톤 패턴이 필요한 경우는 그 객체가 **리소스를 많이 차지하는 역할**을 하는 무거운 클래스일 때 적합하다. 그 예로,
데이터베이스 연결 모듈이 있다. 데이터베이스 접속 작업은 무겁고, 한 번만 객체를 생성하고 돌려쓰면 된다.

## 구성 요소

![image](https://refactoring.guru/images/patterns/diagrams/singleton/structure-ko-2x.png)

## 예제

### Lazy Holder(Bill Pugh 방식)

내부 정적 클래스를 이용하여 Thread-Safe하면서도 Lazy Initialization을 구현한다.

```java
public class Singleton {
    private Singleton() {}

    private static class Holder {
        private static final Singleton INSTANCE = new Singleton(); // 내부 클래스에서 인스턴스 생성
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE; // 내부 클래스의 인스턴스를 반환
    }
}
```

### Enum 방식

Java Enum은 싱글톤을 구현하는 가장 간단하고 안전한 방법이다.

```java
public enum Singleton {
    INSTANCE; // 단일 인스턴스

    public void doSomething() {
        System.out.println("Doing something...");
    }
}
```

## 특징

적용 시기

+ 싱글턴 패턴은 당신 프로그램의 클래스에 모든 클라이언트가 사용할 수 있는 단일 인스턴스만 있어야 할 때 사용한다. 예를 들자면 프로그램의 다른 부분들에서 공유되는 단일 데이터베이스 객체처럼 말이다.

+ 싱글턴 패턴은 전역 변수들을 더 엄격하게 제어해야 할 때 사용한다.

장점

+ 클래스가 하나의 인스턴스만 갖는다는 것을 확신할 수 있다.
 
+ 이 인스턴스에 대한 전역 접근 지점을 얻는다.

+ 싱글턴 객체는 처음 요청될 때만 초기화된다.

단점

+ 단일 책임 원칙을 위반합니다. 이 패턴은 한 번에 두 가지의 문제를 동시에 해결합니다.
 
+ 또 싱글턴 패턴은 잘못된 디자인​(예를 들어 프로그램의 컴포넌트들이 서로에 대해 너무 많이 알고 있는 경우)​을 가릴 수 있습니다.
 
+ 그리고 이 패턴은 다중 스레드 환경에서 여러 스레드가 싱글턴 객체를 여러 번 생성하지 않도록 특별한 처리가 필요합니다.

+ 싱글턴의 클라이언트 코드를 유닛 테스트하기 어려울 수 있습니다. 그 이유는 많은 테스트 프레임워크들이 모의 객체들을 생성할 때 상속에 의존하기 때문입니다. 싱글턴 클래스의 생성자는 비공개이고 대부분 언어에서 정적 메서드를 오버라이딩하는 것이 불가능하므로 싱글턴의 한계를 극복할 수 있는 창의적인 방법을 생각해야 합니다. 아니면 그냥 테스트를 작성하지 말거나 싱글턴 패턴을 사용하지 않으면 됩니다.