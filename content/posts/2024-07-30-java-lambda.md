---
title: "[모던자바인액션] 자바 람다"
date: 2024-07-30 16:20:02 +0500
categories:
    - java
tags:
    - java
    - lambda
    - java8
    - modernJavaInAction
permalink: /java-lambda
---

`Modern Java In Action` 에서 자바 8버전 이후 변경 사항을 갼락하게 소개하는 챕터로 시작한다. 예시를 들어가며  lambda를 소개한다. 이번 글에서 람다가 사용될 수 있는 예제를 통해서 람다 사용 방법에 대해서 알아보자. 

# 변화하는 요구사항을 대처하기 위한 동적 파라미터화

## 문제 상황
`동적 파라미터화` 는 변화하는 요구사항을 유연하게 대처하기 위해 필요하다. 특정 조건에 해당하는 객체를 뽑아내야 한다. 예를 들어, 빨간색 사과만 뽑아내는 코드를 작성해보자
```java
public static List<Apple> filterGreenApples(List<Apple> inventory){
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory){
		if (GREEN.equals(apple.getColor())){
			result.add(apple);
		}
	}
	return result;
}
```
위와 같은 코드를 짤 수 있다. 하지만 다음날 녹색이 아닌 빨간색 사과만 뽑는 조건으로 변경되었다. `if` 문을 수정해야 한다. 혹은 무게가 150g 을 넘는 사과를 뽑는 조건은 어떨까? 이상이 아닌 이하면 또 어떨까? 이런 조건이 `AND` 혹은 `OR`로 겹치는 복잡한 조건을 요구사항으로 지정되면 어떻게 해야 할까? 이런 식으로 잦은 요구사항 변경으로 조건이 달라져야 한다면 조건문 자체를 드러낼 필요성이 있다. 

## 동적파라미터화: 인터페이스를 활용한 방법 (프레디케이트)
자바 인터페이스를 사용하여 조건문 부분을 변경할 수 있다. 
```java
public interface ApplePredicate {
	boolean test(Apple apple);
}

~~~

public class AppleHeavyPredicate implements ApplePredicate {
	boolean test(Apple apple);
}

~~~

public static List<Apple> filterGreenApples(List<Apple> inventory, ApplePredicate p ){
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory){
		if (p.test(apple)){
			result.add(apple);
		}
	}
	return result;
}

```
인터페이스로 메소드를 정의하고 상속을 통해서 다양한 조건을 클래스로 표현할 수 있다. 이전 코드보다 동적으로 처리할 수 있다. 그러나 이런 코드는 다양한 조건을 반복해서 사용할 때 효과적인 것이다. 인터페이스를 정의하고 그에 따라서 상속받는 코드를 작성하면 많은 코드를 작성해야 한다. 좀 더 효과적으로 코드를 작성할 수 있는 방법이 있을까? 

## 동적파라미터화: 익명 클래스 사용
미리 정의한 인터페이스를 활용해서 익명 클래스를 만들어 낼 수 있다. 필요할 때 객체를 구현한다는 점에서 이전 방식보다 수고로움이 덜하다 
```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
	public boolean test(Apple apple){
		return RED.equals(apple.getColor());
	}
})
```
그러나 익명 클래스도 장황하긴 마찬가지다. 코드가 장황하여서 가독성이 떨어진다. 코드 간결성을 위해서 람다를 사용해야 한다. 

## 동적파라미터화: 람다 사용
```java
List<Apple> redApples = filterApples(inventory, (Apple apple) -> 
									RED.equals(apple.getColor()))
```
이전보다 간결하다. 

# 람다를 사용하는 예제
## Comparator 사용하기
```java
inventory.sort(newComparator<Apple>() {
	public int compare(Apple a1, Apple a2){
	return a1.getWeight().compareTo(a2.getWeight())
	}
})
-> 
inventory.sort((Apple a1, Apple a2) -> 
	a1.getWeight().compareTo(a2.getWeight())
)
```

# 람다의 특징
- 익명: 자바에서 다른 메소드와 달리 이름이 없기 때문에 익명이다. 구현해야 할 코드가 줄어든다.
- 함수: 특정 클래스에 종속된 메소드와 달리 종속되지 않은 형태인 함수이다. 하지만 메소드처럼 파라미터, 바디, 예외리스트 모두 표현할 수 있다.
- 전달: 인수로 전달하거나 변수 지정이 가능하다
- 간결성: 익명클래스보다 간결하다. 

위에 예시에서 살펴보았듯 기존 자바 문법에서 메소드 하나를 사용하기 위해서 클래스를 정의하고 메소드를 정의해야 했다. 이로 인해서 구현해야 할 코드는 많아지고 보기에도 장황해졌다. 하지만 람다의 등장으로 자바에서도 메소드가 아닌 함수 사용이 가능해지고 간결해졌다. 

# 어디에 람다 사용할까?
 람다를 사용하기 좋은 곳은 `함수형 인터페이스`이다. 
API로 정의된 함수형 인터페이스인 `Predicate, Comparator, Runnable, Callable`이 대표적인 예시이다.
`@FunctionalInterface`를 사용한다면 명시적으로 함수형 인터페이스임을 알 수 있다. 추상 메소드를 오직 하나만 정의한 인터페이스에 붙일 수 있다. 만약 이 애노테이션이 붙어있는 인터페이스가 조건에 충족하지 않는다면 컴파일 단계에서 컴파일러가 에러를 발생시킨다. 

자세한 함수형 인터페이스 API를 알고 싶다면 [여기](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)를 참고해보자

# 메소드 참조
람다 표현식을 좀 더 응용해보면 메소드 참조를 할 수 있다. 메소드 참조는 무엇일까? 다음 예시를 보면서 확인해보자
```java
inventory.sort((Apple a1, Apple a2) -> 
	a1.getWeight().compareTo(a2.getWeight())
) 
-> 
inventory.sort(comparing(Apple::getWeight));
```
람다 형태에서 메소드 참조를 활용하면 가독성을 높일 수 있다. 직접적으로 메소드를 참조하여 보다 간결한 코드를 만들어 낼 수 있다. 