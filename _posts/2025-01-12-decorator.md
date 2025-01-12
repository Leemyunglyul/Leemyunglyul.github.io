---
title: GoF Design Pattern - Decorator
date: 2025-01-12 13:17:00 +0900
categories: [Design pattern, Structural pattern]
tags: [design pattern, software design, structural pattern, decorator]
render_with_liquid: false
---

## 정의

데코레이터는 객체들을 새로운 행동들을 포함한 특수 래퍼 객체들 내에 넣어서 위 행동들을 해당 객체들에 연결시키는 구조적 디자인 패턴이다.

![image1](https://refactoring.guru/images/patterns/diagrams/decorator/problem3-2x.png)

위의 이미지는 자식 클래스들의 합성으로 클래스 수가 폭발한 모습이다.

![image2](https://refactoring.guru/images/patterns/diagrams/decorator/solution2-2x.png)

데코레이터를 사용하면 다음과 같이 해결할 수 있다.

## 구성 요소

![image3](https://refactoring.guru/images/patterns/diagrams/decorator/structure-2x.png)

+ Component: 기본 인터페이스 또는 추상 클래스. 데코레이터와 구체적인 컴포넌트가 이를 구현합니다.

+ ConcreteComponent: 기본 기능을 구현하는 클래스.

+ Decorator: Component를 감싸는 추상 클래스 또는 인터페이스로, 추가적인 기능을 정의합니다.

+ ConcreteDecorator: 데코레이터를 상속받아 실제로 새로운 기능을 구현하는 클래스.

## 예제

예제 코드:

```java
// Component Interface
public interface Pizza {
    String getDescription();
    double getCost();
}

// Concrete Component
public class BasicPizza implements Pizza {
    @Override
    public String getDescription() {
        return "Basic Dough";
    }

    @Override
    public double getCost() {
        return 5.0; // 기본 도우 가격
    }
}

// Abstract Decorator
public abstract class PizzaDecorator implements Pizza {
    protected Pizza pizza;

    public PizzaDecorator(Pizza pizza) {
        this.pizza = pizza;
    }

    @Override
    public String getDescription() {
        return pizza.getDescription();
    }

    @Override
    public double getCost() {
        return pizza.getCost();
    }
}

// Concrete Decorators
public class CheeseDecorator extends PizzaDecorator {
    public CheeseDecorator(Pizza pizza) {
        super(pizza);
    }

    @Override
    public String getDescription() {
        return super.getDescription() + ", Cheese";
    }

    @Override
    public double getCost() {
        return super.getCost() + 1.5; // 치즈 추가 비용
    }
}

public class PepperoniDecorator extends PizzaDecorator {
    public PepperoniDecorator(Pizza pizza) {
        super(pizza);
    }

    @Override
    public String getDescription() {
        return super.getDescription() + ", Pepperoni";
    }

    @Override
    public double getCost() {
        return super.getCost() + 2.0; // 페퍼로니 추가 비용
    }
}

public class OliveDecorator extends PizzaDecorator {
    public OliveDecorator(Pizza pizza) {
        super(pizza);
    }

    @Override
    public String getDescription() {
        return super.getDescription() + ", Olives";
    }

    @Override
    public double getCost() {
        return super.getCost() + 1.0; // 올리브 추가 비용
    }
}

// Client Code
public class DecoratorPatternPizzaExample {
    public static void main(String[] args) {
        // 기본 피자에 치즈와 페퍼로니를 추가한 경우
        Pizza cheesePepperoniPizza = new CheeseDecorator(new PepperoniDecorator(new BasicPizza()));
        System.out.println("Order: " + cheesePepperoniPizza.getDescription());
        System.out.println("Total Cost: $" + cheesePepperoniPizza.getCost());

        // 기본 피자에 올리브만 추가한 경우
        Pizza olivePizza = new OliveDecorator(new BasicPizza());
        System.out.println("Order: " + olivePizza.getDescription());
        System.out.println("Total Cost: $" + olivePizza.getCost());
    }
}
```

출력 결과:

```text
Order: Basic Dough, Pepperoni, Cheese  
Total Cost: $8.5  

Order: Basic Dough, Olives  
Total Cost: $6.0
```

## vs Builder

개인적으로 처음에 관련 이미지를 보고 빌더와 상당히 유사하다고 느꼈다. 여러 재료들이 있고 그걸 쓰냐 마냐로 판단하는데 그럼
무슨 차이가 있는 걸까? 하고 찾아보았다. 실제로 나와 같은 의문을 가진 사람들이 과거에도 있었고 `pizza problem`을 통해
이를 설명하고 있었다.

아래 코드는 위의 피자 문제를 빌더 패턴으로 구현한 예제다. 출력 결과는 동일하다.

```java
// Product Class (Pizza)
public class Pizza {
    private String description;
    private double cost;

    private Pizza(PizzaBuilder builder) {
        this.description = builder.description;
        this.cost = builder.cost;
    }

    public String getDescription() {
        return description;
    }

    public double getCost() {
        return cost;
    }

    // Static Builder Class
    public static class PizzaBuilder {
        private String description = "Basic Dough";
        private double cost = 5.0;

        public PizzaBuilder addCheese() {
            description += ", Cheese";
            cost += 1.5;
            return this;
        }

        public PizzaBuilder addPepperoni() {
            description += ", Pepperoni";
            cost += 2.0;
            return this;
        }

        public PizzaBuilder addOlives() {
            description += ", Olives";
            cost += 1.0;
            return this;
        }

        public Pizza build() { // 최종적으로 완성된 피자를 반환
            return new Pizza(this);
        }
    }
}

// Client Code
public class BuilderPatternPizzaExample {
    public static void main(String[] args) {
        // 치즈와 페퍼로니가 포함된 피자 생성
        Pizza cheesePepperoniPizza = new Pizza.PizzaBuilder()
                .addCheese()
                .addPepperoni()
                .build();

        System.out.println("Order: " + cheesePepperoniPizza.getDescription());
        System.out.println("Total Cost: $" + cheesePepperoniPizza.getCost());

        // 올리브만 포함된 피자 생성
        Pizza olivePizza = new Pizza.PizzaBuilder()
                .addOlives()
                .build();

        System.out.println("Order: " + olivePizza.getDescription());
        System.out.println("Total Cost: $" + olivePizza.getCost());
    }
}
```

눈에 띄는 점을 찾아보면 client code에서

Decorator: `Pizza cheesePepperoniPizza = new CheeseDecorator(new PepperoniDecorator(new BasicPizza()));`

Builder: `Pizza olivePizza = new Pizza.PizzaBuilder().addOlives().build();`

다음과 같이 표현된다.

차이점을 표로 나타내면 다음과 같다.

| **특징**       | **데코레이터 패턴**                      | **빌더 패턴**                          |
|----------------|-----------------------------------------|----------------------------------------|
| **목적**       | 객체에 동적으로 기능(토핑)을 추가         | 복잡한 객체(피자)를 단계적으로 생성      |
| **구현 방식**  | 객체를 감싸는(wrapper) 방식              | 메서드 체이닝(fluent interface)을 사용   |
| **유연성**     | 런타임에 동적으로 토핑을 추가하거나 제거 가능 | 객체 생성 시점에만 토핑을 설정 가능      |
| **사용 시점**  | 이미 생성된 객체의 동작을 확장            | 객체를 생성하기 전에 구성 단계에서 설정  |

결론만 말하자면 빌더 패턴은 이미 설계 시점에서 피자인 것을 인지한 것이고 데코레이터 패턴은 기존에 이미 다른 시스템이 있었는데
기능을 확장하는 과정에서 피자인 것을 깨달아서 피자처럼 만드는 것이다. 기존의 코드는 수정하지 않으면서 단지 코드를 추가만 해서
피자로 만드는 것을 가능케 하는 거라 보면 되겠다. ~~맞겠지?~~

## 특징

적용 시기

1. 데코레이터 패턴은 이 객체들을 사용하는 코드를 훼손하지 않으면서 런타임에 추가 행동들을 객체들에 할당할 수 있어야 할 때 사용하세요.

2. 이 패턴은 상속을 사용하여 객체의 행동을 확장하는 것이 어색하거나 불가능할 때 사용하세요.

장점

+ 새 자식 클래스를 만들지 않고도 객체의 행동을 확장할 수 있습니다.

+ 런타임에 객체들에서부터 책임들을 추가하거나 제거할 수 있습니다.

+ 객체를 여러 데코레이터로 래핑하여 여러 행동들을 합성할 수 있습니다.

+ 단일 책임 원칙. 다양한 행동들의 여러 변형들을 구현하는 모놀리식 클래스를 여러 개의 작은 클래스들로 나눌 수 있습니다.

단점

+ 래퍼들의 스택에서 특정 래퍼를 제거하기가 어렵습니다.

+ 데코레이터의 행동이 데코레이터 스택 내의 순서에 의존하지 않는 방식으로 데코레이터를 구현하기가 어렵습니다.(ex: A -> B != B-> A)

+ 계층들의 초기 설정 코드가 보기 흉할 수 있습니다.