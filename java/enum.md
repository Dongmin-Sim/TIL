---
title: Enum
description: 
author: codongmin
date: 2025-02-01T21:34:00
categories:
  - 자바
tags: 
image:
  path: /assets/img/thumbnail/.png
  lqip: 
  alt: 이미지 설명
---

## Enum 이란

상수들의 집합을 정의할 때 사용하는 특수한 클래스 타입이다.

예외적인 상황에서도 단일 인스턴스를 유지하도록 설계되어있다.


enum은 다음과 같이 `enum` 선언을 통해서 선언이 가능하다.
```java
{ClassModifier} enum Identifier [Superinterfaces] EnumBody

--- 

public enum Color {

}
```


## Enum 특징 


```java

public abstract class Enum<E extends Enum<E>> 
	implements Constable, Comparable<E>, Serializable 
```


Enum은 클래스를 상속할 수 없지만 인터페이스는 구현이 가능하다.


Enun 은 abstract나 final 선언이 불가능하다.
- 자바의 `enum`은 **기본적으로 `final`이며, 다른 클래스가 이를 상속할 수 없습니다.**
- 즉, `enum`을 **명시적으로 `final`로 선언할 필요가 없고, 선언하면 오류가 발생합니다.**

#### Enum은 기본적으로 final이다. 

```java
public enum Animal {     DOG, CAT; }
```

`Animal` 자체는 `final`이므로 **다른 클래스가 `Animal`을 상속할 수 없습니다.** `DOG`와 `CAT`은 단순한 Enum 상수로만 존재합니다.

❌ **잘못된 상속 시도 (컴파일 오류 발생)**

```java
`class MyAnimal extends Animal {  // ERROR: enum은 상속할 수 없음! }`
```

#### enum 타입들의 부모 클래스는 `Enum<E>`

- 모든 Enum 타입은 **자동으로 `java.lang.Enum<E>` 클래스를 상속받음**.
- 그리고 `Enum<E>` 클래스는 코드로는 확인할 수 없지만, Object를 상속받는다.

```java
public static void main(String[] args) {  
    System.out.println(Color.BLUE);  
    System.out.println(Color.class.getSuperclass());  
    System.out.println(Color.class.getSuperclass().getSuperclass());  
}

---
> BLUE
> class java.lang.Enum
> class java.lang.Object
```

- `javap` 명령어로 바이트코드를 확인하면, `Enum<E>`가 `Object`를 상속하는 구조를 확인할 수 있음.


**중첩된 Enum은 자동으로 `static`**

- 명시적으로 `static`을 지정하는 것은 가능하지만, 중첩된 `enum`은 자동으로 `static`이므로 불필요함.
- 내부 클래스(Inner Class) 안에서는 `enum`을 선언할 수 없음. (내부 클래스는 정적 멤버를 가질 수 없기 때문)


1. **추가적인 인스턴스 생성 방지 메커니즘**
    
    - `Enum`의 `final clone()` 메서드가 **복제(clone) 방지**.
    - 리플렉션(Reflection)으로 인스턴스 생성 금지.
    - **직렬화(Serialization) 과정에서도 중복 인스턴스 생성이 방지됨.**


오버라이드는 
@Override  
public String toString() {  
    return super.toString();  
}

## Enum의 장점 



## 메모리 구조로 보는 Enum 


## Enum의 특징



## Enum 활용

값의 범위를 지정할 수 있다.

상태와 행위를 가질 수 있다. 자바 8부터 함수를 인자값으로 가질 수 있다.
기존의 if else, switch 를 줄일 수 있다. enum을 통해


그룹 관리?
1:N 관계 