---
title: 예외
description: 자바의 예외
author: codongmin
date: 2025-02-14T21:34:00
categories:
  - java
tags: ["exception", "checked-exception", "runtime-exception"]
image:
  path: /assets/img/thumbnail/_.png
  lqip:
  alt:
---

## 들어가며

본 글에서는 예외란 무엇인지, 자바에서 제공하는 예외처리 방법들에는 무엇이 있는지 살펴본다.

적절한 예외 처리는 예기치 못한 프로그램의 종료를 막을 수 있기 때문에 중요하다. 


## 예외란?

예외란 프로그램의 **정상적 흐름을 벗어난 예외적**인 사건, 상황을 의미한다.

정상적 흐름에서 발생할 수 있는 예외적인 사건은 매우 여러가지가 존재한다. 프로그램 상의 오류일 수도 있고, 외부 시스템에 의한 것일 수도 있고 개발자가 의도한 것일수도 있다.

- 파일이 존재하지 않거나 읽을 수 없을 때
- 네트워크 연결이 끊어졌을 때
-  SQL 구문 오류가 있을 때
- 데이터베이스 연결 실패 시
- 수학적 연산 중에 0으로 나누려고 할 때
- 재귀 호출이 너무 깊어져서 스택 메모리가 초과되는 경우
- 메모리 할당이 부족할 때 


## 예외를 마주하게 된다면

예외가 발생하면 할 수 있는 일은 다음과 같다.

### 1. 예외 상황이 현재 위치에서 복구가 가능한 상황이라면

예외란 정상적인 흐름에서 발생된 예외 이벤트라 했다. 때문에 다시 **정상흐름으로 복구**할 수 있다면 그렇게 해야한다. 

예를 들어:
- **재시도**: 외부 시스템과의 연결이 일시적으로 끊어진 경우, 몇 초 후 다시 시도하는 방식으로 복구할 수 있다.
- **대안 방법을 찾기**: 네트워크 요청이 실패했을 때, 데이터베이스에서 해당 데이터를 찾아오는 방법으로 대체할 수 있다. 이를 통해 예외 상황을 처리하고 정상 흐름으로 복구할 수 있다.

이처럼 복구가 가능한 예외는 최소한의 비용으로 빠르게 해결하는 것이 적절하다.

### 2. 현재 위치에서 복구가 불가능하다면

현재 위치에서 이 예외를 복구하는 것이 가능하지 않거나 적절하지 않다면 이 예외에 대해서 최소한의 처리를 진행해주고 다시 **예외를 외부로 넘김으로써 예외 처리가 필요함**을 알려야한다.

최소한의 처리는 현재 예외에 대한 정보를 남기기 위해 **로깅**을 하거나, 별도의 다른 예외로 변환하는 작업들을 생각해볼 수 있다. 다른 예외로 변환하는 이유는 예외 발생 원인과 관련된 상세 정보를 제공하여 문제 해결에 도움을 주기 위해서다.

예외를 다른 예외로 변환할 때는 **더 높은 레벨의 예외**나 **의미 있는 예외**를 사용하는 것이 중요하다. 예외가 너무 일반적이거나, 정보가 부족하면 문제 해결이 어려울 수 있기 때문이다. 예를 들어, `IOException`을 `RuntimeException`으로 바꿀 때, **더 구체적인 메시지**와 **원본 예외**를 전달하는 것이 중요하다. 이를 통해 예외를 발생시킨 원인을 명확하게 파악하고, 예외 처리가 필요한 곳에서 적절한 대응을 할 수 있도록 돕는다.

또한, 예외를 변환하는 과정에서 예외 체인을 유지하는 것이 좋다. 즉, 원본 예외를 **`cause`**로 포함시켜서, 나중에 예외를 추적할 때 원인 파악이 용이하도록 한다.

예를 들어:

```java
try {
    // 예외가 발생할 수 있는 코드
    FileReader file = new FileReader("data.txt");
} catch (IOException e) {
	// 다시 예외를 발생시킨다.
    throw new FileReadException("파일을 읽는 중 오류 발생", e);
}
```

위와 같이 `IOException`을 `FileReadException`으로 변환할 때, `IOException`을 `FileReadException`의 `cause`로 전달함으로써 예외를 추적할 수 있다.

이러한 방식으로 예외 처리를 하면, 시스템이 예상치 못한 문제를 만났을 때 보다 나은 대처가 가능하고, 나중에 문제를 해결할 때 유용한 정보를 제공할 수 있다.

## 예외 처리 매커니즘 

자바에서는 이러한 비정상적인 예외 이벤트를 다룰 수 있는 예외 처리 매커니즘들을 제공한다. 

### try-catch-finally

첫번째는 try-catch-finally 구문이다. 

```java
try {
    // 예외가 발생할 수 있는 코드
} catch (SpecificException e) {
    // 예외 처리 코드
} finally {
    // 예외 발생 여부와 관계없이 실행되는 코드
}
```

- **try**: 예외가 발생할 수 있는 코드를 작성하는 블록.
- **catch**: 발생한 예외를 처리하는 블록.
- **finally**: 예외가 발생하든 발생하지 않든 반드시 실행되는 블록. 자원 해제 등을 처리할 때 유용.

`try` 블록은 우리가 기대하는 정상 흐름이 담긴 코드이자 예외가 발생할 가능성이 존재하는 코드가 위치하는 블록이다. 이때 `try` 블록에서 예외가 발생하면 해당 예외를 처리할 수 있는 `catch` 블록으로 넘어오게 된다. 

`finally`는 예외가 발생하든 발생하지 않든 **무조건 실행**되는 블록이기 때문에, 예외 처리의 유무와 관계없이 리소스를 정리하고 닫는 작업, **리소스를 반드시 해제해야 하는 경우**에 사용된다. 예를 들어, 데이터베이스 연결, 파일 I/O 작업, 네트워크 연결 등에서 리소스를 열고 닫는 작업을 수행하는 도중 예외가 발생하게 된다면 자원이 해제되지 않아 예상치 못한 문제가 발생할 수 있기 때문이다. 

```java
Connection conn = null;
try {
    conn = DriverManager.getConnection("//");
    // DB 작업
} catch (SQLException e) {
    e.printStackTrace();
} finally {
    // 예외가 발생하든 발생하지 않든 연결을 닫는 작업 수행
    if (conn != null) {
        try {
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}

```

예를 들어 자바의 가비지 컬렉터(GC)는 객체를 자동으로 관리하지만, 파일 핸들, 데이터베이스 커넥션 같은 리소스는 GC의 관리 대상이 아니다. 즉, GC가 객체를 수거하더라도 내부적으로 사용하던 리소스는 여전히 점유될 수 있다. 따라서 명시적으로 닫아주지 않으면 프로그램이 종료될 때까지 해당 리소스가 해제되지 않으며, 점점 더 많은 메모리와 CPU 리소스를 소비하게 될 수도 있다.

### throws

throws 키워드는 예외를 위임하기 위한 키워드이다. 예외에 대한 처리를 현재 메서드를 호출한 곳으로 위임하는 것이다. 사실상 예외처리를 진행할 수 없으니 다른 곳에서 해결하라는 의미와 같다. 

이 throws 키워드는 다음과 같이 사용이 가능하다. 

```java
public void readFile() throws IOException {
	// 파일 읽어오기
}
```

위의 예시코드와 같이 메서드 선언부에 사용하며, `throws` 다음에 오는 예외들이 발생할 수 있음을 알려주는 역할을 한다. 예외는 위의 예시인 `IOException` 뿐만 아니라 쉼표 구분을 통해 여러 예외의 다중 선언이 가능하다.

바로 다음의 예외 종류에서도 설명하겠지만 throws 키워드는 Checked Exception를 주로 명시하는데 사용된다. 

## 예외의 종류 

```
Throwable
 ├── Error (프로그램 복구 불가능)
 ├── Exception (일반적인 예외)
      ├── Checked Exception (컴파일 타임 예외)
      ├── Unchecked Exception (런타임 예외)
```

자바의 모든 예외, 즉 Exception은 Throwable이라는 클래스를 구현하고 있음. 이 Throwable 클래스는 자바에서 모든 예외와 오류의 슈퍼 클래스가 된다. 각각의 예외 종류를 알아본다.

### Error

시스템의 비정상적인 상황이 발생하는 경우. 이 경우는 프로그램이 복구할 수 없는 심각한 문제가 발생했음을 의미한다. 

개발 코드에서 Error를 잡아봤자 사실 할 수 있는 것이 없다. 굳이 예외로 잡지 않는 것이 좋다. 이런 Error의 경우에는 별도의 조치가 필요한 수준이기 때문에 로그를 남겨 문제의 원인을 파악하도록 해야한다. 알람이나 notification을 연동하는 방법 등을 고려해볼 수 있다.

대표적인 Error의 예시로 아래 Error 들이 존재한다.
- `OutOfMemoryError` (메모리 부족)
- `StackOverflowError` (스택 오버플로우)
- `VirtualMachineError` (JVM 관련 오류)

### Exception(Checked)

컴파일에 의해 검사되는 예외로 이해하면 쉽다. 때문에 Checked Exception는 컴파일러가 예외 발생 예상 지점에서 **예외처리를 강제**한다. 고로 예외를 처리할 수 있는 try-catch나, throws를 반드시 사용해야만한다는 특징이 있다.

이 예외는 애플리케이션에서 예상하고 복구해야만하는 예외를 의미한다. 

`IOException` : 파일이 존재하지 않을 가능성이 있기 때문에 **컴파일 타임에 예외 처리를 강제**해야 한다.
`ClassNotFoundException` : 특정 클래스가 존재하지 않는 경우를 반드시 대비해야 하기 때문.


복구할 수 없다면 RuntimeException이나, 적절한 추상화된 예외로 전환해서 던져야함. 

### Runtime-Exception(Unchecked)

컴파일러에 의해 검사되지 않기 때문에 Unchecked라고 불린다. 이 예외는 예외처리를 강요받지 않는다.

애플리케이션이 일반적으로 예상하거나 복구할 수 없는 예외를 말한다. 일반적으로 논리 오류나 부적절한 API 사용과 같은 프로그래밍 버그를 나타낸다. 

## Checked Exception에 대한 논쟁

최근에는 Checked Exception을 사용하지 않는 방향으로 나아가고 있음. 
Java를 제외한 Kotlin에서도 기타 최신언어들에서 Checked Exception은 지원하지 않음. 


- [checked excpetion은 자바의 가장 큰 실수다.](https://literatejava.com/exceptions/checked-exceptions-javas-biggest-mistake/)
- [checked excpetion은 사악하다](https://phauer.com/2015/checked-exceptions-are-evil/)

## 좋지 않은 예외처리란?


### 예외를 무시하는 코드

예외는 프로그램 흐름의 제어를 위한 도구가 아니라, 프로그램의 비정상적인 상태나 오류를 다루기 위한 방법으로 사용되어야 한다. 

```java

try {
	// code
} catch(IOException e) {
	e.printStackTrace();
	// do nothing
}

```


### 무의미한 throws

throws 무한 지옥 아무런 정보가 없는 Exception throws

```java
public void method1() throw Exception {
	method2()
}

public void method2() throw Exception {
	method3()
}

public void method3() throw Exception {
	throw new Exception();
}
```

