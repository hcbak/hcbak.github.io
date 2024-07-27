---
title: "MariaDB Replication"
author: hcbak
date: 2024-07-25 17:39:50 +0900
categories: [DBMS, MariaDB]
---

- 테스트 환경  
Ubuntu 24.04  
MariaDB 10.11.8

## Mariadb replication

2대 이상의 Mariadb Server를 Master - Slave 구조로 복제하여 운영하는 방식이다.

DB Server의 부하를 분산시킬 수 있고 데이터 백업의 효과도 가질 수 있다.

\*MariaDB의 기본적인 세팅은 완료했다고 가정한다.  

### Master
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

```
...
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replica, see README.Debian about other
#       settings you may need to change.
server-id              = 1
log_bin                = /var/log/mysql/mysql-bin.log
expire_logs_days        = 10
max_binlog_size        = 100M
...
```
아래 세 개 속성 라인의 주석을 해제한다.  
server-id  
expire_logs_days  
max_binlog_size

```bash
sudo systemctl restart mariadb
```

Slave에서 접근할 계정을 생성한다.

```sql
GRANT REPLICATION slave on *.* to slave@'%' IDENTIFIED BY 'password';
```

```sql
SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000000 |      000 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.005 sec)
```
File과 Position의 값을 slave에 입력해 줄 것이다.

File → master_log_file  
Position → master_log_pos

### Slave
[mysqld] 아래 4개 라인을 작성해준다.
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

...
[mysqld]
server-id = 2
relay_log = mysql-relay-bin
log_slave_updates = 1
read_only = 1
...
```

재시작 후 MariaDB에 root로 접속한다.

```bash
sudo systemctl restart mariadb

sudo mariadb
```

Master와 연결해준다.

```sql
CHANGE MASTER TO master_host='0.0.0.0', master_port=3306, master_user='slave', master_password='password', master_log_file='mysql-bin.000000', master_log_pos=0000;

START SLAVE;

SHOW SLAVE STATUS \G;

-- root를 제외한 관리자 권한 제거
UPDATE mysql.user SET super_priv='N' WHERE user <> 'root';
```
CHANGE MASTER를 잘못 입력했을 경우 STOP SLAVE를 입력하여 SLAVE 모드를 종료하고 다시 CHANGE MASTER 문을 입력해준다.

이후 Master 서버에서 Datebase나 Table을 변경해보고 Slave에 동기화가 되는지 확인하면 된다.
