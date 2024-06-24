---
title: "shell script - 텍스트 일치 여부 검사"
date: 2023-04-15 16:20:02 +05:00
categories:
    - shell-script
tags:
    - shell-script
    - text-matching
permalink: /shell-script-text-matching
---

## 문제 상황

<aside>
💡 실행 결과물과 예상 결과물 간 비교하여서 일치하는 지 알아보기

</aside>

C 언어로 짠 코드를 make 명령어를 통해서 빌드, 실행하면 `result` 라는 파일을 나온다. 이 결과물과 예상 결과물 사이 비교하고자 한다. 

## 문제 해결 방법

<aside>
💡 Shell Script 로 해결하기

</aside>

### shell script 작성

```jsx
vim diff_case.sh
```

우선 `diff_case.sh`  파일 만들어야 한다. 

```
#!/bin/bash

# make 명령어 실행
make

# make 명령어로 생성된 결과 파일 경로
result_file="path/to/result_file.txt"

# 테스트 결과 파일 경로
test_file="path/to/test_file.txt"

# 결과 파일과 테스트 파일 비교
diff_output=$(diff "$result_file" "$test_file")

# 결과 파일과 테스트 파일의 차이 여부 확인
if [ "$diff_output" == "" ]; then
  echo "일치합니다."
else
  echo "일치하지 않습니다."
fi
```

결과물이 나올 경로와 예상된 결과물이 나올 경로를 절대 경로로 넣어주도록 한다. 

```jsx
chmod 775 diff_case.sh 
```

위에서 생성한 파일의 권한이 없기 때문에 권한 설정을 해준다. 만약 해주지 않는 다면, `Permission Error` 가 난다. 

## 최종 결과물
![](https://velog.velcdn.com/images/inshining/post/0c220a9b-7cf1-4334-87bd-5a6763b1fc02/image.png)

일치하면 쉘에서 다음과 같은 텍스트가 나온다. 

## 발전 가능성

- make 명령어 대신 다른 실행 명령어를 넣으면 다양한 언어와 코드에 적용할 수 있다.
- 구체적으로 몇 번째 줄에서 두 텍스트 간 일치하지 않는지 알 수 있는 기능을 shell script에 넣어 볼 수 있다.