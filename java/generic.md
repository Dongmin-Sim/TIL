---
title: 제네릭
description: 타입안정성과 재사용을 지원하는 제네릭
author: codongmin
date: 2025-02-14T21:34:00
categories:
  - 회고
  - 연말
tags: ["generic"]
image:
  path: /assets/img/thumbnail/.png
  lqip:
  alt: 제네릭
---



## 들어가며

업 캐스팅된 타입을 다시 다운캐스팅할 때 발생할 수 있는 실수나, 이를 점검하기 위한 코드들이(`instanceof` 생기는 단점을 보완하기 위해 Java5부터 새롭게 추가된 개념이 바로 제네릭.



## 제네릭이란?

제네릭은 클래스나 인터페이스, 메소드를 정의할때 **타입을 파라미터화**하여 컴파일 시점에 구체적인 타입을 결정하도록한 자바 문법이다.



본 글에서는 예시를 통해서 제네릭의 필요성과 타입을 파라미터화 한다는 것의 장점을 설명하려고 한다. 

화물을 옮길 수 있는 트럭을 예시로 들어본다. 현재 이 트럭은 옥수수곡물을 담기 위한 트럭으로 설계되었다. 이 트럭에는 100만큼의 무게 적재량이 존재하고, 각각의 옥수수 곡물 포대를 담을 공간이 존재한다. 

```java
public class Truck {

    private final int MAX_LOAD_WEIGHT = 100;
    private final List stowage = new ArrayList();
    private int currentWeight = 0;

    public void load(Corn payload) throws IllegalStateException {
        if (currentWeight + payload.getWeight() >= MAX_LOAD_WEIGHT) {
            throw new IllegalStateException("더 이상 화물을 실을 수 없습니다.");
        }
        stowage.add(payload); // 화물 싣기
        addWeight(payload.getWeight()); // 무게 추가
    }

    public Corn unload() {
        if (currentWeight <= 0) {
            throw new IllegalStateException("더 이상 내릴 화물이 없습니다.");
        }
        Corn payload = (Corn) stowage.remove(0); // 화물 내리기
        loseWeight(payload.getWeight()); // 무게 빼기
      
        return payload;
    }
  
  	//...
}
```



곡물을 생산하는 곡물 공장도 지어본다. 

```java
public class GrainFactory {

    public static Corn createCorn() {
        Random random = new Random();
        int randomWeight = random.nextInt(30) + 1;

        return new Corn(randomWeight);
    }
  
		public static void loadPayload(Truck truck) {
        while (!truck.isFull()) {
            try {
                Corn corn = GrainFactory.createCorn();
                truck.load(corn);
            } catch (IllegalStateException e) {
                System.out.println(e.getMessage());
                break;
            }
        }
    }
}
```



만일 곡물이 하나 더 늘어나게 된다면 어떻게 될까? 곡물 공장에서는 올해부터 옥수수에 추가로 밀가루를 추가하기로 했다.





제네릭은 타입 형 변환에서 발생할 수 있는 문제점을 “사전”에 없애기 위해 만들어졌음. 여기서 의미하는 “사전”은 런타임이 아닌 컴파일 시점에 점검할 수 있도록 한 것을 말함.

- 타입 캐스팅을 반복적으로 사용하면 코드가 복잡해지고 실수가 발생할 가능성이 많음
- 잘못된 타입 캐스팅으로 인한 런타임 오류 가능성이 존재 (컴파일 시에는 문제가 없지만 런타임시에 `ClassCastException` 이 발생)
- 제네릭이 없던 시절, 컬렉션에 다양한 타입의 객체를 저장할 수 있었음. 다른 타입이 들어와도 컴파일러가 이를 감지할 수 없었다.

제네릭은 **타입 안정성**을 보장하고, 코드 **재사용성을 높이는데** 사용된다.

제네릭은 런타임 오류를 컴파일 오류로 전환했다는 것에서 의미가 있음.

```java
public class CastingDTO implements Serializable {

    private Object object;

    public Object getObject() {
        return object;
    }

    public void setObject(Object object) {
        this.object = object;
    }
}
```

다음과 같은 DTO 객체가 있다고 했을때 `CastingDTO` 의 인스턴스변수는 Object 로 어떤 타입이든 선언할 수 있음. 이 클래스를 제네릭으로 선언하면 다음과 같음.

```java

public class CastingGenericDTO<T> implements Serializable {

    private T object;

    public T getObject() {
        return object;
    }

    public void setObject(T object) {
        this.object = object;
    }
}

```

클래스 선언문에 열리고 닫힌 <> 꺽쇠가 추가되었고, 기존의 선언되어있던 타입이 전부 T라는 대문자로 바뀌었다. 여기서 T 대문자는 클래스 안에서 하나의 타입처럼 사용할 수 있다. 다만 아직 어떤 타입인지는 지정되지 않은 상태를 말한다. 가상의 타입이라고 생각할 수 있다. 때문에 어떠한 문자도 들어갈 수 있다.

제네릭을 사용할 경우 별도의 불필요한 형변환 로직을 제거할 수 있다. 이는 재사용과도 관련이 있다.

## 제네릭의 동작 원리

제네릭은 타입
**타입 안정성**을 보장하고, 코드 **재사용성을 높이는데** 사용된다. 제네릭은 **컴파일 시점 타입을 체크**하며, 런타임에는 타입 정보를 제거하는 **타입 이레이저(Type Erasure) 매커니즘**을 통해 동작함.

- 타입 파라미터
  - 제네릭은 클래스, 인터페이스, 메서드에서 사용할 타입을 파라미터로 지정함.
  - 이때 제네릭 타입으로 지정할 수 있는 문자열에는 제한이 없음.
  - 하지만 보통 다음과 같은 컨벤션을 따른다.
    - `E` : 요소(Element, 자바 컬렉션에서 주로 사용됨)
    - `K` : 키
    - `K` : 숫자
    - `K` : 타입
    - `K` : 값
    - `S` , `U` , `V` : 두 번째, 세 번째, 네 번째 선언된 타입

```java
public class Box<T> {
    private T item;

    public void setItem(T item) {
        this.item = item;
    }

    public T getItem() {
        return item;
    }
}

public class Main {
    public static void main(String[] args) {
        Box<String> box = new Box<>();

        box.setItem("Hello, World!");
        // box.setItem(123); // 컴파일 에러 발생

        String value = box.getItem();
        System.out.println(value);
    }
}
```

- 컴파일 시 타입 체크

  - 제네릭은 컴파일 시점에 타입 체크를 수행해 잘못된 타입 사용을 1차적으로 방지함.
  - 제네릭을 적용한

  

  ### 타입 이레이저

  - 자바의 제네릭은 컴파일 시점에는 사용되지만, 컴파일 이후에 제네릭 정보를 제거하고 원시타입으로 변환됨. 이러한 과정을 **타입 이레이저(Type Erasure)** 라고 부름.
  - 이는 자바의 하위 호환성을 유지하기 위해 설계된 메커니즘으로, 제네릭이 도입되기 이전(Java 5 이전)의 코드와 호환되도록 하기 위함 **(왜???상세 필요 )**

따라서 위 제네릭을 사용했던 소스코드가 컴파일 후에는 다음과 같은 원시타입으로 변경됨.

```java
public class Box {
    private Object item;

    public void setItem(Object item) {
        this.item = item;
    }

    public Object getItem() {
        return item;
    }
}
```

그리고 이를 사용하는 코드에서는 컴파일러가 적절한 곳에 강제 캐스팅 코드를 입력해준다.

```java
public class Main {
    public static void main(String[] args) {
        Box box = new Box();
        box.setItem("Hello, World!");
        String value = (String) box.getItem(); // 강제 캐스팅 추가
        System.out.println(value);
    }
}
```

이러한 제네릭의 동작원리 덕분에 타입변환 시 런타임에서 발생할 수 있는 예외를 미리 잡아낼 수 있다.





## 바운디드 타입, 제네릭 타입 한정자(extends)







## 제네릭, 와일드 카드

와일드 카드는 제네릭 클래스를 만들때 사용하는 것이 아니라, 이미 만들어진 **제네릭 클래스를 사용할**때 타입을 지정할때 이용되는 것이다.



**와일드카드의 장점은 하나의 참조변수에 다양한 매개변수화된 타입을 가지는 지네릭 객체를 담는 다형성 구현입니다.** 



### 필요성 - 오버로딩 불가

앞서 제네릭에서는 컴파일 시점에 제네릭 정보를 제거하고 원시타입으로 변환하는 타입 이레이저로 인해서 컴파일 이후에는 특정 타입정보가 제거되기 때문에 오버로딩이 불가능하다는 단점이 존재.

```java
static Juice makeJuice(FruitBox<Fruit> box) { ... }
static Juice makeJuice(FruitBox<Apple> box) { ... }
```

타입 이레이저 이후

```java
static Juice makeJuice(FruitBox box) { ... }
static Juice makeJuice(FruitBox box) { ... }
```

이러한 문제를 해결하기 위해 와일드카드(?)가 등장. 와일드카드는 특정 타입 대신 "알 수 없는 타입"을 표현하며, 이를 통해 더 유연하게 메서드를 정의할 수 있게 됨.



메서드의 매개변수로 사용됨. 

(와일드 카드가 있다면?)



다만, 와일드카드는 제네릭 클래스나 인터페이스를 선언할 때 타입 매개변수로 직접 사용할 수는 없음. 

```java
public class Box<?> {
  
}
```





## 주의사항

제네릭 타입으로 배열을 생성할 수 없음.

- 배열은 new 연산자를 통해서 타입이 무엇인지 알아야함

Static 멤버에는 제네릭을 사용할 수 없음. 

- Static 멤버는 로드되는 시점에 메서드 영역에 저장되는데







---

참고자료 

- https://escapefromcoding.tistory.com/854

