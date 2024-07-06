---
title: "[한화 Beyond SW 캠프] 프리코스 Git&Github"
author: hcbak
date: 2024-07-06 14:55:23 +0900
categories: [Study, Hanwha Beyond SW Camp]
---

한화 Beyond SW 캠프에 참여하게 되었다.

본 과정은 아직 시작하지 않았지만 프리코스라고 하여 선행학습을 요구했다.  
사실 아무것도 할 게 없어서 쓸데없이 C#을 공부하면서 시간을 때우고 있었기 때문에 반가운 일이었다.

Git&Github 과정과 Java 기초 과정이 있었는데 Git에 먼저 익숙해지고 Java는 Git을 복습하면서 같이 공부하면 좋을 것 같아서 Git 먼저 시작했다.

Command 방식의 Git 사용법을 알려주기 위해 기초적인 bash 명령어 등을 알려주는데, 이미 리눅스에 익숙해서 다 아는 내용이기 때문에 Git 명령어에 대해서만 정리한다.

## Git

### 초기 설정
#### name, email 설정
```bash
git config --global user.name ""
git config --global user.email ""
```
Commit한 사람의 정보를 작성한다.  
--global은 전역 설정을 의미한다.

#### config 확인
```bash
git config --list
```
config 값 전체를 확인할 수 있다.

#### 기본 브랜치명 수정
```bash
git config --global init.defaultBranch main
```
main 부분을 원하는 이름으로 변경할 수 있다.  
기본값은 master이다. 예전에 master-slave 구조를 공부하면서 당연하게 받아들였는데 강사의 말로는 주인-노예의 의미라서 차별적일 수 있다나... main을 권장한다고 한다.

### 주요 명령어
#### 초기화
```bash
git init
```
현재 위치한 경로를 Git으로 관리할 수 있게 초기화를 하는 명령어이다.

#### 상태 확인
```bash
git status
```

#### Commit 할 파일 지정
```bash
git add .
git rm --cached ""
```
add로 추가하고 rm로 지운다. add의 마지막 .은 해당 폴더 전체를 의미한다.  
.을 지우고 하나하나 지정할 수도 있다.  

#### Commit
```bash
git commit -m ""
```
Commit을 한다. Commit은 반드시 메시지를 포함해야 하므로 -m 파라미터로 메시지를 넘겨준다.

#### 이전 Commit 확인
```bash
git log --graph
```

#### 특정 버전의 특정 파일 내용 확인
```bash
git show main:readme.md
```
main은 branch 명이고, commit hash로 대체할 수 있다.

#### 변경 사항 확인
```bash
git diff # 현재 작업 트리 ↔ 스테이징 영역
git diff --staged # 스테이징 영역 ↔ 최신 커밋
git diff "hash" # 현재 작업 트리 ↔ hash에 해당하는 커밋
```

#### branch 이름 변경
```bash
git branch -M main
```

### 원격 저장소 활용
Github에서 레포지토리를 생성하는 작업을 선행해야 한다.

#### 원격 주소 등록
```bash
git remote add origin "https://[Token]@github.com/username/repository"
git remote -v # 상태 확인
git remote set-url origin "" # 주소 변경
```
Github에서 보안 토큰을 생성한 후에 URI에 맞춰서 주소를 작성하면 된다.

##### * URI
scheme:[//[user[:password]@]host[:port][/path][?query][#fragment]]

#### 로컬 레포지토리 원격으로 업로드
```bash
git push -u origin main
```

#### 원격 저장소 업로드 루틴(예시)
```bash
git status
git add .
git status
git commit -m "message"
git push origin
```

#### 복제
```bash
git clone ""
```
큰따옴표 안에 원격 저장소의 주소를 넣으면 해당 저장소의 작업 내용을 그대로 복제해올 수 있다.

#### 병합
```bash
git fetch # 원격 저장소와 동기화
git merge # 병합
```
자동으로 병합이 되면 행복 코딩이 가능하지만 안되어 충돌하면 나중에 push 시도한 사람이 수동으로 병합하는 게 관례인 것 같다.  
상상만 해도 ㅂㄷㅂㄷ하다.

수동으로 병합을 마쳤다면 add, commit, push 과정을 추가로 진행해서 병합한 최종본을 원격 저장소에 반영해야 한다.

### Branch
#### 조회
```bash
git branch # 모든 브랜치
git branch -a # 원격 브랜치 포함
git branch -r # 원격 브랜치
git branch -v # 브랜치와 마지막 커밋
```

#### 생성
```bash
git branch branch-name
```

#### 삭제
```bash
git branch -d branch-name
```

#### 이동
```bash
git checkout branch-name
```

#### 병합
```bash
git checkout main
git merge feature
```
feature 브랜치를 main 브랜치에 병합하는 명령어이다.  
main 브랜치로 이동한 후에 merge로 병합할 브랜치를 지정해주면 된다.