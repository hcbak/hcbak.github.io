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

## Trouble Shooting

이게 몇 가지 알아야 할 게 있는데, SHOW MASTER STATUS에서 받아온 Postion은 변경 로그의 index인 것으로 보인다.

저 변경 로그가 발생하면 Slave에서 로그를 가져와서 그대로 적용하는 것으로 보이는데, 여기에서 문제가 몇 가지 생긴다.

Master - Slave 전부 아예 백지 상태에서 시작하면 문제가 없는데, **이미 운영중인 DBMS를 Replication 하고자 할 때 동기화가 제대로 되지 않는 문제**가 생긴다.

Master 쪽에서 운영 중인 Database를 그대로 복제해주는 것이 아니라 **변경사항만 로그로** 보내주기 때문에, 텅 빈 Slave의 Database는 CREATE DATABASE를 제외하고는 **허공에 쿼리를 날리게 된다**. 그렇게 오류나서 뻗는다.

따라서 만약 Master가 이미 운영중이라면, 해당 Database를 복제해서 Slave에 미리 넣어주고 Replication을 시작해야 한다.

비슷한 문제인데 **다수의 Database**가 Master에서 운영중이라면 그것의 **일부만 Replication하는 것이 불가능**하다.

원하는 Database만 잘 복제해놨다고 해도 다른 Database에서 쿼리가 발생하면 허공 쿼리를 시전하고 그대로 뻗어버리게 된다.

겉으로 보기에는 좋아 보였는데, 막상 적용하고 보니 설정값을 수동으로 넣는 것도 그렇고 개복치마냥 뻗고 제대로 알려주지도 않는 것으로 보여서 실 서비스에 적용은 정말 많은 테스트를 거치고 진행해야 할 것 같다.
