---
title: "Python 멀티 스레드"
date: 2022-09-27 16:20:02 +05:00
categories:
    - python
tags:
    - python
    - multithread
    - gil
permalink: /python-multithread
---

## 멀티 스레드?

멀티 스레드는 하나 프로세스 내에서 복수 스레드가 동시에 작업을 수행하는 것이다. 

멀티 스레드 환경에서 스레드끼리 메모리를 공유하지만 각 스레드 마다 registers, stack, pc 모두 독립적으로 존재한다.  

## 파이썬으로 구현해보는 멀티 스레딩

기본적으로 파이썬에서 하나 프로세스에 하나 스레드 작업을 수행하지만 threading을 이용하면 스레드 작업을 해 볼 수 있다. 
```python
from threading import Thread 
import time

def job(id, start, end, result):
    print(f"job Id : {id}")
    total = 0
    for i in range(start, end):
        total += i
    result.append(total)
    return

if __name__ == '__main__':
    START, END = 0, 100000000
    result = list()
    thread1 = Thread(target=job, args=(1, START, int(END /4), result))
    thread2 = Thread(target=job, args=(2, int(END/4),int(END/2), result))
    thread3 = Thread(target=job, args=(3, int(END/2),int(END * 3/4), result))
    thread4 = Thread(target=job, args=(4, int(END * 3/4), END, result))

    thread1.start()
    start_time = time.process_time()
    thread2.start()
    thread3.start()
    thread4.start()
    thread1.join()
    thread2.join()
    thread3.join()
    thread4.join()
    print(f"결과 값 : {sum(result)}")
    end_time = time.process_time()
    print(f"프로세스 실행 시간 : {end_time - start_time}")
```

1부터 100,000,000까지 자연수를 더하는 간단한 함수를 멀티 스레드 환경에서 수행하려고 했다. 스레드 갯수에 따른 수행 시간을 비교하기 위해서 스레드 갯수를 1, 2, 4개로 구분하여 테스트를 실행하였다. 

### 최종 결과

스레드 1개인 경우![](https://velog.velcdn.com/images/inshining/post/170f8725-4646-4e7e-afcf-8dbc657ae8ab/image.png)


스레드 2개인 경우 
![](https://velog.velcdn.com/images/inshining/post/90d798ff-9317-495a-b0b5-cc004dfb0200/image.png)

스레드 4개인 경우
![](https://velog.velcdn.com/images/inshining/post/6e74af52-4403-44a1-b11b-9686cd784fe1/image.png)

 

상식적으로 스레드 갯수가 많을 수록 수행 시간이 짧아질 것으로 기대하였지만 멀티 스레드 환경에서 수행 시간의 큰 변화가 나타나지 않았다. 오히려 스레드 갯수가 많을 수록 수행 시간이 길어진 것으로 나타났다. 

## 왜 파이썬 멀티 스레딩은 느릴까?

스레드의 갯수가 많아 질 수록 Context Switching의 비용만 커져서 수행 시간이 길어졌다. 근본적으로 파이썬 멀티 스레딩 환경에서 Global Interpreter Lock(이하 GIL) 때문에 효율적으로 멀티 스레딩이 일어나지 않을 수 있다. 

### GIL

 파이썬 인터프리터는 하나 시점에서 하나 스레드만 사용하도록 강요한다. 보통 다른 언어가 스레드를 병렬적으로 처리하는 것과 다른 작동 방식이다. 이는 파이썬 인터프리터의 동작 방식과 연관되어 있다. 

### 파이썬 인터프리터의 가비지 컬렉터

 파이썬에서 모든 것이 객체로 존재한다. 파이썬은 객체를 참조하는 방식으로 동작하는데. 파이썬 객체를 참조 횟수를 기반으로 가비지 컬렉터가 동작한다. 해당 객체에 대한 참조 횟수가 0이 되면 메모리에서 해당 객체가 삭제된다. 그렇기 때문에 여러 스레드에서 동시에 특정 객체에 접근하면 가비지 컬렉터가 정상적으로 작동하기 어렵다. 이를 방지하기 위해서 하나 스레드가 하나 객체에 접근할 수 있는 권한을 주는 방식은 뮤텍스(Mutex)를 사용할 수 있다. 

### GIL를 사용할 수 밖에 없는 이유

 모든 파이썬 객체에 뮤텍스를 사용하면 그만큼 많은 리소스를 사용해야 하기 때문에 비효율적일 수 있다. 그래서 파이썬느 GIL를 사용하여서 전역적으로 객체에 대한 접근 권한을 통제하는 것이다. 

## 파이썬 멀티 스레딩을 효율적으로 사용할 수 있는 환경

 GIL은 순전히 cpu 사용에 한해서 엄격하게 동작하기 때문에 하나 스레드가 i/o와 같은 활동으로 cpu를 사용하기 않게 된다면 다른 스레드가 cpu를 사용할 수 있다. 다시 말해, i/o와 같이 외부 연산이 많은 작업에 경우 멀티 스레드가 효율적으로 동작할 수 있다.