---
title: GoF Design Pattern - Template Method
date: 2025-01-13 23:11:00 +0900
categories: [Design pattern, Behavioral pattern]
tags: [design pattern, software design, behavioral pattern, template method]
render_with_liquid: false
---

## 정의

템플릿 메서드는 부모 클래스에서 알고리즘의 골격을 정의하지만, 해당 알고리즘의 구조를 변경하지 않고 자식 클래스들이 알고리즘의 특정 단계들을 오버라이드​(재정의)​할 수 있도록 하는 행동 디자인 패턴이다. 변하지 않는 부분은 상위 클래스에 작성하고, 변하는 부분만 하위 클래스에서 오버라이드한다. 

## 구성 요소

![image](https://refactoring.guru/images/patterns/diagrams/template-method/structure-2x.png)

+ AbstractClass: 템플릿 메서드를 정의하며, 알고리즘의 골격을 제공합니다. 추상 메서드를 포함합니다.

+ ConcreteClass: AbstractClass를 상속받아 추상 메서드를 구현하며, 구체적인 동작을 정의합니다.

아주 단순환 구조를 취하고 있다. 상위 클래스에서 전체적인 틀을 정의하고, 알고리즘의 일부 단게만 하위 클래스에서 구현하도록 한다.
Hook 메서드(Optional)를 통해 하위 클래스가 반드시 구현하지 않아도 되는 선택적 메서드를 제공할 수 있다.

## 예제

예제 코드:

```java
// 음료 준비 과정을 정의한 추상 클래스
abstract class Beverage {
    // 템플릿 메서드: 음료 준비 과정의 골격
    public final void prepareBeverage() {
        boilWater();
        brew();
        pourInCup();
        if (customerWantsCondiments()) { // Hook 메서드
            addCondiments();
        }
    }

    // 공통된 단계 (구현된 메서드)
    private void boilWater() {
        System.out.println("물을 끓이는 중...");
    }

    private void pourInCup() {
        System.out.println("컵에 따르는 중...");
    }

    // 세부 단계 (추상 메서드)
    protected abstract void brew(); // 재료 준비
    protected abstract void addCondiments(); // 첨가물 추가

    // Hook 메서드: 기본값은 true이며, 필요 시 하위 클래스에서 오버라이드 가능
    protected boolean customerWantsCondiments() {
        return true;
    }
}

class Coffee extends Beverage {
    @Override
    protected void brew() {
        System.out.println("커피를 필터로 내리는 중...");
    }

    @Override
    protected void addCondiments() {
        System.out.println("설탕과 우유를 추가하는 중...");
    }

    @Override
    protected boolean customerWantsCondiments() {
        return false; // 첨가물을 원하지 않는 경우
    }
}

class Tea extends Beverage {
    @Override
    protected void brew() {
        System.out.println("차를 우려내는 중...");
    }

    @Override
    protected void addCondiments() {
        System.out.println("레몬을 추가하는 중...");
    }
}

public class TemplateMethodExample {
    public static void main(String[] args) {
        System.out.println("커피 준비:");
        Beverage coffee = new Coffee();
        coffee.prepareBeverage();

        System.out.println("\n차 준비:");
        Beverage tea = new Tea();
        tea.prepareBeverage();
    }
}
```

출력 결과:

```text
커피 준비:
물을 끓이는 중...
커피를 필터로 내리는 중...
컵에 따르는 중...

차 준비:
물을 끓이는 중...
차를 우려내는 중...
컵에 따르는 중...
레몬을 추가하는 중...
```

## 특징

적용 시기

1. 템플릿 메서드 패턴은 클라이언트들이 알고리즘의 특정 단계들만 확장할 수 있도록 하고 싶을 때, 그러나 전체 알고리즘이나 알고리즘 구조는 확장하지 못하도록 하려고 할 때 사용하세요.

2. 이 패턴은 약간의 차이가 있지만 거의 같은 알고리즘들을 포함하는 여러 클래스가 있는 경우에 사용하세요. 결과적으로 알고리즘이 변경되면 모든 클래스를 수정해야 할 수도 있습니다.

장점

+ 클라이언트들이 대규모 알고리즘의 특정 부분만 오버라이드하도록 하여 그들이 알고리즘의 다른 부분에 발생하는 변경에 영향을 덜 받도록 할 수 있습니다.

+ 중복 코드를 부모 클래스로 가져올 수 있습니다.

단점

+ 일부 클라이언트들은 알고리즘의 제공된 골격에 의해 제한될 수 있습니다.

+ 당신은 자식 클래스를 통해 디폴트 단계 구현을 억제하여 리스코프 치환 원칙을 위반할 수 있습니다.

+ 템플릿 메서드들은 단계들이 더 많을수록 유지가 더 어려운 경향이 있습니다.