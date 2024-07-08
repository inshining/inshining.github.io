---
title: "TIL: 서비스 레이어 간 분리"
date: 2024-07-02 16:20:02 +0500
categories:
    - spring
tags:
    - java
    - spring
    - service-layer
permalink: /service-layer-seperation
---

# 문제 상황
downloadFile 기능을 테스트 코드로 작성하던 중 테스트를 위한 임시 파일 생성과 다운로드 기능에서 계속된 에러가 발생했다. 임시 파일 생성하고 해당 파일을 다운로드 받아야 하는데. 생성된 파일 경로를 읽어 오지 못했기 때문이다. 테스트하기 어려운 구조로 짜여져있다는 것을 깨닫고 고칠려고 했다. 그러다보니 해당 테스트는 메타 데이터에 관한 테스트인데. 실제 파일 생성과 다운로드를 테스트 과정에 집어넣는 것이 잘못된 것이라고 깨닫고 분리하려고 했다. 요약하자면 테스트하기 어려운 구조로 변경하려다가 서비스 레이어 코드에서 분리가 필요하다고 느꼈다. 

## 이전 다운로드 기능 코드 
이전에 `getFileAsInputStream` 이름으로 파일 다운로드 메소드를 분리하였다. 그러나 구조상 메타데이터 서비스 레이어의 하위 메소드로 파일 다운로드 기능이 들어가는 것이 맞지 않는다고 판단했다. 
```java
    public FileDownloadDTO downloadFile(String filename, String username) throws IOException {
        // TODO: null 반환하는 것은 추후에 처리하도록 변경
        MetaData metaData = metadataRepository.findByOriginalFilenameAndUsername(filename, username);
        if (metaData == null) {
            return null;
        }
        InputStream inputStream = getFileAsInputStream(metaData.getId().toString());
        if (inputStream == null) {
            return null;
        }
        return new FileDownloadDTO(inputStream, metaData.getOriginalFilename(), MediaType.parseMediaType(metaData.getContentType()), metaData.getSize());
    }

    private InputStream getFileAsInputStream(String storagePath) throws IOException{
        try {
            return Files.newInputStream(Paths.get(UPLOAD_DIR, storagePath));
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }
```

# 해결 방법 
## 클래스 분리하기 

### Storage Service 인터페이스 정의 
```java
public interface StorageService {  
    InputStream getFileAsInputStream(String storagePath) throws IOException;  
    boolean uploadFile(String storagePath, MultipartFile file) throws IOException;  
    boolean deleteFile(String storagePath) throws IOException;  
    boolean isFileExists(String storagePath);  
}
```
인터페이스로 정의해서 추후에 유지보수에 자유롭도록 하였다. 현재 서버 컴퓨터에 파일을 저장하도록 되어 있지만 추후에 클라우드 서비스를 이용해서 파일을 따로 관리할 수도 있고, 스토리지 전용 서버를 운용할 수 있기 때문이다. 

### LocalStroage 서비스 정의하기
```java
@Service  
public class LocalStorageService implements StorageService {  
    @Value("${storage.location}")  
    private String storageLocation;  
  
    @Override  
    public InputStream getFileAsInputStream(String storagePath) throws IOException {  
        Path path = Paths.get(storageLocation, storagePath);  
        return Files.newInputStream(path);  
    }  
  
    @Override  
    public boolean uploadFile(String storagePath, MultipartFile file) throws IOException {  
        Path path = Paths.get(storageLocation, storagePath);  
        Files.write(path, file.getBytes());  
        return Files.exists(path);  
    }  
  
    @Override  
    public boolean deleteFile(String storagePath) throws IOException {  
        Path path = Paths.get(storageLocation, storagePath);  
        return Files.deleteIfExists(path);  
    }  
  
    @Override  
    public boolean isFileExists(String storagePath) {  
        Path path = Paths.get(storageLocation, storagePath);  
        return  Files.exists(path);  
    }  
}
```

### 변경후 다운로드 서비스 레이어 코드 
```java
public FileDownloadDTO downloadFile(String filename, String username) throws IOException, NullPointerException{  
    MetaData metaData = metadataRepository.findByOriginalFilenameAndUsername(filename, username);  
    if (metaData == null) {  
        throw new NullPointerException("Failed to download file: MetaData not found");  
    }  
  
    InputStream inputStream = null;  
    inputStream = storageService.getFileAsInputStream(metaData.getStoragePath());  
    if (inputStream == null) {  
        throw new NullPointerException("Failed to download file: File not found");  
    }  
    return new FileDownloadDTO(inputStream, metaData.getOriginalFilename(), MediaType.parseMediaType(metaData.getContentType()), metaData.getSize());  
}
```
이전에 private 메소드로 기능했던 것을 새로운 서비스 레이어로 빼서 불러들이는 것으로 바꾸었다. 
## 바뀐 테스트 코드
```java
@Test  
void downloadFile_Success() throws IOException {  
    String filename = "test.txt";  
    String username = "testUser";  
    UUID fileId = UUID.randomUUID();  
    MetaData metaData = new MetaData(fileId, username, "text/plain", filename, 100L);  
  
    when(metadataRepository.findByOriginalFilenameAndUsername(filename, username)).thenReturn(metaData);  
    when(metaDataService.getFileAsInputStream(metaData.getStoragePath())).thenReturn(Files.newInputStream(Files.createTempFile("test", ".txt"))  
  
    // Create a temporary file for testing  
    String[] st = metaData.getOriginalFilename().split("\\.");  
    Path tempFile = Files.createTempFile(fileId.toString() + "-" + st[0], ".txt");  
    Files.write(tempFile, "Test content".getBytes());  
  
  
    FileDownloadDTO downloadDTO = metaDataService.downloadFile(filename, username);  
  
    assertNotNull(downloadDTO);  
    assertEquals(filename, downloadDTO.filename());  
    assertEquals(MediaType.TEXT_PLAIN, downloadDTO.contentType());  
    assertEquals(100L, downloadDTO.size());  
  
    // Clean up  
    Files.delete(tempFile);  
}
```


# 추후 고민할 점 
- 서비스 레이어에서 또 다른 서비스레이어를 불러들이는 것이 맞는지? 컨트롤러 레이어에서 두 서비스 레이어를 불러와야 하는지 궁금했다. 
- 만약 하나 서비스 레이어에서 다른 서비스 레이어 불러온다면 현재처럼 메타데이터에서 파일를 불러오는 것이 맞는지? 
	- 반대로 파일 클래스가 메타 데이터를 불러오는 것이 좀 더 자연스러운 것이 아닌지? 

