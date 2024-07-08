---
title: "GitHubActions로 자동 테스트 CI 도입하기"
date: 2024-07-02 16:20:02 +0500
categories:
    - CI
tags:
    - ci
    - githubactions
permalink: /github-actions-ci
---


이전에 회사에서 githubActions 를 도입해본 경험이 있지만 PR 이전에 Integration 작업 도입해 본 적이 없다. 마침 스프링으로 사이드 프로젝트를 만들어보면서 테스트 코드를 작성해보고 있기 때문에 PR 이전에 테스트 코드가 모두 통과하도록 하는 CI를 만들어 보도록 하겠다. 
여러 CI/CD 툴이 있겠지만 GitHubActions를 선택하였다. 현재 레포를 github에 포스팅하고 있기 때문에 연동하기 쉽고, 사용 방법이 간단하다는 장점이 있기 때문이다. 결국 CI/CD 툴도 컴퓨팅 리소스가 필요한데. 다른 툴을 도입하려면 컴퓨팅 리소스를 자체적으로 마련해야 하지만 githubActions는 컴퓨팅 리소스를 따로 마련하지 않고 무료로 사용할 수 있다는 점도 장점이다. (actions를 돌릴 수 있는 시간이 정해진 것으로 알고 있는데. 이 또한 개인 플젝 기준으로 거의 무료로 쓸 수 있다.) 
GiHubAction는 다음과 같이 정의하면 된다. 
```
- .github
	- workflow
		- {원하는 파일이름}.yml 
```

# 초기 ci.yml 
```yml
name: Java CI with Gradle  
  
on:  
  push:  
    branches: [ "main" ]  
  pull_request:  
    branches: [ "main" ]  
  
jobs:  
  build:  
  
    runs-on: ubuntu-latest  
    permissions:  
      contents: read  
  
    steps:  
    - uses: actions/checkout@v4  
    - name: Set up JDK 22  
      uses: actions/setup-java@v4  
      with:  
        java-version: '22'  
        distribution: 'corretto'  
  
    - name: Grant execute permission for gradlew  
      run: chmod +x gradlew  
  
    # Gradle test 실행  
    - name: Test with Gradle  
      run: ./gradlew --info test  
        
    # 테스트 후 Result 출력  
    - name: Publish Unit Test Results  
      uses: EnricoMi/publish-unit-test-result-action@v1  
      if: ${{ always() }}  # 'always' : 테스트 실패해도 Result 출력  
      with:  
        files: build/test-results/**/*.xml  
  
    # 캐시 파일 삭제  
    - name: Cleanup Gradle Cache  
      if: ${{ always() }}  
      run: |  
        rm -f ~/.gradle/caches/modules-2/modules-2.lock  
        rm -f ~/.gradle/caches/modules-2/gc.properties  
```
해당 파일의 워크플로우는 다음과 같다. 
1. `main` branch로 `push` or `PR`이 이뤄지면 아래 jobs이 수행된다.
2. 해당 브랜치 소스 코드를 `checkout`한다 
3. JDK 22를 세팅한다 
4. gradlew 에 권한을 부여한다 
5. `gradlew test` 수행한다 
6. 테스트 수행 결과를 출력한다 
7. 캐시 파일을 삭제한다. 

# 문제 상황 
위 CI를 수행하는 과정에서 실패하였다. 
```terminal
Description:
~~
a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.
~~
 Reason: Failed to determine a suitable driver class
```
"DataSource 정보가 정확하지 않다" 는 에러 메세지를 확인할 수 있었다. 현재 내 컴퓨터에서 실행 중인 mysql 를 테스트 서버로 삼고 있었다. 테스트를 실행하는 컴퓨터에서 mysql 데이터베이스를 찾을 수 없었기 때문에 문제가 발생하고 있었다. 

### 해결 방안 1: 내 컴퓨터 mysql 연결하기 
이렇게 만들려면 네트워크 설정이 복잡해진다. 인터넷에 내 외부에서 내 컴퓨터로 연결이 열어줘야 하는데. 보안적으로 위험할 뿐만 아니라 집이 아닌 외부로 나간다면 이 방법을 지속할 수 없다. 

### 해결 방안 2: mysql 클라우드 생성 후 연결
Azure의 데이터베이스나 프리 티어 서비스를 이용할 수 있지만 자주 일어나지 않는 테스트를 위해서 클라우드 데이터베이스 여는 것이 옳바른지 모르겠다..

### 해결 방안3: Test 시 DB 연결 설정 끄기 
현재 테스트 코드에서 데이터베이스를 직접적으로 이용하지 않기 때문에 아예 테스트 시 DB 연결하려는 설정는 끄는 방법도 있다. 괜찮은 방법이지만 추후에 데이터베이스를 이용하는 테스트에 또 다시 설정을 수정해야 하고 그때 또 해결방안을 고민해야 하는 점에서 문제를 단순히 미루는 방법이라고 판단했다. 
### 해결 방안4: H2 내장 데이터베이스 사용
Spring 프로젝트에서 가장 쉽게 데이터베이스를 쓸 수 있는 내장 H2를 사용할 수 있다. 이를 이용해서 테스트에 도입하면 쉽게 테스트용 데이터베이스를 구축할 수 있으니 괜찮을 것 같다. 

## H2 데이터베이스 연결 
H2 드라이버를 프로젝트 패키지에 추가했다. `application-test.yml`을 정의하여서 테스트 시 h2 연결이 가능하도록 하였다. 
자세한 방법은 [여기](https://www.baeldung.com/spring-testing-separate-data-source) 를 참고하면 된다. 

## Publish Unit Test 에러 
```terminal 
github.GithubException.GithubException: 403 {"message": "Resource not accessible by integration", "documentation_url": "[https://docs.github.com/rest/checks/runs#create-a-check-run](https://docs.github.com/rest/checks/runs#create-a-check-run)", "status": "403"}
```
위와 같은 에러가 발생했다. 권한 문제로 판단하고 해당 스텝을 수행하는 [레포](https://github.com/EnricoMi/publish-unit-test-result-action)에 들어가보니 권한 문제가 발생하면 yml 파일에 다음과 같이 추가해주면 된다고 했다. 
```
permissions:
  checks: write
  pull-requests: write
```
그렇게 다시 한번 시도해봤지만 해결 되지 않았다. 

한참을 구글링하다가,, 같은 문제를 겪은 사람들이 단순 권한만 부여해서 해결되었다는 글을 보고 이건 나만 겪는 문제라고 판단해서 삽질을 열나게 했다.... 

### 문제 발견: 이미 권한 부여가 되어 있었다.. 
```
jobs:  
  build:  
  
    runs-on: ubuntu-latest  
    permissions:  
      contents: read 
```
 앞 선 yml에 jobs 하위에 Permissions 이미 정의되어 있었다. 그래서 상위에서 아무리 permission 을 작성하여도 하위 스코프에서 권한 부여가 제대로 되지 않았다.. 
 기존 권한 부여 삭제하니 잘 작동하였다.. 

# 최종 CI.yml 
```yml 
name: Java CI with Gradle  
  
on:  
  push:  
    branches: [ "main" ]  
  pull_request:  
    branches: [ "main" ]  
  
permissions:  
  checks: write  
  pull-requests: write  
  
jobs:  
  build:  
  
    runs-on: ubuntu-latest  
  
    steps:  
    - uses: actions/checkout@v4  
    - name: Set up JDK 22  
      uses: actions/setup-java@v4  
      with:  
        java-version: '22'  
        distribution: 'corretto'  
  
    - name: Grant execute permission for gradlew  
      run: chmod +x gradlew  
  
    # Gradle test 실행  
    - name: Test with Gradle  
      run: ./gradlew --info test  
        
    # 테스트 후 Result 출력  
    - name: Publish Unit Test Results  
      uses: EnricoMi/publish-unit-test-result-action@v1  
      if: ${{ always() }}  # 'always' : 테스트 실패해도 Result 출력  
      with:  
        files: build/test-results/**/*.xml  
  
    # 캐시 파일 삭제  
    - name: Cleanup Gradle Cache  
      if: ${{ always() }}  
      run: |  
        rm -f ~/.gradle/caches/modules-2/modules-2.lock  
        rm -f ~/.gradle/caches/modules-2/gc.properties  
  
```