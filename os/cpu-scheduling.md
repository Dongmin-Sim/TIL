---
title: CPU 스케쥴링
description: 스케쥴링에 대하여
author: codongmin
date: 2025-02-14
tags: ["os", "cpu", "schedule"]
Keyword: ["선점", "비선점", "기아상태"]
---

# CPU 스케쥴링이란

CPU 스케쥴링이란 어떤 작업에 CPU를 배정할지 결정하는 것이다. 컴퓨터 시스템의 효율은 어떤 프로세스에 CPU를 먼저 배정하느냐에 따라 달라진다. 이러한 작업을 총괄하는 역할이 CPU 스케쥴러다.

CPU 스케쥴러는 프로세스 스케쥴러라고도 함. 정리하자면 스케쥴링은 여러 프로세스의 상황을 고려해서 CPU와 시스템 자원을 어떻게 배정할지 결정하는 일을 의미하고 그 일을 담당하는 것이 CPU 스케줄러(프로세스 스케줄러)

CPU 스케줄링의 목적은 모든 프로세스가 공평하게 작업하도록 자원을 배분해주는 것에 있음. 공평성을 유지하면서도 안정적으로 동작하는 것을 목표로 함.

## CPU 스케줄링의 방식

CPU 스케줄링의 방식은 크게 2가지로 나뉨. "선점형 스케줄링"과 "비선점형 스케줄링"이다.

선점형 스케줄링은 어떤 프로세스가 CPU를 할당받아 실행 중이더라도 운영체제가 CPU를 강제로 빼앗을 수 있는 스케줄링 방식이다.

비선점형 스케줄링은 어떤 프로세스가 CPU를 점유하면 다른 프로세스가 이를 빼앗을 수 없는 스케줄링 방식을 말한다.

### 선점형 스케줄링

선점형 스케줄링 방식에서는 운영체제가 필요하다고 판단되면 실행 상태에 있는 프로세스의 작업을 잠시 멈추고, 새로운 작업을 시작하는 것이 가능하다. 선점형 스케줄링의 대표적인 예로는 <u>인터럽트 처리</u>가 있음.

- CPU가 인터럽트를 수신하면 현재 실행중인 작업을 멈추고 인터럽트를 먼저 처리하게 함.

선점형 스케줄링 방식의 특징으로는 프로세스가 우선순위에 따라 처리될 수 있고, 때문에 우선순위가 높은 작업일 경우 우선적으로 실행이 가능해 응답성이 빠르다. (대화형시스템에 장잠), 하지만 프로세스가 멈추고 변경되기 때문에 컨텍스트 스위칭이 자주 발생하고 그에 따른 오버헤드가 존재한다는 점.

### 비선점형 스케줄링

비선점형 스케줄링에서는 어떤 프로세스가 실행상태에 들어가 CPU 자원을 사용하고 있는 동안에는 다른 프로세스가 이를 사용할 수없음. 자발적으로 종료되거나 대기 상태로 들어가기 전까지 해당 프로세스가 CPU 자원을 사용하게 됨.

이러한 특성 때문에 비선점형 스케줄링에서는 선점형 스케줄링보다 컨텍스트 스위칭 비용은 적음. 물론 그에 따른 오버헤드도 적다. 주로 배치 처리 시스템에서 주로 사용된다. 대신에 작업 시간이 긴 프로세스의 경우 작업 시간만큼 응답시간이 느릴 수 있다.

- 비선점형 스케줄링은 작업의 우선순위가 선점형보다 크게 중요하지 않아 도착 순서이거나, 작업 시간이 짧은 프로세스를 우선적으로 처리함.
- 배치처리시스템은 사용자 간 실시간 상호작용이 필요한 작업이 아니고, 처리 순서가 명확하기 때문에 비선점형 스케줄링과 적합할 수 있음.
