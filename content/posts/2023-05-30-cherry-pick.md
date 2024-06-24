---
title: "git cherry-pick"
date: 2023-05-30 16:20:02 +05:00
categories:
    - git
tags:
    - git
    - git-cherry-pick
permalink: /git-cherry-pick
---

## 문제 상황

### 문제 상황

만약 다른 브랜치에서 쓰인 코드를 현재 브랜치에서도 쓰고 싶다면 2가지 방법이 있다. 
1. 원하는 소스 코드가 있는 브랜치에 가서 복사를 해서 붙여넣기를 통해서 현재 브랜치에 삽입힌다. 

이 방법은 상당히 ‘무식’한 방법이다. 이런 방법이 자주 쓰인다면 브랜치 관리가 의미없을 정도로 브랜치 간 꼬인 현상이 나타날 것이다. 원하는 코드가 커밋 단위로 쪼개져서 작성되어 있다면 우아하게 해결할 수 있다. 바로 Cherry-pick 을 사용할 수 있다. 즉, 

1. 체리 픽을 사용해서 해결한다. 

### 정의

**`git cherry-pick`**은 Git 버전 관리 시스템에서 특정 커밋을 다른 브랜치로 선택적으로 가져오는 명령어이다. 다른 브랜치에서 하나 이상의 커밋을 선택하여 현재 브랜치로 가져올 수 있다. 주로 다른 브랜치에서 필요한 변경사항만을 선택적으로 가져오고 싶을 때 사용한다.

### 사용 방법

**`git cherry-pick`** 명령어는 다음과 같은 형식으로 사용된다:

```
phpCopy code
git cherry-pick <commit-hash>

```

여기서 **`<commit-hash>`**는 가져올 커밋의 해시 값이나 브랜치 이름이다.

**`git cherry-pick`**은 선택한 커밋을 현재 브랜치에 적용하고, 새로운 커밋을 생성한다. 이 새로운 커밋은 원본 커밋의 변경사항만을 가지고 있다. 따라서 원본 커밋의 변경사항을 선택적으로 가져올 수 있다.

**`git cherry-pick`**은 다양한 옵션을 제공하여 더 정교한 커밋 선택을 가능하게 하다. 예를 들어, 여러 개의 커밋을 한 번에 cherry-pick할 수 있고, 충돌이 발생한 경우 충돌을 해결한 후 cherry-pick을 계속 진행할 수 있다.

### 사용 예시

![](https://velog.velcdn.com/images/inshining/post/c9497ee4-e855-4b7b-846f-42ca5c8396fd/image.png)
(출처: https://www.git-tower.com/learn/git/faq/cherry-pick)

develop 브랜치는 c1에서 분기하여서 c2, c4 커밋을 한 상태이다. 이 때, master 브랜치에서 c3까지 커밋하였고, develop 브랜치의 c2 커밋을 그대로 가져고자 한다. 

```jsx
git switch master
git cherry-pick C2 
```

체리 픽을 사용하여서 C2 커밋을 가져 올 수 있다. 

### 주의점

 **`git cherry-pick`**은 원본 커밋의 복사본을 생성하기 때문에 원본 커밋과 다른 커밋 히스토리가 생성될 위험에 조심해야한다. 따라서 **`git cherry-pick`**을 사용할 때에는 주의하여 사용하고, 원본 커밋의 이력을 유지하기 위해 적절한 접근 방법을 선택해야 한다.