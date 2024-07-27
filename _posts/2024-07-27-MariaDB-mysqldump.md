---
title: "MariaDB mysqldump"
author: hcbak
date: 2024-07-27 16:40:50 +0900
categories: [DBMS, MariaDB]
---

- 테스트 환경  
Ubuntu 24.04  
MariaDB 10.11.8

## mysqldump

DB 백업과 복구, 혹은 복제를 목적으로 할 때 mysqldump를 사용할 수 있다.

root를 기준으로 설명한다.

### 덤프

```bash
# 관리자 패스워드를 사용하지 않는 경우
sudo mysqldump myDB > dump.sql

# 관리자 패스워드를 사용하는 경우
mysqldump -uroot -p myDB > dump.sql
```

mysqldump가 익숙하지만 Mariadb 10.5 부터 **mariadb-dump**라는 정식(?) 명칭이 생겼다고 한다.

mysqldump가 정말 너무 익숙하지만 슬슬 넘어갈 준비를 하는 게 좋아 보인다.

--no-data 옵션으로 데이터를 제외한 스키마만 백업할 수도 있다.

### 복구

복구라고 써 놓기는 했지만 sql 파일을 가져오는 명령어와 같다고 보면 된다.

한 가지 주의할 점은 mysqldump는 Database는 백업하지 않으므로 DBMS에 Database가 존재해야 한다.

```bash
# 관리자 패스워드를 사용하지 않는 경우
sudo mariadb myDB < dump.sql

# 관리자 패스워드를 사용하는 경우
mariadb -uroot -p myDB < dump.sql
```



### 참고
[MariaDB: mariadb-dump](https://mariadb.com/kb/en/mariadb-dump/){:target="_blank"}
