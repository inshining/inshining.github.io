---
title: '직접 HashMap vs HashTable vs ConcurrentHashMap의 비교하기'
date: 2024-11-24 16:20:02 +0500
categories:
    - java
tags:
    - java
    - map
permalink: /java-map-compare
---

자바로 알고리즘을 공부를 하다보면 가장 자주 사용하게 되는 패키지 중 하나는 `import java.util`이다. 이 패키지를 사용하며 `List`, `Map`, `Set`, `Queue` 를 자주 사용하게 된다. 자주 사용하는 클래스에 대해서 알아보고 싶어서 이러저런 자료를 찾다보니 다음과 같은 다이어그램을 발견했다.

[java-collections-framework-diagram](/images/java-map/collections framework overview.png)
출처: https://www.codejava.net/java-core/collections/overview-of-java-collections-framework-api-uml-diagram

`java.util` 패키지를 통으로 임포트해오면서 느끼지 못했는데. `Map`은 다른 자료구조와 다르게 `Collection` 인터페이스를 공유하지 않는다. `Map` 은 키와 밸류라는 구조로 되어 있기 때문에 다른 단일한 값을을 다수 저장하는 `collection` 인터페이스와 차이가 있는 것이다.
좀 더 `Map`에 대해서 자세히 알고 싶던 차에 `HashMap vs HashTable vs ConcurrentHashMap의 차이`라는 면접 질문이 있다는 것을 알게 되었다. 셋의 차이를 직접 코드로 구현해보며 차이를 비교해보려고 한다. ~~솔직히 무수히 많은 기술블로그에서 이론적으로 정리했을 것이다.. 그리고 챗지피티가 더 정리 잘한다...~~

그래도 간단하게 셋 자료 구조의 차이를 알아보자

### **비교 요약**

| 특징               | HashMap         | HashTable     | ConcurrentHashMap        |
| ------------------ | --------------- | ------------- | ------------------------ |
| **스레드 안전성**  | ❌ (비동기화)   | ✅ (동기화됨) | ✅ (부분 동기화)         |
| **성능**           | 빠름            | 느림          | 멀티스레드 환경에서 우수 |
| **Null 허용 여부** | 키와 값 허용    | 허용 안 됨    | 허용 안 됨               |
| **도입 시기**      | 자바 1.2        | 자바 1.0      | 자바 1.5                 |
| **추천 용도**      | 싱글스레드 환경 | 레거시 코드   | 멀티스레드 환경          |

# 테스트 설계

-   **측정 대상 작업**:
    -   데이터 삽입 (`put`)
    -   데이터 검색 (`get`)
    -   데이터 삭제 (`remove`)
-   **테스트 데이터**:
    -   테스트 데이터는 일정한 크기의 임의의 키-값 쌍(예: `Integer`와 `String`)으로 생성합니다.
-   **스레드 환경**:
    -   단일 스레드에서 성능 측정.
    -   멀티스레드 환경에서 동시 작업 성능 측정.
-   **데이터 크기**:
    -   1,000,000 개.

# 코드

```java
public class MapPerformanceTest {

    private static final int DATA_SIZE = 1_000_000; // 테스트 데이터 크기
    private static final int THREAD_COUNT = 10; // 멀티스레드 테스트용 스레드 수

    static class Result{
        long insertTime, retTime, delTime;

        public Result(long insertTime, long retTime, long delTime) {
            this.insertTime = insertTime;
            this.retTime = retTime;
            this.delTime = delTime;
        }
    }


    // 단일 스레드 성능 측정
    public static Result testMapPerformance(Map<Integer, String> map, String mapName) {
        long startTime, endTime;

        startTime = System.nanoTime();
        for (int i = 0; i < DATA_SIZE; i++) {
            map.put(i, "Value" + i);
        }
        endTime = System.nanoTime();
        long insertTime = (endTime - startTime) / 1_000_000;

        // 데이터 검색 테스트
        startTime = System.nanoTime();
        for (int i = 0; i < DATA_SIZE; i++) {
            map.get(i);
        }
        endTime = System.nanoTime();
        long retTime = (endTime - startTime) / 1_000_000;

        // 데이터 삭제 테스트
        startTime = System.nanoTime();
        for (int i = 0; i < DATA_SIZE; i++) {
            map.remove(i);
        }
        endTime = System.nanoTime();
        long delTime = (endTime - startTime) / 1_000_000;
        return new Result(insertTime, retTime, delTime);
    }

    // 멀티스레드 성능 측정
    public static Result testConcurrentPerformance(Map<Integer, String> map, String mapName) {
        ExecutorService executor = Executors.newFixedThreadPool(THREAD_COUNT); // 새 ExecutorService 생성
        long startTime, endTime, insertTime,  retTime, delTime ;
        insertTime = retTime = delTime = 0;

        try {
            // 데이터 삽입 테스트
            startTime = System.nanoTime();
            for (int i = 0; i < THREAD_COUNT; i++) {
                final int threadId = i;
                executor.execute(() -> {
                    int start = (DATA_SIZE / THREAD_COUNT) * threadId;
                    int end = start + (DATA_SIZE / THREAD_COUNT);
                    for (int j = start; j < end; j++) {
                        map.put(j, "Value" + j);
                    }
                });
            }
            executor.shutdown(); // 모든 작업 제출 완료 후 shutdown 호출
            executor.awaitTermination(1, TimeUnit.MINUTES); // 모든 작업 완료 대기
            endTime = System.nanoTime();
            insertTime = (endTime - startTime) / 1_000_000;

            // 검색 테스트
            executor = Executors.newFixedThreadPool(THREAD_COUNT); // 새 ExecutorService 생성
            startTime = System.nanoTime();
            for (int i = 0; i < THREAD_COUNT; i++) {
                final int threadId = i;
                executor.execute(() -> {
                    int start = (DATA_SIZE / THREAD_COUNT) * threadId;
                    int end = start + (DATA_SIZE / THREAD_COUNT);
                    for (int j = start; j < end; j++) {
                        map.get(j);
                    }
                });
            }
            executor.shutdown(); // 모든 작업 제출 완료 후 shutdown 호출
            executor.awaitTermination(1, TimeUnit.MINUTES); // 모든 작업 완료 대기
            endTime = System.nanoTime();
            retTime = (endTime - startTime) / 1_000_000;

            // 삭제 테스트
            executor = Executors.newFixedThreadPool(THREAD_COUNT); // 새 ExecutorService 생성
            startTime = System.nanoTime();
            for (int i = 0; i < THREAD_COUNT; i++) {
                final int threadId = i;
                executor.execute(() -> {
                    int start = (DATA_SIZE / THREAD_COUNT) * threadId;
                    int end = start + (DATA_SIZE / THREAD_COUNT);
                    for (int j = start; j < end; j++) {
                        map.remove(j);
                    }
                });
            }
            executor.shutdown(); // 모든 작업 제출 완료 후 shutdown 호출
            executor.awaitTermination(1, TimeUnit.MINUTES); // 모든 작업 완료 대기
            endTime = System.nanoTime();
            delTime = (endTime - startTime) / 1_000_000;

        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            if (!executor.isTerminated()) {
                executor.shutdownNow(); // 강제 종료
            }
        }

        return new Result(insertTime, retTime, delTime);
    }
}
```

# 결과

## Single-thread Performance

|          | HashMap | HashTable | ConcurrentHashMap |
| -------- | ------- | --------- | ----------------- |
| **삽입** | 77ms    | 74ms      | 85ms              |
| **탐색** | 14ms    | 21ms      | 15ms              |
| **삭제** | 13ms    | 10ms      | 25ms              |
|          |         |           |                   |

## Multi-thread Performance

|          | HashMap | HashTable | ConcurrentHashMap |
| -------- | ------- | --------- | ----------------- |
| **삽입** | 140ms   | 179ms     | 114ms             |
| **탐색** | 22ms    | 58ms      | 30ms              |
| **삭제** | 34ms    | 64ms      | 21ms              |

# 결과분석

전반적으로 이론적 특성에 따라서 예상된 결과가 나왔다.

## 싱글 스레드 환경

싱글 스레드 환경에서 `HahsMap`이 두 자료구조에 비해서 빠른 것으로 확인되었다. 하지만 일부 메소드에 따라서 큰 차이가 나지 않거나 오히려 느린 결과가 나왔다.
그 중 하나로 삽입 메소드 수행 시간만 보면 HashMap이 HashTable보다 미세하게 느렸다. 이는 2가지 원인으로 추정되는데. 일단 데이터 크기가 크지 않았기 때문에 일반적인 차이가 나지 않았다고 추정되었다. 또한, 계속된 삽입 과정에서 리사이즈 과정을 거치게 되는데. 이 과정에서 HashTable이 HashMap에 비해서 최적화가 잘되어 더 빠르게 실행된다는 추정을 해볼 수 있었다.
탐색 과정에서 `HashMap`과 `ConcurrentHashMap` 큰 차이가 나지 않았는데. `Read` 행위이기 때문에 삽입과 삭제에 비해서 더 빠르게 동작하는 것이 아닌가라고 생각해볼 수 있었다.

## 멀티 스레드 환경

멀티 스레드 환경에서 `HashTable`의 한계가 명확히 보였다. 싱글 스레드 환경에서 `HashTable`과 `HashMap`간 차이가 얼마 나지 않아서 `HashTable` 쓸만 할지도 라고 생각했다면 멀티 스레드 환경에서 나머지 두 자료구조와 큰 차이를 보이며 최근 멀티 스레드 환경에서 `ConcurrentHashMap` 권장한다는 내용을 이해할 수 있었다.
싱글 스레드와 마찬가지로 탐색 메소드에서 `HashMap`와 `ConcurrentHashMap` 간 큰 차이를 보이지 않았다.
멀티 스레드 환경에서 `ConcurrentHashMap` 의 월등한 성능을 보이며 멀티 스레드 환경에서 무조건 `ConcurrentHashMap`을 써야 함을 알 수 있었다.

# 테스트 한계

### 테스트 사이즈 크기

테스트 케이스가 크지 않아서 일부 메소드에서 확실히 성능 차이를 보이지 않았다. 테스트 케이스를 키우면 더욱 분명한 차이를 보일 것 같다.

# 테스트 방식에 대한 고민

처음 테스트할 때, `HashMap`, `HashTable`, `ConcurrentHashMap` 순으로 순차적으로 실행했더니 다음과 같은 결과가 나왔다.

```
=== Single-thread Performance ===
HashMap - Insertion: 201 ms
HashMap - Retrieval: 29 ms
HashMap - Deletion: 20 ms
Hashtable - Insertion: 119 ms
Hashtable - Retrieval: 27 ms
Hashtable - Deletion: 55 ms
ConcurrentHashMap - Insertion: 138 ms
ConcurrentHashMap - Retrieval: 46 ms
ConcurrentHashMap - Deletion: 58 ms

=== Multi-thread Performance ===
HashMap - Concurrent Insertion: 290 ms
Hashtable - Concurrent Insertion: 176 ms
ConcurrentHashMap - Concurrent Insertion: 154 ms
```

이론에 완전히 반대되는 결과값이라서 혼란스러웠다. 자료구조 실행 순번을 바꾼 결과, 가장 먼저 실행하는 자료구조가 가장 오랜 시간이 걸리는 것으로 확인되었다. 코드를 살펴보니 각 자료구조가 동일한 메소드에 파라미터를 넘겨 테스트하였는데. 이 과정에서 JVM의 초기 설정 시간에 의해서 가장 처음 실행되는 자료구조가 오랜 시간이 걸리는 것으로 추측할 수 있었다. 즉, 자료구조 특징보다 JVM 환경에 의해서 테스트 결과가 결정되었다. 따라서 각 자료구조 분리해서 실행하는 방식으로 테스트 방법을 변경하였다.
