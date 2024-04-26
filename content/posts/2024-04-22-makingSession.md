---
title: "자바 서블릿을 이용해서 세션 구현해보기"
date: 2024-04-22 16:20:02 +0500
categories:
    - implementing
tags:
    - Session
    - Http
permalink: /implement-http-session
---


이 글은 "[자바 웹 프로그래밍 NextStep](https://m.yes24.com/Goods/Detail/31869154)" 을 참고하였습니다. 

# 세션이란? 
### 세션의 필요성 
HTTP는 무상태 프로토콜이다. 클라이언트와 서버 사이 상태를 유지할 수 없다. 클라이언트와 서버 사이 통신 맥락을 모두 공유할 수 없다는 뜻이다. 만약 클라이언트가 로그인을 성공하더라도 서버는 클라이언트가 로그인을 성공한 것인지 알 수 없다. 이 때문에 유저별 권한이 설정된 웹 사이트라면 매번 인증을 통해서 클라이언트가 원하는 동작을 수행해야 한다. 이러한 비효율을 줄이고자 세션과 같은 서버가 클라이언트를 식별할 장치가 필요하다. 

## 세션 기본 구조 
![세션기본구조](https://miro.medium.com/v2/resize:fit:1400/0*bejurH3uZoTrZita.png)

### 쿠키를 사용하는 이유?
쿠키에 세션을 담아서 사용한다. 서버에서 "Set-Cookie"헤더를 응답으로 내려주면, 클라이언트는 다음에 동일한 서버에 요청을 할 때, "Cookie"헤더에 서버가 내려준 값을 그대로 담아서 요청한다. 

### 세션 ID를 만든 이유?
그러나 쿠키는 보안에 취약하다. 당장 브라우저 개발자 도구에서 쿠키의 값을 알 수 있다. 쿠키에 사용자의 비밀번호와 같은 개인 정보 데이터를 전달하면 안된다. 세션에서 저장하고자 하는 데이터는 서버에서 저장한다. 각 클라이언트를 식별할 수 있는 ID는 서버에서 임의로 만들어서 쿠키에 담아서 클라이언트에게 보내주는 방식이다. 

# 세션 설계 
![Session Architecture](https://terasolunaorg.github.io/guideline/5.3.0.RELEASE/en/_images/session-management_overview_cooperation.png)

세션을 만들기 위해서는 세션 클래스와 세션을 관리하는 클래스, 세션 필요로 하는 로직이 필요하다. 
세션을 만들고, 세션에 데이터를 저장하고, 세션의 데이터를 이용하는 과정은 각각 (2), (3), (6)에 해당한다. 이를 구현할 것이다. 
# 세션 구현하기

## 1. 쿠키에 세션 추가하기 
```java
....
// 쿠키 만들어주기
HttpCookie httpCookie = request.createHttpCookie();  

// cookie에 세션있는지 검사. 없으면 새로 생성    
if (httpCookie.isSessionIdEmpty()){
	// 세션 만들기. 세션 ID를 생성한다. 
    HttpSession session = HttpSession.createSession();  

	// 전체 세션을 맵핑할 때 사용할 클래스인 HttpSessions에 신규 세션을 추가한다. 
    HttpSessions.addSession(session); 

	// 쿠키에 세션 아이디를 추가한다. 
    httpCookie.setSessionId(session.getId());  
    // Set-Cookie를 헤더를 추가한다. 
    response.addHeader("Set-Cookie", httpCookie.toString());  
}
....
```

## 2. 세션 클래스 정의하기 
```java
public class HttpSession {
	// session 아이디 
    private String id; 
    // 세션에 저장할 데이터를 Map 타입으로 저장한다.  
    private Map<String, Object> attributes = new HashMap<>();  
	
    public static HttpSession createSession(){  
    // UUID 클래스를 사용해서 ID를 생성한다. 
    // UUID는 다른 ID와 충돌할 가능성이 낮다. 
    // UUID는 개인의 식별 정보를 추론하기 어렵다. 
    // 이러한 이유로 UUID를 사용했다. 
        UUID uuid = UUID.randomUUID();  
        return new HttpSession(uuid);  
    }  
  
    public HttpSession(UUID uuid){  
        this.id = uuid.toString();  
    }  
    public String getId(){  
        return this.id;  
    };  
  
    public void setAttribute(String key, Object value){  
        attributes.put(key, value);  
    };  
  
    public Object getAttribute(String key){  
        return attributes.get(key);  
    };  
  
    public void removeAttribute(String key){  
        attributes.remove(key);  
    };  
  
    public void invalidate(){  
        attributes.clear();  
    };  
}

```

## 3. 세션 매니저 클래스 정의하기 
세션를 담고 있는 클래스 
```java
public class HttpSessions {  
	// Key에는 SessionID를 저장하고, Value는 HttpSession을 저장한다. 
    public static Map<String, HttpSession> sessions = new HashMap<>();  
  
    public static void addSession(HttpSession session) {  
        sessions.put(session.getId(), session);  
    }  

	// ID를 통해서 세션을 구할 수 있는 메소드
    public static HttpSession getSession(String sessionId) {  
        HttpSession httpSession =  sessions.get(sessionId);  
        if (httpSession == null){  
            httpSession = HttpSession.createSession();  
            addSession(httpSession);  
        }  
        return httpSession;  
    }  
    public static void deleteSession(String sessionId) {  
        sessions.remove(sessionId);  
    }  
}
```

## 4. 세션에 데이터 담기 
### 로그인시 user 데이터 담기 
```java
...
// 로그인 성공시 아래 로직 수행
// 세션ID에 해당하는 세션을 불러오기
HttpSession httpSession = request.getHttpSession();  
// 세션에 user 정보 저장하기 
httpSession.setAttribute("user", user);
```
### 세션으로 접근 권한 설정
```java
.....
// 로그인한 회원만 접근할 수 있도록 로그인 하지 않은 유저는 loging 페이지로 리다이렉트한다
if (!isLogin(request.getHttpSession())) {  
    response.sendRedirect("/user/login.html");  
    return;
    }
    
.....
// 로그인 후에 user 정보를 세션을 넣어서 갱신하기 때문에 
// 로그인하지 않았다면 user 정보가 없다. 
private boolean isLogin(HttpSession session) {  
    Object user = session.getAttribute("user");  
    if (user == null) {  
        return false;  
    }  
    return true;  
}
.....
```

# 추후 변경될 수 있는 포인트

## 쿠키 보안을 강화하기 위한 장치들 
쿠키 보안을 위해서 쿠키 속성을 더 추가할 필요가 있다. 아래 속성을 참고하자. 

- domain: 브라우저가 어느 사이트에 이 쿠키를 보낼지 정해준다. domain 으로 설정한 사이트 뿐만 아니라 서브 도메인 사이트 역시 포함된다. 
- path: 쿠키가 유효한 경로를 설정한다. 예를 들어서, Path=/images 로 설정하면 images 경로 아래 페이지만 쿠키를 보낼 수 있다. 
- max-age: 쿠키 유효 기간을 설정하는 옵션이다. 
- secure : HTTPS 프로토콜에서만 쿠키를 보낼 수 있도록 하는 옵션이다. 

## 세션 정보를 외부 데이터베이스로 분리하기
위 코드는 HttpSessions 클래스에 세션 정보를 담고 있다. 즉, 로컬 메모리에 세션 정보를 올려서 저장하고 있다. 프로세스를 중지하면 모든 데이터는 사라진다. 안정적으로 데이터를 저장해야 한다면 세션을 외부 데이터베이스로 분리해야 한다. 만약 응답을 빠르게 해야 하는 경우 Redis, Memcache와 같은 인메모리를 사용하면 좋을 것이다. 


# 고민 포인트
직접 구현한 고민 포인트를 모아보았다. 책에서 나온 모범코드와 다르다. 

#### HttpSessions 의 Mapping 정보 저장하기 
```java
public class HttpSessions {  
    public static Map<String, HttpSession> sessions = new HashMap<>();
    ......
}
```
HttpSessions 클래스를 Mapping 자료 구조를 어떻게 처리할 지 고민이 많았다. 프로젝트의 어느 파일에서나 접근할 수 있어야 한다는 조건을 고려해야 했다. 만약 이 클래스를 인스턴스로 선언한다면 이 인스턴스를 파라미터로 넘겨야 한다는 단점이 생긴다. 따라서 클래스 단위로 데이터를 접근해야 했다. 
**static** 를 이용해서 어디서든 접근 가능하도록 한다. static 은 스태틱 메모리 영역에 저장된다. 이 메모리 영역은 프로그램 시작부터 종료까지 메모리에 남아있기 때문에 언제, 어디서나 이 메모리에 접근할 수 있다. 

#### 하나 클래스에서 책임을 다하기 위한 코드
```java
// 쿠키에 키 값으로 들어간 SessionID final로 설정한다. 
private final String SESSIONID = "JSESSIONID";
.....
public void setSessionId(String value){  
    cookies.put(SESSIONID, value);  
};
....
```
모범 코드에서 "JSESSIONID"을 매번 하드 코딩하였다. 이 값을 변경해야 한다면 일일이 변경해야 하는 불편함이 존재한다. final로 정의하여 하드코딩을 예방했다. 
모범 코드과 다르게 쿠키 클래스에서 세션ID를 저장하는 메소드를 추가하였다. 쿠키과 관련된 로직은 쿠키 클래스에서 맡아야 한다고 보았기 때문이다. 


#### httpCookie 클래스내에 아래 메소드 정의하기 
```java
public String toString(){  
    StringBuilder sb = new StringBuilder();  
    for (String key : cookies.keySet()){  
        sb.append(key + "=" + cookies.get(key) + "; ");  
    }  
    return sb.toString();  
};
```
모범코드에서 쿠키 값을 헤더에 바로 작성하는 코드로 구성되어 있다. 그러나 헤더에 쿠키를 추가하는 로직이 이후에 추가하면 기존 쿠키값이 없어질 여지가 있다. toString 메소드를 정의하여서 새로 쿠키를 추가하더라도 빠지지 않도록 할 수 있다. 

#### HttpSession 생성 메소드
- 사용자가 HttpSession 인스턴스 생성에 있어서 관심을 두지 않고도 만들 수 있도록 한다. 
- 만일 UUID가 아닌 다른 값을 넣을려고 하면  의도치 않은 결과를 낳을 수 있다. 
- 또한, 추후에 ID를 정의하는 방식이 변경될 경우 아래 코드만 수정하면 된다. 
```java
public static HttpSession createSession(){  
    UUID uuid = UUID.randomUUID();  
    return new HttpSession(uuid);  
}
```


# 참조 
- https://terasolunaorg.github.io/guideline/5.3.0.RELEASE/en/ArchitectureInDetail/WebApplicationDetail/SessionManagement.html
- https://medium.com/@alysachan830/cookie-and-session-i-the-basic-concept-fea3d52bc8d3
- https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies

