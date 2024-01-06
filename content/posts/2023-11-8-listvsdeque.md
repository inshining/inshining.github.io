---
title: "List vs Deque in Python"
date: 2023-11-08 16:20:02 +0500
categories:
    - python 
tags:
    - python
permalink: /list-vs-deque
---

파이썬으로 코딩테스트를 준비하다보면 가끔적 list보다 deque를 사용하라는 팁을 듣곤 한다. list 보다 deque가 수행시간이 빠르다는 이유였다. 그럼에도 불구하고 기존 문제들을 풀어오면서 deque의 필요성을 느끼지 못했기에 별로 신경쓰지 않았다. 카카오 인터쉽 문제를 풀다보니 deque를 써야하는 상황이 생겨서 구체적으로 deque와 list가 어떻게 다른지 궁금했다. 

# List in Python 
- 기본적으로 array 구조를 띈다. 
- 즉, 물리적으로 연속된 Linear 자료 구조이다. 
- 그러나 array랑 다르게 파이썬 list는 같은 데이터 타입일 필요가 없다. 

# Deque in Python 
- Deque는 FIFO(First In Frist Out) 구조를 띈다. 
- 다만, 맨 앞과 맨 뒤에 요소를 추가하거나 삭제할 수 있다.
- in-built가 아니기 때문에 collections 라이브러리에서 불러와야 한다.

# 구현 측면에서 차이 

### List
Cpython 기준으로 List는 array of pointers 로 구현되어 있다. 리스트 구현체에 'size of the list'와 'allocated slots' 이 존재한다. 'size of the list'는  len() 메소드의 호출할 때 쓰이는 리스트의 요소의 개수를 의미한다. 앞서 서술했듯, 리스트는 포인터의 배열이기 때문에 배열을 반복적으로 할당하는 것을 막기 위해서 allocated slots으로 일정 크기 포인터가 들어갈 메모리를 확보한다. 즉, allocated slots은 size of the list 보다 크거나 같다. 
![list_cpython](../assets/images/list_cpython.png)

요소를 추가할 때, 2의 제곱으로 배열의 크기를 resize한다. 

> Though [`list`](https://docs.python.org/3/library/stdtypes.html#list "list") objects support similar operations, they are optimized for fast fixed-length operations and incur O(n) memory movement costs for `pop(0)` and `insert(0, v)` operations which change both the size and position of the underlying data representation.

공식 문서에서 list 맨 앞에서 삽입과 제거 과정에서 O(n)이 일어날 것으로 지적하였다. list가 array of pointers로 구현되어 있기 때문에 맨 앞 요소를 제거하거나 추가할 경우 새로운 배열로 옮겨서 담아야 하기 때문에 O(n)이 발생한다. 이 지점에서 deque와 차이점이 존재한다. 
### Deque 
 Deque는 Double-ended Queue 를 일컫는다. 즉, 양 끝 단에서 추가와 제거가 가능하다는 뜻이다. Deque는 구조적으로 Double Linked List 이다. 이러한 자료구조의 각 노드는 앞 뒤 노드의 주소를 보유하고 있다. 물리적으로 메모리가 일련로 존재하지 않기 때문에 메모리 재할당할 필요가 없다. 
 ![double_linked_list](../assets/images/double_linked_list.png)
 그림으로 보면 위와 같다. head에서 appendleft(), popleft() 가 수행되고, tail에서 append()와 appendleft()과 수행된다. 


### 결론
 테이블로 두 자료구조의 차이를 나타내보면 다음과 같다.

Append Function 

|   |   |   |
|---|---|---|
|Position|List|Deque|
|Beginning|O(n)|O(1)|
|Middle|O(n)|O(n)|
|End|O(1)|O(1)|


Pop Function 

|   |   |   |
|---|---|---|
|Position|List|Deque|
|Beginning|O(n)|O(1)|
|Middle|O(n)|O(n)|
|End|O(1)|O(1)|



# 문제 풀이에 적용하기 
문제 [출처](https://school.programmers.co.kr/learn/courses/30/lessons/118667)
특정 테스트케이스에서 시간초과가 발생했다. 

```
while i < N:
        ~~
        if target_v > q1_sum:
            v = list1.pop(0)
            list1_sum += v
            list2.append(v)
	    ~~~
```

`list1.pop(0)` 이 O(n)이기 때문에 최악의 경우 위 코드에서 O(N ^ 2)이 발생한다. 

deque를 적용해보면 다음과 같다. 
```
while i < N :
        ~~
        if target_v > q1_sum:
            v = q2.popleft()
            q1_sum += v
            q1.append(v)
        ~~
```
deque에서 `popleft()` 는 O(1)을 보장하기 때문에 최악의 경우에도 O(N)이 되었다. 


# List vs Deque 선택하기 

상황에 맞는 자료구조를 선택할 필요가 있다. 위 사례처럼 맨 앞과 맨 뒤 요소만을 접근해도 괜찮으면 deque 사용을 고려할 만하다. 각 요소에 접근하기 위해서 index를 사용하거나 Slicing 을 사용해야 하는 경우에 List를 사용하는 것이 더 현명하다. 

### 출처 
- https://www.laurentluce.com/posts/python-list-implementation/
- https://dev.to/v_it_aly/python-deque-vs-listwh-25i9
- https://stackoverflow.com/questions/6256983/how-are-deques-in-python-implemented-and-when-are-they-worse-than-lists
- https://www.askpython.com/python/list/list-vs-deque-comparison