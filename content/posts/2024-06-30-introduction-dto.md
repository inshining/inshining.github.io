---
title: "Controller <-> Service 간 DTO 도입"
date: 2024-06-30 16:20:02 +0500
categories:
    - java
tags:
    - java
    - spring
    - dto
permalink: /dto-introduction
---

# 문제 상황 

Controller 테스트 코드 작성시에 성공과 실패 코드 짜는 중에 
서비스 레이어에서 단순히 String 타입으로 리턴하기 때문에 비정상적인 결과를 표기할 수 없었다. 


### 테스트 코드 
```java
@Test  
public void testUpload() throws Exception{  
    MockMultipartFile file = new MockMultipartFile(  
            "file",  
            "hello.txt",  
            MediaType.TEXT_PLAIN_VALUE,  
            "Hello, World!".getBytes()  
    );  
  
    when(metaDataService.uploadFile(file, "test")).thenReturn("File uploaded successfully");  
  
    mockMvc.perform(  
            multipart("/file/upload").file(file)  
            .param("user", "test")  
            ).andExpect(status().isOk())  
            .andExpect(content().string("File uploaded successfully"));  
}
```

### 이전 Controller 코드 
```java
    if (file.isEmpty()) {  
        return ResponseEntity.badRequest().body("Please select a file to upload");  
    }  
    String body =  metaDataService.uploadFile(file, username);  
    return ResponseEntity.ok(body);  
}
```
현재 구조로는 실패해도 status code를 제대로 보내지 못한다. 이를 해결하기 위해서 Controller <-> Service 간 데이터 교환을 할 객체가 필요했다. 레이어들 간 데이터 교환 객체를 여러 용어로 부르지만 이 문제 상황에서 DTO라고 부르도록 하겠다. 

### 이전 Service 코드 
서비스 레이어에서 String으로 반환하기 때문에 서비스 함수 성공 여부 전달할 수 없다. ~~String 안에 문자를 다르게 할 수도 있겠지만 짜친다..~~ 아래 코드를 보다시피 단순 String 타입으로 리턴하면 표현의 한계가 존재한다. 
```java
public String uploadFile(MultipartFile file, String username){  
  
    try {  
        // Get the file and save it somewhere  
        byte[] bytes = file.getBytes();  
        Path path = Paths.get(UPLOAD_DIR + file.getOriginalFilename());  
        Files.write(path, bytes);  
  
        MetaData metaData = initMetaData(file, username);  
  
        // Save metadata to database  
  
    } catch (IOException e) {  
        e.printStackTrace();  
        return  "Failed to upload file: " + e.getMessage();  
    }  
    return  "File uploaded successfully";  
}
```

# 해결 방법 DTO로 도입하자
### 최종본 DTO를 넣은 코드 
```java
public FileUploadResponse uploadFile(MultipartFile file, String username){  
  
    try {  
        // Get the file and save it somewhere  
        byte[] bytes = file.getBytes();  
        Path path = Paths.get(UPLOAD_DIR + file.getOriginalFilename());  
        Files.write(path, bytes);  
  
        MetaData metaData = initMetaData(file, username);  
  
        // Save metadata to database  
  
    } catch (IOException e) {  
        e.printStackTrace();  
        return new FileUploadResponse(false, "Failed to upload file: " + e.getMessage());  
    }  
    return new FileUploadResponse(true, "File uploaded successfully");  
}
```
위 코드를 보다시피 DTO 객체에 '성공여부', '메세지' 라는 두 필드를 정의했다. try-catch 문을 통해서 실패한 경우 false로 리턴하게 하였다. 
# 느낀점
많은 책들에서 테스트코드를 많이 짜길 권했다. 가장 대표적인 이유로 테스트코드를 짜다보면 더 나은 코드를 짤 수 있다고 말했기 때문이다. 이번에 테스트 코드를 짜다보니 여러 상황에 적합한 코드를 생각해 볼 수 있었고 자연스럽게 DTO 도입으로 이어졌다. 테스트 코드의 중요성을 깨닫고 테스트 코드를 제대로 공부해보고 싶었다. 

# 추후 개선점
현재 IOException 만 에러 처러하였는데. 추후에 코드가 추가되면서 다른 예외처리가 필요할 것으로 보인다. 또한, 현재 DTO 필드는 2개인데. 다른 정보를 컨트롤러에 추가해야 할지 고민해봐야 한다. 

