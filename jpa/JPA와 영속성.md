---
title:
description:
author: codongmin
date: 2025-02-01T21:34:00
categories:
  - 자바
tags: [""]
image:
  path: /assets/img/thumbnail/.png
  lqip:
  alt: 이미지 설명
---


## JPA 란 

자바 진영의 ORM 기술 표준. 
쉽게말하면 객체와 SQL을 서로 매핑해주는 기술. 

(JPA 구조 그림)


과거 자바 빈즈라는 기술 표준에 엔티티 빈이라는 ORM 기술이 포함되어있었지만, 너무 복잡하고 설정이 까다로운 문제, 호환성도 좋지않아 자바 엔터프라이즈 애플리케이션 서버에서만 동작했었음. 

이때 이러한 불편함을 해결해줄 오픈소스 ORM 프레임워크가 등장했는데, 기존 기술에 비해 가볍고 실용적이었음. 바로 하이버네이트, 하이버네이트를 기점으로 새로운 자바 ORM 기술 표준이 많들어졌는데 바로 JPA.

JPA는 자바 ORM 기술에 대한 API 표준 명세. ORM기술을 구현하기 위한 인터페이스들이 모여 있다. 
즉, JPA는 ORM기술에 대한 구현체가 아니라 추상화된 인터페이스. 이를 구현한 ORM 구현체중 대표적인 것인 하이버네이트이다.


## 주요 버전별 차이 
**🔹 JPA 1.0 (Java EE 5, 2006년 출시)**
📌 **기본적인 ORM 기능 도입**

- 엔티티(Entity), 필드 매핑(@Entity, @Table, @Column 등)
- 기본적인 연관관계 매핑(@OneToOne, @OneToMany, @ManyToOne, @ManyToMany)
- JPQL(Java Persistence Query Language) 도입
- EntityManager를 통한 영속성 관리 (CRUD 기능 지원)
- 기본적인 트랜잭션 관리

---
**🔹 JPA 2.0 (Java EE 6, 2009년 출시)**
📌 **기능 확장 및 성능 개선**

- **Criteria API** 도입 (타입 세이프한 쿼리 작성 가능)
- **ElementCollection** 지원 (값 타입 컬렉션 사용 가능)
- **MappedSuperclass** 및 `@Inheritance` 전략 강화
- `@OneToMany` 관계에서 `orphanRemoval = true` 옵션 추가
- 2차 캐시(Second Level Cache) 지원
- `@OrderColumn` 추가 (컬렉션 순서 유지)
- JPQL 기능 확장 (서브쿼리 지원 강화)

---

### **🔹 JPA 2.1 (Java EE 7, 2013년 출시)**

📌 **추가적인 매핑 및 쿼리 기능 강화**

- **Entity Graph** 도입 (`@NamedEntityGraph`, `@EntityGraph`)
- **Stored Procedure (저장 프로시저) 호출 지원**
- `@AttributeConverter` 추가 (타입 변환 기능)
- `@NamedStoredProcedureQuery` 추가
- `@ConstructorResult` 추가 (JPQL에서 DTO 매핑 가능)
- `CriteriaUpdate`, `CriteriaDelete` 지원 (Criteria API에서 업데이트 및 삭제 가능)

**🔹 JPA 2.2 (Java EE 8, 2017년 출시)**
📌 **Java 8 기능 지원**

- **Java 8 날짜/시간 API 지원 (`LocalDate`, `LocalDateTime` 등)**
- JPQL에서 `STREAM` 처리 기능 추가
- `@QueryHint` 확장

 **🔹 JPA 3.0 (Jakarta EE 9, 2021년 출시)**
📌 **Jakarta EE로 전환 및 현대화**
- **패키지 변경 (`javax.persistence → jakarta.persistence`)**
- Reflection 기반의 **부트스트래핑 제거**
- `@OrderBy` 속성 정렬 기능 강화
- **XML 매핑 메타데이터 정리** (XML 설정 방식 축소)
- `PersistenceUnitUtil` 확장

---

**🔹 JPA 3.1 (Jakarta EE 10, 2022년 출시)**
📌 **최신 자바 기능 적용 및 편의성 개선**
- `@Converter(autoApply = true)` 기본 적용 가능
- `FROM` 절에서 서브쿼리 지원 (`FROM (SELECT ...)`)
- **JPQL에서 `SELECT VALUE` 지원**
- JSON과 같은 복합 데이터 타입 매핑 지원 확대


## 영속성 관리 

JPA 가 제공하는 기능은 크게 2가지 
- 엔티티 & 테이블 매핑하는 설계 
- 엔티티 실제 사용하는 기능


JPA에서 가장 핵심이되는 객체가 바로 엔티티 매니저. 

엔티티를 CRUD 엔티티와 관련된 모든일을 처리. 엔티티를 관리하는 관리자. JPA를 사용하면 데이터베이스와 직접 연결이 아니라 엔티티 매니저를 통해 작업을 수행한다.


#### 엔티티 매니저의 생명주기 

엔티티매니저 팩토리에서 생성
- 애플리케이션 실행 시 **한 번만 생성**해야 함


- 일반적으로 **트랜잭션 단위(요청별)로 생성하고 사용 후 닫음**
- 멀티스레드 환경에서 공유하면 **안됨** (Thread-unsafe

#### 엔티티 매니저 역할 

엔티티의 라이프 사이클을 관리하고 트랜잭션을 관리 
엔티티 매니저는 매 트랜잭션마다 생성되는 구조. 


| 메서드                      | 설명                        |
| ------------------------ | ------------------------- |
| `persist(entity)`        | 엔티티 저장 (영속 상태로 변경)        |
| `find(entityClass, id)`  | 엔티티 조회 (1차 캐시 먼저 확인)      |
| `remove(entity)`         | 엔티티 삭제                    |
| `merge(entity)`          | 준영속 상태의 엔티티를 다시 영속 상태로 변경 |
| `createQuery(JPQL)`      | JPQL 쿼리 실행                |
| `createNativeQuery(SQL)` | 네이티브 SQL 실행               |
| `flush()`                | 변경 사항을 DB에 반영             |
| `clear()`                | 영속성 컨텍스트 초기화 (1차 캐시 제거)   |
| `close()`                | EntityManager 종료          |