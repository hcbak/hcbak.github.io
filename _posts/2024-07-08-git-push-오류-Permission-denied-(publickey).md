---
title: "git push 오류 Permission denied (publickey)"
author: hcbak
date: 2024-07-08 17:36:27 +0900
categories: [VCS, Git]
img_path: /assets/img/posts/blog/2024-07-08-git-push-오류-Permission-denied-(publickey)/
---

### 오류
CLI Git에서 remote 주소를 지정하고 첫 push를 했을 때 위와 같은 오류가 발생했다.

```bash
$ git push -u origin main
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

### 해결
검색해보니 Public Key를 생성하여 Github에 등록하라는 답변이 있는데, 정확히 맞는 말이지만 내 의도는 그것이 아니었다.

#### SSH 방식
위의 방법은 SSH로 Github에 연결하는 방법이고, 아래와 같이 주소를 입력하고 push를 한 결과였을 것이다.
```bash
git remote add origin git@github.com:hcbak/new-repository.git
```
만약 SSH 방식으로 연결하기를 정말 원한다면, 오류 문구를 참고해서 Github 계정에 Public Key를 등록하면 된다

#### HTTPS 방식
하지만 웹에서 로그인을 하는 방식으로 인증하기를 원한다면 HTTPS 방식으로 원격 주소를 등록해야 한다.
```bash
git remote add origin https://github.com/hcbak/new-repository.git
```
이렇게 하고 push를 시도하면 웹페이지가 열리고 Github 로그인 창이 나올 것이다.

SSH와 HTTPS방식이 어떻게 달라지는지는 Github에서 처음 Repository를 만들면 있는 버튼으로 확인할 수 있다.

![01](01_github.png)