---
title: "세마포란?"
date: 2023-09-17 16:20:02 +05:00
categories:
    - OS
tags:
    - Post Formats
    - readability
    - standard
permalink: /semaphore
---

시스템프로그래밍, 데이터베이스를 공부하다보면 계속해서 세마포 개념이 등장해서 명확하게 정리하고자 한다. 

## 동기화 배경 지식

세마포, 스핀락은 동기화 작업을 수행하기 위한 방법이다. 동기화는 공유 데이터를 일관성을 유지하며 작업하는 것을 의미한다. 이 때 동기화를 처리하기 위한 공간을 임계 공간이라고 한다. 정리하면 세마포나 스핀락과 같은 방법은 임계 영역 안에서 처리 방법에 관한 내용이다. 

## 스핀락?

스핀락은 세마포보다 비교적 간단한 개념으로 스핀락의 한계로 인해서 세마포 개념이 정립되었기 때문에 스핀락부터 알아보자. 임계 영역이라는 자물쇠를 열고 들어 가기 위해서 열쇠가 필요하다. 스핀락은 계속해서 열쇠를 탈취하기 위해서 공유 변수 값에 접근한다. 이 과정에서 TestAndSet 이라는 cpu 레벨에서 쓰이는 함수가 쓰인다. TestAndSet은 실행 중간에 간섭을 받지 않고 같은 메모리 영역이라면 동시에 실행할 수 없다. 이러한 특징때문에 동기적으로 해당 함수를 사용할 수 있다. TestAndSet이 동기적으로 처리된다고 해도 하나 프로세스 혹은 스레드가 임계 영역에서 작업을 하는 동안 다른 프로세스 혹은 스레드는 계속해서 TestAndSet 함수를 호출해야 하기 때문에 cpu 낭비로 이어진다. 

## 세마포란?

여러 스핀락의 한계를 극복하기 위해서 세마포가 탄생하였다. 

세마포는 큐라는 대기 공간을 마련하여서 cpu 소비를 최소화하였다. 음수가 아닌 정수를 설정하여서 공유 데이터에 다중 이용자가 접근할 수 있도록 한 것도 하나 특징 중 하나이다. 세마포는 크게 2가지 함수로 구성되어 있다. 

```java
# P 연산 (자물쇠 열거나 자물쇠 잠구는 경우) 

if (S > 0) then S ← S -1;

else wait on the queue Q;
```

```java
# P 연산 (자물쇠 열거나 자물쇠 잠구는 경우) 
if (S > 0) then S ← S -1;

else wait on the queue Q;
```

```java
# 일끝내고 나갈 때 V 실행 

if (waiting process on Q) → then wake up on of them; else S ← S + 1
```

P연산을 통해서 0이상 S가 있으면 임계 공간에 접근하고 그렇지 못하면  q 에 대기한다. 동기화 작업을 마치면 V 함수를 수행하여서 대기하는 Q가 있으면 깨우고 나가고 그렇지 않으면 S 을 증가하고 나간다. 이러한 세마포 작업으로 인해서 순차적으로 작업을 수행할 수 있다. 

### 참고 자료

[스핀락(spinlock) 뮤텍스(mutex) 세마포(semaphore) 각각의 특징과 차이 완벽 설명! 뮤텍스는 바이너리 세마포가 아니라는 것도 설명합니다!](https://youtu.be/gTkvX2Awj6g?si=kFrQKXW7qRHnzboZ)

[[OS ] Lec 6. Process Synchronization and Mutual Exclusion (5/7) - Semaphore (OS supported Sol. 2)](https://youtu.be/CitsUz-Dx7A?si=5sXXZE7kPky8FwBT)

[[OS] 뮤텍스와 세마포어 (Mutex and Semaphore)](https://hoyeonkim795.github.io/posts/mutex-semaphore/)