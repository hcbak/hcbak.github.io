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
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replica, see README.Debian about other
#       settings you may need to change.
server-id              = 1
log_bin                = /var/log/mysql/mysql-bin.log
expire_logs_days        = 10
max_binlog_size        = 100M
```

server-id  
expire_logs_days  
max_binlog_size  
주석 해제

```bash
sudo systemctl restart mariadb
```

```sql
GRANT REPLICATION slave on *.* to slave@'%' IDENTIFIED BY 'password';
```

```bash
SHOW MASTER STATUS
```

### Slave
```bash
sudo nano /etc/mysql/mariadb.conf.d/50.server.cnf
```

[mysqld] 밑에 아래 라인을 추가한다.
```
[mysqld]
server-id = 2
relay_log = mysql-relay-bin
log_slave_updates = 1
read_only = 1
```

```bash
sudo systemctl restart mariadb

sudo mariadb
```

```sql
CHANGE MASTER TO master_host='0.0.0.0', master_port=3306, master_user='master', master_password='password', master_log_file='mysql-bin.000000', master_log_pos

START SLAVE;

SHOW SLAVE STATUS \G;

UPDATE mysql.user SET super_priv='N' WHERE user <> 'root';
```
