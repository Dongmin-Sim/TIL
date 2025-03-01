---
title: 예외
description: 자바의 예외
author: codongmin
date: 2025-02-25
categories:
  - java
tags:
  - exception
  - checked-exception
  - runtime-exception
image:
  path: /assets/img/thumbnail/_.png
  lqip: 
  alt:
---

## 들어가며

프로그램을 개발하고, 실행하는 과정 속에서는 여러가지 이벤트들이 벌어지곤 한다. 그 중에서도 사용자의 잘못된 입력이라던가, 네트워크의 연결이 원활하지 않거나, 예상하지 못했던 수 많은 예외적인 이벤트들이 발생해 프로그램을 동작을 방해한다. 

예외적인 이벤트들은 비단 자바 뿐만 아니라 어떤 프로그래밍 언어를 사용하더라도 발생하게 된다. 이런 이벤트들을 적절히 처리하지 않으면 시스템의 안정성과 신뢰성을 보장할 수 없다. 따라서 개발자들은 견고한 프로그램을 위해 이러한 이벤트들을 잘 핸들링할 수 있어야한다.

본 글에서는 예외란 무엇인지, 자바에서는 어떻게 예외 처리를 지원하는지에 대해 알아본다. 이어서 자바의 예외 종류, 좋지 못한 예외 처리 패턴들을 살펴본다. 예외에 대해서 생소하거나, 자바에서 예외를 어떻게 다루는지 궁금한 경우 이 글의 대상 독자이다.

## 예외란?

예외란 프로그램의 **정상적 흐름을 벗어난 예외적**인 사건, 상황을 의미한다.

정상적 흐름에서 발생할 수 있는 예외적인 사건은 매우 여러가지가 존재한다. 프로그램 상의 오류일 수도 있고, 외부 시스템에 의한 것일 수도 있고 개발자가 의도한 것일수도 있다.

예외들의 종류는 다양하다.
- 파일이 존재하지 않거나 읽을 수 없을 때
- 네트워크 연결이 끊어졌을 때
- SQL 구문 오류가 있을 때
- 데이터베이스 연결 실패 시
- 수학적 연산 중에 0으로 나누려고 할 때
- 재귀 호출이 너무 깊어져서 스택 메모리가 초과되는 경우
- 메모리 할당이 부족할 때 


## 예외를 마주하게 된다면

예외가 발생하면 프로그램 코드 내에서 취할 수 있는 일은 다음과 같다.

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

또한, 예외를 변환하는 과정에서 예외 체인을 유지하는 것이 좋다. 즉, 원본 예외를 **`cause`** 로 포함시켜서, 나중에 예외를 추적할 때 원인 파악이 용이하도록 한다.

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

`try` 블록은 우리가 기대하는 정상 흐름이 담긴 코드이자 예외가 발생할 가능성이 존재하는 코드가 위치하는 블록이다. 이때 `try` 블록에서 예외가 발생하면 해당 예외를 처리할 수 있는 `catch` 블록으로 넘어오게 된다. `catch` 블록에서는 인자로 넘어온 예외를 가지고 로깅을 하거나, 재시도를 시도하거나 예외를 복구할 수 있는 코드를 작성할 수 있다. 이 공간은 예외 처리를 수행하기 위한 목적의 공간이다. 

`catch` 문은 여러 블록으로 작성이 가능하다. 위에서 아래로 내려오면서 필터처럼 `try`문 안에서 발생하는 예외와 일치하는 `catch` 블록이 실행된다.

```java
try {
	// 코드 실행
} catch (FileNotFoundException e) {
	// FileNotFoundException일 경우 처리
} catch (IOException e) {
    // IOException일 경우 처리 
}
```

위에서 아래로 내려오면서 


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


만일 아래와 같이 `try`문에서 예외가 발생하지 않고 정상 흐름으로 진행되다가 `return` 을 만나게 된다면 바로 메서드가 종료될까? `finally` 구문이 실행이 될까?

```java
public static String readFile(String filePath) {
	FileReader fileReader = null;
	BufferedReader bufferedReader = null;

	try {
		fileReader = new FileReader(filePath);
		bufferedReader = new BufferedReader(fileReader);
		
		String content = bufferedReader.readLine();
		return content; // 메서드 종료
	} catch (IOException e) {
		// 예외처리
	} finally {
		// finally 블럭이 실행될까?
	}
}
```

`finally` 블럭은 실행이 실행된 후에야 비로소`return` 구문이 실행된다.

### try-with-resources

자바에서는 자원 해제 방식을  `try` 문의 괄호 안에서 선언한 리소스를 자동으로 정리해주는 문법도 지원한다.
대신 괄호 안에 올 수 있는 리소스는 `AutoCloseable` 인터페이스를 구현한 리소스만 위치할 수 있다. 이 인터페이스에는 리소스를 정리하는 close 메서드를 구현해야하도록 정의하고 있다.
```java
try (BufferedReader br = new BufferedReader(new FileReader("example.txt"));) {  
    br.readLine();  
} catch (IOException e) {  
    throw new RuntimeException(e);  
}


// AutoCloseable.java
public interface AutoCloseable {
	void close() throws Exception;
}
```

### throws

throws 키워드는 예외를 위임하기 위한 키워드이다. 예외에 대한 처리를 현재 메서드를 호출한 곳으로 위임하는 것이다. 사실상 예외처리를 진행할 수 없으니 다른 곳에서 해결하라는 의미와 같다. 

이 throws 키워드는 다음과 같이 사용이 가능하다. 

```java
public void readFile() throws IOException {
	// 파일 읽어오기
}
```

위의 예시코드와 같이 메서드 선언부에 사용하며, `throws` 다음에 오는 예외들이 발생할 수 있음을 알려주는 역할을 한다. 예외는 위의 예시인 `IOException` 뿐만 아니라 쉼표 구분을 통해 여러 예외의 다중 선언이 가능하다.

바로 다음의 예외 종류에서도 설명하겠지만 `throws` 키워드는 `Checked Exception`를 주로 명시하는데 사용된다. 

## 예외의 종류 

```
Throwable
 ├── Error (프로그램 복구 불가능)
 ├── Exception (일반적인 예외)
      ├── Checked Exception (컴파일 타임 예외)
      ├── Unchecked Exception (런타임 예외)
```

자바의 모든 예외, 즉 Exception은 `Throwable`이라는 클래스를 구현하고 있다. 이 `Throwable` 클래스는 자바에서 모든 예외와 오류의 슈퍼 클래스가 된다. 하이어라키의 각 요소들과 예외 종류들을 알아본다.

### Error

시스템 발생하는 치명적인 오류를 의미한다. 이 경우는 프로그램에서 **복구할 수 없는 심각한 문제**가 발생했음을 의미한다. 

개발 코드에서 Error를 잡아봤자 사실 할 수 있는 것이 없다. 굳이 예외로 잡지 않는 것이 좋다. 이런 Error의 경우에는 별도의 조치가 필요한 수준이기 때문에 로그를 남겨 문제의 원인을 파악하도록 해야한다. 알람이나 notification을 연동하는 방법 등을 고려해볼 수 있다.

대표적인 Error의 예시로 아래 Error 들이 존재한다.
- `OutOfMemoryError` (메모리 부족)
- `StackOverflowError` (스택 오버플로우)
- `VirtualMachineError` (JVM 관련 오류)

### Exception(Checked)

자바는 예외를 크게 2가지로 나누어서 표현한다. 예외를 나누는 큰 분류 기준은 컴파일러가 예외처리를 강제하는지 하지 않는지에 따라 나뉜다. 

두 예외 중 컴파일에 의해 검사되는 예외들을 Checked Exception이라 부른다. Checked Exception은 기본적으로 **예외 발생이 예상**되고 **복구가 가능한 예외**들을 표현하기 위한 목적으로 설계되었다. 때문에 Checked Exception는 컴파일러가 예외 발생 예상 지점을 확인하고 개발자에게 **예외처리를 강제**한다. 고로 예외를 처리할 수 있는 try-catch나, throws를 반드시 사용해야만 한다. 그렇지 않으면 컴파일이 불가능하다.

에를 들어, 대표적인 Checked Exception들은 다음과 같다. 

`IOException` : 파일이 존재하지 않을 가능성이 있기 때문에 **컴파일 타임에 예외 처리를 강제**해야 한다.
`ClassNotFoundException` : 특정 클래스가 존재하지 않는 경우를 반드시 대비해야 하기 때문.

### Runtime-Exception(Unchecked)

Unchecked Exception은 복구가 어렵거나, 할 수 없는 프로그램사의 예외를 나타내기 위해 설계되었다. 컴파일러에 의해 검사되지 않기 때문에 Unchecked라고 불린다. 이 예외는 예외처리를 강요받지 않는다. Unchecked Exception에 해당되는 예외들은 `RuntimeException` 이라는 별도의 클래스 상속받는다. 

애플리케이션이 일반적으로 예상하거나 복구할 수 없는 예외들이 이에 해당된다. 일반적으로 논리 오류나 부적절한 API 사용과 같은 프로그래밍 버그를 나타낸다. 

대표적으로 부적절한 인자가 전달될때 발생하는 `IllegalArgumentException`이나 유효하지 않은 인덱스에 접근할 때 발생하는 `IndexOutOfBoundsException`가 존재한다.

## Checked Exception에 대한 논쟁

최근에는 Checked Exception을 사용하지 않는 방향으로 나아가고 있음. Java를 제외한 Kotlin에서도 기타 최신언어들에서 Checked Exception을 사용하지 않는 방향으로 변화하고 있다. 

이러한 방향성의 대표적인 이유들은 다음과 같다. 

Checked Exception의 경우 예외를 처리하도록 컴파일러에서 강제하기 때문에, try-catch 블록을 작성하거나, throws 키워드를 추가해야만 한다. 

Checked Exception는 개발자가 복구해야만 하는 예외를 표시한 것이지만, 만약 호출된 곳에서 예외를 처리할 수 없는 경우에 이 예외를 호출한 곳으로 위임을 해야하는데, 당장에 예외를 처리할 수 없는 상황이라면 적절한 예외처리를 할 수 있는 곳까지 호출 스택을 따라 exception이 전파되는 현상이 발생하게된다. 

```java
public class ExceptionTestClass {  
  
    public static void main(String[] args) {  
        try {  
            methodA();  
        } catch (IOException e) {  
            // 예외처리  
        }  
    }  
  
    public static void methodA() throws IOException {  
        methodB();  
    }  
  
    public static void methodB() throws IOException {  
        methodC();  
    }  
  
    private static void methodC() throws IOException {  
        CustomReader aa = new CustomReader("aa");  
    }  
}
```

이런 형태가 되어버리면 `methodA`과 `methodB` 는 예외를 단순히 전달, 위임만 하지만, 특정 예외에 강하게 결합되는 결과가 되어버린다. 만약 `CustomException`이 다른 예외로 변경된다면 throws 로 전파된 코드들에도 변경이 불가피해진다. 뿐만 아니라 모든 Checked Exception에 대해서 `try-catch` 하는 코드의 가독성도 그리 좋지 못하다는 문제점이 있다.

자바에서는 이런 경우 Unchecked Exception으로 예외를 변환하는 방식으로 이 문제를 해결할 수 있다. Unchecked Exception은 `RuntimeException`을 상속한 예외로, 컴파일 타임에 예외 처리를 강제하지 않기 때문에 코드의 결합도를 줄이고 더 유연하게 예외를 처리할 수 있다.

```java
public class ExceptionTestClass {  
  
    public static void main(String[] args) {  
        try {  
            methodA();  
        } catch (RuntimeException e) {  
            // 예외처리  
        }  
    }  
  
    public static void methodA() {  
        methodB();  
    }  
  
    public static void methodB() {  
        methodC();  
    }  
  
    private static void methodC() {  
        try {  
            CustomReader aa = new CustomReader("aa");  
        } catch (IOException e) {  
            throw new RuntimeException(e);  
        }  
    }  
}
```

Unchecked Exception을 사용하면 `methodC`과 `method2`가 예외를 던지더라도, 이를 호출하는 곳에서 예외를 처리할지 여부를 선택할 수 있다. 이렇게 예외를 처리할 필요 없이 더 간결한 코드로 유지할 수 있게 되며, 예외 처리의 강제성을 줄여서 코드의 유연성을 높일 수 있다.


아래 글은 Checked Exception에 대해 다소 비판적인 의견이 포함된 글들이다.
- [checked excpetion은 자바의 가장 큰 실수다.](https://literatejava.com/exceptions/checked-exceptions-javas-biggest-mistake/)
- [checked excpetion은 사악하다](https://phauer.com/2015/checked-exceptions-are-evil/)

## 좋지 않은 예외처리란?

### 예외를 흐름제어로 사용하는 경우

예외는 프로그램 흐름의 제어를 위한 도구가 아니라, 프로그램의 비정상적인 상태나 오류를 다루기 위한 방법으로 사용되어야 한다. 

```java
public class FlowControlExample {
    public static void main(String[] args) {
        try {
            // 사용자가 입력한 값에 따라 분기
            if (someCondition) {
                throw new RuntimeException("정상적인 흐름에서 예외를 던짐");
            } else {
                System.out.println("정상적인 처리");
            }
        } catch (RuntimeException e) {
            System.out.println("예외로 흐름을 제어한 경우: " + e.getMessage());
        }
    }
}
```

`try` 문에서 정상 흐름일 경우에도 조건에 맞지 않는다면 `catch`문으로 이어지기 때문에 실제로 이것이 예외 상황으로 발생한 것인지 하나의 분기 흐름인 것인지 코드를 읽고 이해하는 시간이 한번 더 필요하기 때문에 가독성이 좋지 않다. 

또한 예외가 발생할 때는 예외 객체가 생성되고, (객체 생성 비용) 이 예외를 처리하기 위한 스택 트레이스를 추적하는 과정이 발생을 하게 된다. 이 예외가 어디서, 왜 발생했는지 전체 추적 정보를 저장해야하는 스택 트레이스 정보 또한 처리를 해야하기 때문에 성능상에도 그리 좋지 않다. 즉, 처리 비용이 일반적인 분기문에 비해 크다. 때문에 굳이 예외를 이용해 흐름제어를 해야할 이유가 없다.

### 예외를 무시하는 코드

예외 발생을 무시하도록 작성해서는 안된다. 단순히 예외를 잡아서 로깅만 한다던가 이에 대한 처리를 묵인해버리는 경우는 바람직한 코드가 아니다. 아래 코드를 보자.

```java

try {
	// code
} catch(IOException e) {
	logger.error("파일을 읽는 중 오류 발생: " + e.getMessage());
	// do nothing
}

```

위 코드의 경우 예외를 로깅한다는 행위를 통해서 catch 문에서 '무언가'처리한 것 같은 느낌을 주지만, 사실 어떠한 처리도 진행하지 않았다. 이 catch 문이 지나가버린 이후에는 이 예외가 발생했다는 것을 어떠한 상위 메서드에서도 알 수 없다. 문제가 발생했지만, 나만 볼 수 있는 메모장에 적어두고 다른 할일을 하는 것과 동일하다. 

로깅을 남기는 행위는 예외를 해결하는 행위가 아니기 때문이다. 예외는 비정상적인 이벤트로 이는 복구되거나, 처리가 되어야 하는 대상이다. 예를 들어 예외가 발생할 경우 사용자에게 예외가 발생함을 알리고, 정책에 따라 기본 설정값으로 반환을 한다거나, 적절한 예외로 변환해서 다시 상위 메서드에게 전달하는 방법들이 이에 해당된다.


### 추상적인 예외 사용

구체적이지 않는 예외를 사용을 지양해야한다. `Exception` 클래스는 자바의 예외 계층 구조에서 **모든 예외의 최상위 클래스**이다. 즉, `Exception`은 구체적인 예외 정보를 담기에는 무리가 존재한다.  

```java
public void readFile(String fileName) throws Exception {
    if (fileName == null) {
        throw new Exception("파일 이름이 null일 수 없습니다.");
    }
    File file = new File(fileName);
    if (!file.exists()) {
        throw new Exception("파일이 존재하지 않습니다: " + fileName);
    }
    // 파일 읽기 작업
}
```

위 코드에서는 입력 인자가 잘못되어도, 파일이 실제로 존재하지 않아도 모두 Exception이라는 추상적인 예외 타입으로 두 가지 다른 속성의 예외가 표현된다. 그러면 포함되는 메시지를 통해서 예외를 구분하면되지 않을까 하지만, 메시지를 통해서 예외를 구분하는 방식을 다음과 같은 이유로 그리 좋은 방식이 아니다. 

메시지의 형식이나, 내용이 일관적이지 않을 수 있다. 동일한 상황에 대해서 문자열을 동일하게 작성하는 것은 어렵고, 한글자만 달라져도 이를 쉽게 파악할 수 없다. 또 메시지 변경을 해야한다면, 해당 문자열을 사용하는 모든 코드를 수정해야만한다. 또한 문자열만으로는 예외의 원인이나 성격이 바로 파악하기가 어렵다. 한번 문자열을 읽고 이게 어떤 예외상황인지 이해하고 변환하는 과정이 일어나므로 가독성이 좋지 않다.

아래와 같이 상황에 맞는 구체적인 예외 클래스를 사용해 예외 상황을 구분하는 것이 좋다. 

```java
public void readFile(String fileName) throws FileNotFoundException {
    if (fileName == null) {
        throw new IllegalArgumentException("파일 이름이 null일 수 없습니다.");
    }
    File file = new File(fileName);
    if (!file.exists()) {
        throw new FileNotFoundException("파일이 존재하지 않습니다: " + fileName);
    }
    // 파일 읽기 작업
}

```

## 마무리하며

예외란 무엇인지 알아보고 예외상황을 마주했을때 취할 수 있는 액션들에 대해서 알아보았다. 이어서 자바에서는 예외를 처리하기 위한 구체적인 매커니즘과 문법, `try-catch`, `throws`에 대해서 정리했다. 자바에서 지원하는 예외의 종류(Checked Exception, Unchecked Exception)에 차이점에 대해서 알아보고 Checked Exception에 대한 논쟁을 살펴보며 Checked Exception 가지는 한계점에 대해서도 간략하게 알아봤다. 예시를 통해서 각 상황별로 예외를 처리하는 안티패턴들에 대해서 이야기해보았다. 

적절한 복구와 위임을 통한 예외 처리는 예상하지 못한 예외들을 핸들링하여 소프트웨어의 안정성을 확보하는 가장 기본적인 방법이자 코드의 가독성과 유지보수성을 높이는 중요한 접근법이다. 그러나 예외를 무분별하게 사용하거나, 예외 메시지에 의존하는 등의 **안티패턴**을 피하고, **구체적이고 명확한 예외 처리**를 통해 예외 발생 원인에 대한 적절한 대응을 하는 것이 중요하다.