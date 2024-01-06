---
title:  "Memcached VS Redis"
date: 2023-11-10 12:00:00 +0900
categories: [DB]
tags: [DB]
---

Facebook의 서버 아키텍처 관련 [블로그 글 ](https://engineercodex.substack.com/p/how-facebook-scaled-memcached)을 보고 
Memcached와 Redis 간 차이점에 대해 궁금해서 정리해보았다.

# Memcached vs Redis?

Memcached 와 redis 모두 오픈 소스 인메모리 데이터 베이스로 캐시 용도로 사용된다. 두 사이 몇가지 차이점이 존재한다. 

### Data Structure and Storage:

- Mecached: 단순 Key-Value Pair 구조이다. 간단한 String 데이터를 저장할 수 있다.
- Redis: 다양한 종류(String, list, set, sorted set, bit array)와 같은 다양한 구조 데이터 구조를 지원한다.

### Persistence

- Memcached: data persistence를 지원하지 않는다. 서버가 재식하거나 망가지면 데이터 보존이 안 이뤄진다.
- Redis: data persistence를 지원한다. 디스크에 데이터를 쓰기 때문에 서버를 재시작하거나 망가져도 데이터를 복구할 수 있다.

> Data Persistence? 
어플리케이션이나 시스템이 더 이상 작동하지 않더라도 데이터를 접근 가능하고 온전하게 남아 있도록 하는 데이터의 특성.
> 

### Execution Speed

- 일반적으로 Memcached가 Redis보다 처리 속도가 빠르다. 간단한 데이터 구조만을 지원하기 때문이다.

### Architecture:

- Memcached: 멀티 스레드 아키텍처를 지원한다. 한번에 다양한 작동을 할 수 있다. 큰 테이터셋을 스케일업하여서 사용할 수 있다.
- Redis: 단일 스레드 아키텍처리르 지원한다. 하나 프로세스가 하나 명령어를 수행할 수 있다. 여러 Redis 인스턴스를 사용하여서 이 문제를 해결할 수 있다.

### Replication

- Memcached: replication를 지원하지 않는다. scalability와 durability에 한계가 있다. 서드 파티를 이용하면 이 문제를 해결할 수 있다고 한다.
- Redis: Master-slave 구조를 지원한다. scalability와 고가용성을 확보할 수 있다.