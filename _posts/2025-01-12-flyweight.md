---
title: GoF Design Pattern - Flyweight
date: 2025-01-12 18:16:00 +0900
categories: [Design pattern, Structural pattern]
tags: [design pattern, software design, structural pattern, flyweight]
render_with_liquid: false
---

## 정의

플라이웨이트는 각 객체에 모든 데이터를 유지하는 대신 여러 객체들 간에 상태의 공통 부분들을 공유하여 사용할 수 있는 RAM에 더 많은 객체들을 포함할 수 있도록 하는 구조 디자인 패턴이다. 즉, 재사용 가능한 객체 인스턴스를 공유시켜 메모리 사용량을 최소화하는 구조 패턴이다.

## 핵심 개념

이 플라이웨이트 패턴에서 주의깊게 봐야 할 점은 **Intrinsic와 Extrinsic의 상태를 구분하는 것**이다.

Intrinsic는 '고유한, 본질적인' 의미가 있다. 이 상태에서는 인스턴스가 어떠한 상황에서도 변하지 않는 정보를 말한다. 값이 고정되어 있기에
**공유해도 문제 없다**.

**공유 객체** 라는 개념도 있다. 동일한 내부 상태를 가진 객체는 재사용되며, 새로운 인스턴스를 생성하지 않는다.

Extrinsic은 '외적인, 비본질적인' 의미가 있다. 인스턴스를 두는 장소나 상황에 따라서 변화하는 정보를 말한다. 따라서 **공유할 수 없다**.


## 구성 요소

![image](https://refactoring.guru/images/patterns/diagrams/flyweight/structure-2x.png)

+ Flyweight: 고유한 상태를 저장하며, 여러 콘텍스트에서 재사용된다.

+ Context: 외부 상태를 저장하며, 특정 플라이웨이트와 결합하여 동작을 수행한다.

+ FlyweightFactory: 플라이웨이트 객체들을 관리하며, 중복 생성을 방지한다.

+ Client: 팩토리를 통해 플라이웨이트 객체를 요청하고, 필요한 외부 상태를 전달하여 작업을 수행한다.

## 예제

### Flyweight

```java
// Flyweight Interface
public interface TreeType {
    void draw(int x, int y); // 외부 상태(x, y 좌표)를 전달받아 동작 수행
}

// Concrete Flyweight
public class ConcreteTreeType implements TreeType {
    private final String color;   // 고유한 상태 (Intrinsic State)
    private final String texture; // 고유한 상태 (Intrinsic State)

    public ConcreteTreeType(String color, String texture) {
        this.color = color;
        this.texture = texture;
    }

    @Override
    public void draw(int x, int y) {
        System.out.println("Drawing tree of color " + color + " and texture " + texture + " at (" + x + ", " + y + ")");
    }
}
```

### Context 

```java
// Context: 외부 상태를 저장하는 클래스
public class Tree {
    private final int x; // 외부 상태 (Extrinsic State)
    private final int y; // 외부 상태 (Extrinsic State)
    private final TreeType type; // Flyweight 객체 참조

    public Tree(int x, int y, TreeType type) {
        this.x = x;
        this.y = y;
        this.type = type;
    }

    public void draw() {
        type.draw(x, y); // Flyweight 객체와 외부 상태 결합
    }
}
```

### FlyweightFactory 

```java
import java.util.HashMap;
import java.util.Map;

public class TreeFactory {
    private static final Map<String, TreeType> treeTypes = new HashMap<>();

    public static TreeType getTreeType(String color, String texture) {
        String key = color + "-" + texture;
        if (!treeTypes.containsKey(key)) {
            treeTypes.put(key, new ConcreteTreeType(color, texture));
            System.out.println("Creating new TreeType: " + key);
        }
        return treeTypes.get(key);
    }
}
```
### Client

```java
import java.util.ArrayList;
import java.util.List;

public class FlyweightPatternExample {
    public static void main(String[] args) {
        List<Tree> forest = new ArrayList<>();

        // 동일한 색상과 텍스처를 가진 나무는 공유됨
        TreeType oakType = TreeFactory.getTreeType("Green", "OakTexture");
        forest.add(new Tree(10, 20, oakType));
        forest.add(new Tree(15, 25, oakType));

        TreeType pineType = TreeFactory.getTreeType("DarkGreen", "PineTexture");
        forest.add(new Tree(30, 40, pineType));
        forest.add(new Tree(35, 45, pineType));

        // 새로운 타입의 나무 생성
        TreeType cherryBlossomType = TreeFactory.getTreeType("Pink", "CherryBlossomTexture");
        forest.add(new Tree(50, 60, cherryBlossomType));

        // 모든 나무 그리기
        for (Tree tree : forest) {
            tree.draw();
        }
    }
}
```

### 출력 결과

```text
Creating new TreeType: Green-OakTexture
Creating new TreeType: DarkGreen-PineTexture
Creating new TreeType: Pink-CherryBlossomTexture
Drawing tree of color Green and texture OakTexture at (10, 20)
Drawing tree of color Green and texture OakTexture at (15, 25)
Drawing tree of color DarkGreen and texture PineTexture at (30, 40)
Drawing tree of color DarkGreen and texture PineTexture at (35, 45)
Drawing tree of color Pink and texture CherryBlossomTexture at (50, 60)
```

## 특징

적용 시기: 플라이웨이트 패턴은 당신의 프로그램이 많은 수의 객체들을 지원해야 해서 사용할 수 있는 RAM을 거의 다 사용했을 때만 사용하세요.

장점: 당신의 프로그램에 유사한 객체들이 많다고 가정하면 많은 RAM을 절약할 수 있습니다.

단점

+ 누군가가 플라이웨이트 메서드를 호출할 때마다 콘텍스트 데이터의 일부를 다시 계산해야 한다면 당신은 CPU 주기 대신 RAM을 절약하고 있는 것일지도 모릅니다.

+ 코드가 복잡해지므로 새로운 팀원들은 왜 개체​(entity)​의 상태가 그런 식으로 분리되었는지 항상 궁금해할 것입니다.