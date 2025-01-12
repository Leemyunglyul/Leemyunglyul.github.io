---
title: GoF Design Pattern - Composite
date: 2025-01-12 14:16:00 +0900
categories: [Design pattern, Structural pattern]
tags: [design pattern, software design, structural pattern, composite]
render_with_liquid: false
---

## 정의

복합체 패턴은 객체들을 트리 구조들로 구성한 후, 이러한 구조들과 개별 객체들처럼 작업할 수 있도록 하는 구조 패턴이다.
즉, 복합체는 앱의 핵심 모델이 **트리**로 표현될 수 있을 때 사용하는 게 좋다.

## 구성요소

![image](https://refactoring.guru/images/patterns/diagrams/composite/structure-ko-2x.png)

+ Component: 공통 인터페이스 또는 추상 클래스. 단일 객체와 복합 객체가 공통으로 제공해야 할 메서드를 정의.

+ Leaf: 트리의 말단 노드로, 더 이상 하위 요소를 가지지 않는 단일 객체.

+ Composite: 다른 Component(Leaf 또는 Composite)를 포함할 수 있는 복합 객체.

+ Client: Component 인터페이스만 사용하며, 개별 객체(Leaf)와 복합 객체(Composite)를 구분하지 않는다.

## 예제

예제 코드:

```java
// Component: 공통 인터페이스
public interface Component {
    void showDetails();
}

// Leaf: 개별 구성 요소
public class Leaf implements Component {
    private String name;

    public Leaf(String name) {
        this.name = name;
    }

    @Override
    public void showDetails() {
        System.out.println("Leaf: " + name);
    }
}

// Composite: 복합 구성 요소
import java.util.ArrayList;
import java.util.List;

public class Composite implements Component {
    private String name;
    private List<Component> children = new ArrayList<>();

    public Composite(String name) {
        this.name = name;
    }

    public void add(Component component) {
        children.add(component);
    }

    public void remove(Component component) {
        children.remove(component);
    }

    @Override
    public void showDetails() {
        System.out.println("Composite: " + name);
        for (Component child : children) {
            child.showDetails(); // 재귀 호출
        }
    }
}

public class CompositePatternDemo {
    public static void main(String[] args) {
        // Leaf 노드 생성
        Component leaf1 = new Leaf("Leaf 1");
        Component leaf2 = new Leaf("Leaf 2");
        Component leaf3 = new Leaf("Leaf 3");

        // Composite 노드 생성
        Composite composite1 = new Composite("Composite 1");
        Composite composite2 = new Composite("Composite 2");

        // 트리 구조 구성
        composite1.add(leaf1);
        composite1.add(leaf2);

        composite2.add(leaf3);
        composite2.add(composite1);

        // 전체 트리 출력
        composite2.showDetails();
    }
}
```

출력 결과:

```text
Composite: Composite 2
Leaf: Leaf 3
Composite: Composite 1
Leaf: Leaf 1
Leaf: Leaf 2
```

## 특징

적용 시기

1. 복합체 패턴은 나무와 같은 객체 구조를 구현해야 할 때 사용하세요.

2. 이 패턴은 클라이언트 코드가 단순 요소들과 복합 요소들을 모두 균일하게 처리하도록 하고 싶을 때 사용하세요.

장점

+ 다형성과 재귀를 당신에 유리하게 사용해 복잡한 트리 구조들과 더 편리하게 작업할 수 있습니다.

+ 개방/폐쇄 원칙. 객체 트리와 작동하는 기존 코드를 훼손하지 않고 앱에 새로운 요소 유형들을 도입할 수 있습니다.

단점

+ 기능이 너무 다른 클래스들에는 공통 인터페이스를 제공하기 어려울 수 있으며, 어떤 경우에는 컴포넌트 인터페이스를 과도하게 일반화해야 하여 이해하기 어렵게 만들 수 있습니다.