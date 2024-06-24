---
title: "ORM은 무엇인가요? (+장고 코드로 알아보기)"
date: 2022-10-05 16:20:02 +05:00
categories:
    - oop
tags:
    - orm
    - django
    - n+1
permalink: /what-orm
---

# ORM (Object Relational Mapping) ?

관계형 데이터베이스(RDBMS)를 객체형으로 매핑해주는 기술이다. 

## 데이터는 어디에 보관되는가?

결제 정보와 같이 중요한 데이터를 메모리에 저장하게 된다고 가정해보자. 서버 컴퓨터의 전원이 나가게 되면 데이터가 날아간다. 이러한 데이터를 영원히 저장하기 위해서  파일 시스템이나, 관계형 데이터 베이스 혹은 객체 데이터베이스를 이용한다. 

## ORM을 사용하는 이유

 관계형 데이터베이스과 객체 지향 프로그래밍 사이 형태 차이가 존재한다. 데이터를 마치 객체와 같이 사용하기 위해서 관계형 데이터베이스와 클래스 사이를 연결해줘야하는데. 이 역할을 ORM이 한다. 

## ORM 장점

- **객체 지향적인 코드로 비즈니스 로직에 집중할 수 있다.** sql문을 직접 작성하지 않고 객체로 접근하기 때문에 비지니스 로직에 좀 더 집중해서 개발할 수 있다.
- **재사용하거나 유지보수하기 편리하다**. orm은 주로 MVC 에서 Model 를 담당하고 있기 때문에 디자인 패턴을 형성하기 쉽다.
- **DBMS에 대한 종속성을 낮출 수 있다.**  ORM을 사용하면 DB에 종속되지 않기 때문에 다른 종류 RDMBS를 사용하더라도 얼마든지 교체가능하다.

## ORM 단점

### ORM이 오히려 속도 저하로 이어질 수 있다.

복잡한 비지니스 로직을 orm으로 단순히 해결하려다가 sql문 보다 더 느려질 수 있다. 예를 들어, N+1 문제가 발 생할 수 있다. 

### N+1 문제

```python
# Django 모델
class Author(models.Model):

    first_name = models.CharField(max_length=200)
    last_name = models.CharField(max_length=200)
    
        
class Book(models.Model):
    author = models.OneToOneField(Author, on_delete=models.CASCADE)
    title = models.CharField(max_length=50)
```

위와 같은 모델이라고 가정해보자 

```python
park_authors = Author.objects.filter(first_name='park')

for author in park_authors:
	title = author.book.title
	print(f"The Book's title: {title}")
```

위와 같은 코드를 짠다고 했다고 했을 때, 첫 줄에서 한 번 쿼리를 날린 후 n명에 대한 쿼리를 날린다. 이후 for을 순회하면서  n 회 쿼리를 보내는데. 이를 N+1라고 일컫는다. 

ORM을 사용한다면 이러한 N+1 문제가 생긴다. 만약 sql문을 작성했다면 join을 통해 쿼리의 수를 줄일 수 있을 것이다.  

## OOP에서만 ORM을 사용할까?

ORM은 관계형 데이터베이스를 객체(클래스) 형태로 사용할 수 있도록 하는 도구이다. 그렇다면 OOP 이외에 다른 영역에서 ORM을 사용하지 못하느냐에 대해 반문할 수 있다. 결론부터 말하면 ORM도 database abstraction layer 중 하나로 OOP가 아닌 함수형 프로그래밍에서도 orm을 안 쓸 뿐 database abstraction layer 중 하나는 사용한다. 대표적인 예로 [SQLAlchemy](http://www.sqlalchemy.org/)의 경우, orm을 지원하는 분야와 db api 코어 영역이 이 두 가지로 구성되어있다. 이 처럼 orm 을 쓰지 않더라도 db 에 쉽게 접근할 수 있는 추상화된 레이어 툴을 사용해서 프로그래머가 신경 쓰지 않아도 되는 스키마 내용은 숨길 수 있다.