---
title: "도커를 곁들인 프로젝트 리팩토링하기"
date: 2024-01-05 16:20:02 +0500
categories:
    - Docker 
tags:
    - Docker
permalink: /refactoring-with-docker
---

CV를 업로드하면 기술 인터뷰를 연습해 볼 수 있는 [프로젝트](https://github.com/ddoddii/resume-ai-chat) 에 합류하게 되었다. 이미 백엔드 서버가 구성되어 있었기 때문에 퍼블릭 레포로 변경하면서 필요한 작업과 자동 배포 위주로 기여하였다. 

# 기존 문제
- 보안 이슈 : 퍼블릭 레포로 변경하면서 하드 코딩되어 있는 보안 키 노출
- 자동 배포 : AWS EC2에 SSH로 접근하여 수동 배포 
기존 문제를 정의해보면 위와 같았다. 

# 문제해결 방향
## 보안 이슈
노출되면 안될 보안키를 .env파일로 옮겨두고 dotenv 라이브러리로 환경변수를 불러오는 식으로 변경하였다. 레포를 검토하고 보안 이슈가 생길 코드를 이슈로 제보하였다. 문제가 될 코드의 종류는 다음과 같았다. 
- 디비 경로 (sqlite를 사용했기 때문에 디비 파일 경로)
- 개인 컴퓨터 파일 경로 (/User/유저이름/home 과 같은 경로) 
- JWT 시크릿키, 알고리즘 종류
- GPT 모델 종류 
이와 같은 종류 코드는 보안 문제를 예방하려는 차원에서 선정하였다. 보안 문제는 보수적으로 접근해야 하기 때문에 "알고리즘 종류"나 "개인 컴퓨터 파일 경로"와 같은 불필요한 정보는 노출되지 않는 것이 좋다고 보았다. ".env"파일에 변수를 정리하면 하드코딩하지 않고 주요 변수를 관리할 수 있어서 유지보수 차원에서 이득이 될 수 있다. 

## 자동 배포
자동 배포를 구현하기 앞서 몇가지 선택지를 고려해야 했다. 

### CD 도구? GitHubActions vs Jenkins 
결론적으로 GitHubActions를 선택하기로 했다. 그 이유는 다음과 같다. 

1. 비용: 자동 배포를 위해서 컴퓨터 리소스를 할당해야 하는데. GitHubActions는 일정 수준까지 무료로 이용할 수 있다. 반면, Jenkins는 따로 컴퓨팅 자원을 마련해야 한다. 
2. 구성의 편리성: GitHubActions는 yaml 형식 스크립트를 짜면 바로 CI/CD 모두 구현하기 쉽다. 반면, Jenkins는 설치부터 환경설정까지 오랜 시간이 걸릴 것으로 예상하였다. 

###  Code vs Docker
CD 도구를 선택했다면, 다음으로 어떤 형식으로 백엔드 서버에 배포할 코드를 전달할지 고민하였다. 선택지도 정리해보면 아래와 같았다. 
1. 코드를 압축하여 서버에 전달하기 
2. AWS EB 사용하기 (AWS EB는 S3에 코드를 압축하여 필요한 버전을 로드하는 식으로 동작한다. 도커도 도입할 수 있다.)
3. Docker 사용하기 + DockerHub

3번 선택지인 Docker를 사용하기로 했다. 가장 큰 이유는 도커를 공부해보고 싶은 팀원의 의지가 강했다는 점이다. Redis와 같은 다른 툴을 사용할 있는 확정성이 강하다는 점도 선택의 이유가 될 수 있다. 

## 구현
구현 과정에서 [@소은](https://github.com/ddoddii) 과 함께 페어 프로그래밍하였다.
함께하는 팀원이 도커를 처음 사용해보기도 하고, 디버깅하기 쉽게 하기 위해서 단계를 나눠서 프로그래밍하였다. 
그 단계는 다음과 같다. 
1. 로컬에서 도커 빌드 실행
2. 도커 허브에 올려보기 
3. 이를 수동으로 ec2에서 pull해서 실행하기 
4. 이 과정을 깃헙액션으로 맡기기
5. 디비 사용을 위한 볼륨 설정하기 
6. 환경 변수 사용을 위한 .env 파일 주입하기 

추후에 튜토리얼 형식으로 코드와 함께 글을 올리도록 하겠다.. 

### 최종 결과물 
```
#dockerfile

FROM python:3.10

WORKDIR /app/

COPY . /app/
COPY ./requirements.txt /app/

RUN pip install -r requirements.txt

VOLUME /app/resume-db

CMD uvicorn --host=0.0.0.0 --port 8000 main:app

```

```
#GitHubActions 
name: ci

on:
  push:
    branches:
      - "main"

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      -
        name: Checkout
        uses: actions/checkout@v4
      -
        name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      -
        name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      -
        # Build and push to dockerhub
        name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/${{ secrets.PROJECT_NAME }}:latest

      # EC2 인스턴스 접속 및 애플리케이션 실행
      -
        name: Application Run
        uses: appleboy/ssh-action@v0.1.6
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_KEY }}

          script: |
            sudo docker ps -a --filter "name=${{ secrets.PROJECT_NAME }}" | xargs -r docker stop
            sudo docker ps -a --filter "name=${{ secrets.PROJECT_NAME }}" | xargs -r docker kill 
            sudo docker ps -a --filter "name=${{ secrets.PROJECT_NAME }}" | xargs -r docker rm -f
            sudo docker rmi ${{ secrets.DOCKERHUB_USERNAME }}/${{ secrets.PROJECT_NAME }} 
            sudo docker pull ${{ secrets.DOCKERHUB_USERNAME }}/${{ secrets.PROJECT_NAME }}
  
            sudo docker run -p  ${{ secrets.PORT }}:${{ secrets.PORT }} \
            --env-file ${{ secrets.ENV }} \
            --name ${{ secrets.PROJECT_NAME }} \
            -v /home/ubuntu/resume_ai_chat.db:/app/resume_ai_chat.db \
            -d ${{ secrets.DOCKERHUB_USERNAME }}/${{ secrets.PROJECT_NAME }}
```

레포를 자세히 알고 싶다면? [링크](https://github.com/ddoddii/resume-ai-chat/tree/main)

# Shout-out 
이번 프로젝트 진행하면서 잔소리랑 헛소리를 번갈아가면서 했음에도 잘 따라준 [@소은](https://github.com/ddoddii)님에게 감사하다고 전하고 싶다. 
