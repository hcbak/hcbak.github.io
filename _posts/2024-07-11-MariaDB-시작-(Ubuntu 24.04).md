---
title: "MariaDB 시작 (Ubuntu 24.04)"
author: hcbak
date: 2024-07-11 11:38:52 +0900
categories: [DBMS, MariaDB]
---

Ubuntu 24.04에서 테스트한 내용이다.  
MariaDB는 10.11.8 버전이 설치된다.

## 설치 & 설정

### 설치
```bash
sudo apt update
sudo apt install mariadb-server

mariadb -V
```

### 접속
```bash
# 최초 root 접속
sudo mariadb

# User 생성 후 접속
mariadb -u{username} -p
```

### 종료
```sql
quit
```

### Bind IP 설정
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

...
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address            = 0.0.0.0 # 모든 네트워크 접근 허용
...

sudo systemctl restart mariadb
```

## 기본 명령어

### 계정 생성 & 삭제
```sql
-- 생성
CREATE USER 'username'@'%' IDENTIFIED BY 'password';

-- 삭제
DROP USER 'username'@'%';

-- 조회
USE mysql
SELECT User, Host FROM user;
```
% 자리는 IP 자리이다.  
%는 모든 IP에서의 접근을 허용한다는 의미이다.

#### * root 계정 패스워드 설정
```sql
USE mysql
SET password=password('password');
```

### DB 생성 & 삭제
```sql
-- 생성
CREATE DATABASE myDB;

-- 삭제
DROP DATABASE myDB;

-- 조회
SHOW DATABASES;

-- DB 접근
USE myDB
```

### 권한
```sql
-- 계정에 DB 권한 부여
GRANT ALL PRIVILEGES ON myDB.* TO 'username'@'%';

-- 부여한 권한 삭제
REVOKE ALL ON myDB.* FROM 'username'@'%';

-- 계정 권한 확인
SHOW GRANTS FOR 'username'@'%'
```

### 변경된 설정 적용
```sql
FLUSH PRIVILEGES;
```