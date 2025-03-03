---
title: 빈 설정 클래스 어노테이션
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


## @Configuration 이란?

`@Configuration` 어노테이션은 클래스에 적용되는 어노테이션이다. 
그리고 이 어노테이션이 붙은 클래스는 **스프링 빈(Bean)을 등록/설정하는 클래스** 라는 것을 의미한다. 

즉, **`@Configuration`은 스프링 빈을 생성하고 관리하는 설정을 담당**하는 역할을 함.


다음과 같이 `@Bean`어노테이션으로 스프링을 빈을 생성하는 메서드를 정의함으로 스프링에 빈을 등록할 수 있다.

```java
@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        System.out.println("MyService 빈 생성");
        return new MyService();
    }
}
```

## 동작 방식 

- `@Configuration`이 붙은 클래스는 **스프링이 프록시 객체를 생성하여 관리**한다.
- 내부적으로 **CGLIB**(바이트코드 조작 라이브러리)를 사용하여 **싱글턴 보장**을 보장한다.

## 빈 등록 원리

**스프링의 컴포넌트 스캔(Component Scan) 기능** 으로 `@Configuration`이 붙은 클래스들도 빈으로 등록되어 관리된다.

`@Configuration`은 **설정 클래스 전용** 어노테이션이며, 내부적으로 `@Component`를 포함하고 있음.

```java
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Component
public @interface Configuration {

}
```

`@Component`는 `@ComponentScan`에 의해서 자동적으로 스프링 빈(Bean)으로 등록이 된다. 
이 `@ComponentScan` 은 SpringBoot 프로젝트를 처음 생성하면 존재하는 @SpringBootApplication 어노테이션에 안에 존재한다. 

```java
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Inherited  
@SpringBootConfiguration  
@EnableAutoConfiguration  
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),  
       @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })  
public @interface SpringBootApplication {
}
```

따라서 `@ComponentScan`이 `@Configuration` 클래스도 자동으로 빈으로 등록함.

`@Controller`, `@Service`, `@Repository` 와 같은 어노테이션들도 내부에 `@Component` 어노테이션을 가지고 있기 때문에 동일하게 적용된다. 


