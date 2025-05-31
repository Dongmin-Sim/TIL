---
title: 스프링 배치 기본 개념
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


## 배치란? 

배치는 여러 작업을 일정한 단위로 묶어 한꺼번에 처리하는 방식을 의미한다. 배치 프로그램은 정해진 시간에 일괄적으로 작업을 처리하는 프로그램을 말한다. 실시간성으로 작업을 처리하는 것이 아닌, 정해진 시간에 몰아서 처리하는 특성을 갖는다. 

서비스를 운영하는 관점에서는 요청에 따라 지금 당장 처리될 필요는 없지만, 주기적으로 처리해주어야하는 일들이 배치성 작업에 속한다. 주로 필요한 데이터를 모아서 한꺼번에 처리해야하거나, 지연처리가 필요할 때, 시간이 오래걸리거나 대량의 데이터를 처리해야하는 상황들이 있다. 

예를 들면 월말에 기존에 발생했던 내역들을 종합해서 월별 거래 명세서를 만든다거나 매일 자정에 사용자 포인트를 정산한다거나, 누락된 데이터들을 주기적으로 검증하거나 보증해야하는 경우가 예시들이 될 수 있다.

## 배치 예시 

### 서비스 분야에서의 배치 처리  

1. 메시지, 이메일, 푸시 등을 발송할 때, 
2. 데이터를 마이그레이션할 때, 
3. 실패한 트랜잭션을 재처리할 때, 
4. 쿠폰, 포인트 등이 만료되었을 때 소진시키는 처리를 할 때, 
5. 월말 또는 월초에 특정 데이터를 생성할 때, 월별 거래 명세서

### 데이터 분야에서의 배치 처리 

1. 각 서비스의 데이터를 데이터 웨어하우스에 저장할 때, (ETL, Extract-Transform-Load)
2. 아마존에서 연관 상품을 추천하는 데이터 모델을 만들 때, 
3. 유저의 리텐션 등 마케팅에 참고할 데이터 지표를 집계할 때


## Spring Batch란 

가볍고 다양한 기능을 가진 배치 프레임워크. 견고한 배치 애플리케이션 개발이 가능하도록 설계된 프레임워크.

스프링 배치 프레임워크는 스프링 기반으로 작성되어서 서비스에 작성했던 코드들을 재활용해서 사용이 가능하다는 특징이 있다. 기존 모듈을 스프링 배치에서 Import 해서 사용할 수도 있다. 덕분에 생산성 있게 사용이 가능하다.

대용량 데이터 배치처리에 필요한 다양한 기능들을 제공한다. 
- 로깅과 추적, 트랜잭션 관리, 작업 처리 통계.
- 청크 기반 작업 처리.
- 실패 복구, 재시작/건너뛰기 가능.
- 스케줄링 연동 용이.
- 다양한 입출력 (Reader, Writer) 지원.
- 멀티 코어, 또는 멀티 서버에서 처리 분산 기능.


스프링 배치는 크게 다음과 같은 구성 요소들로 이루어져있다.

![batch-diagram](https://docs.spring.io/spring-batch/reference/_images/spring-batch-reference-model.png)


먼저 배치처리할 작업을 Job이라고 부른다. 이 Job을 실행시키는 역할을 하는 컴포넌트가 JobLauncher이다. 일종의 트리거라고 볼 수 있다. 

Step은 Job을 처리하기 위한 단계를 의미한다. 작업을 처리하기 위한 단계는 여러 단계가 존재할 수 있으므로 1:N 관계를 갖는다. 하나의 Job이 여러 Step을 가질 수 있음을 의미한다.

JobRepository 는 Job, Job 실행 이력, 파라미터, Step 정보 등을 저장한다. 이러한 정보들은 Job이 중단되었을 때 재시작을 하거나, 혹은 중복 실행을 방지하기 위한 기능들을 위해 저장된다.

예를 들어 "회원 포인트 정산"이라고 하는 배치 작업에 비유를 해보면, 
- "회원 포인트 정산" Job이 되고
- "회원 정보 읽기 → 포인트 계산 → DB 저장" 와 같은 세부 단계가 Step이 된다.

이때 Step은 크게 세 가지 단계로 구성된다. 보통 배치처리를 할 때 데이터를 읽어오고 읽어온 데이터에 재처리 후 쓰는 구조로 이루어져있다. 

조금 더 세부 구조에 대해서 자세하게 정리해본다. 


### Job 

![Job](https://docs.spring.io/spring-batch/reference/_images/job-stereotypes-parameters.png)

위 그림은 Job의 계층구조를 표현한 그림이다. 
**Job**은 **전체 배치 프로세스를 캡슐화한** 도메인을 의미한다. **논리적인 배치 처리 단위**인 셈이다. 비유하자면 객체를 만들기 위한 클래스-like에 비유해볼 수 있다. Job에는 어떤 Step들을 **어떤 순서로 실행할지가** 정의되어 있다.

```java
@Bean
public Job footballJob(JobRepository jobRepository) {
    return new JobBuilder("footballJob", jobRepository)
                     .start(playerLoad())
                     .next(gameLoad())
                     .next(playerSummarization())
                     .build();
}
```


**JobInstance**는 Job과 JobParameter의 조합으로 이루어진 실질적인 실행 단위가 된다. JobParameter는 Job 실행 시 입력되는 **외부 파라미터**이다. 예를 들어 날짜, 사용자 ID 등이 이에 해당된다.
동일한 Job이더라도, JobParameter가 다르다면 이는 다른 JobInstance로 식별된다. 예를 들면 "회원 포인트 정산Job" + "오늘날짜, parameter"의 조합이 유일한 JobInstance로 식별된다. 

**JobExecution**은 특정 JobInstance의 **실행 기록**을 의미한다. 실패/재시도할 때마다 새로운 JobExecution이 생성된다고 볼 수 있다. 상태: `STARTING`, `STARTED`, `FAILED`, `COMPLETED`, `STOPPED` 등



### Step

Step은 Job을 수행하기 위한 세부 단계, 작업 처리의 단위이다. 
주로 Chunk 기반과 Tasklet 기반인 2가지 종류의 Step으로 나뉜다.

#### Chunk-Step


#### Tasklet-Step