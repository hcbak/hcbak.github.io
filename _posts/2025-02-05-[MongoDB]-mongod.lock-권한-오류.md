---
title: "[MongoDB] mongod.lock 권한 오류"
author: hcbak
date: 2025-02-05 22:41:58 +0900
categories: [DBMS, MongoDB]
---

## 환경

OS|OpenMediaVault 7.4.13-1

## 발단

이전에 MongoDB를 사용하려고 내 NAS에 설치하려고 시도한 적이 있었다. 당시 최신 버전이 정상적으로 실행되지 않았는데, 원인은 CPU의 AVX 미지원 때문이었다. 임시로 NAS의 HDD를 마운트한 메인 서버에 MongoDB 경로만 NAS HDD로 바꿔서 동작시키고 사용했었다.

별로 좋은 구조는 아니기 때문에 이제 다른 DBMS처럼 NAS에 이전하고 NAS쪽 Port를 잡아서 서비스를 돌리려고 하였고, AVX 지원의 조건이 5버전 이후로 생겼다고 하길래 4버전을 설치했다.

## 문제 발생

일단 설치를 하고 그냥 돌렸더니 정상적으로 실행됐다. 하지만 데이터 경로를 HDD로 옮겨야 했기 때문에 /etc/mongod.conf 파일에서 dbPath를 수정했다. 이후 실행을 시켰더니 오류가 발생했다.

```bash
sudo systemctl start mongod
sudo systemctl status mongod
```

<details>
  <summary>결과</summary>
  <div markdown="1">

```text
× mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; enabled; preset: enabled)
     Active: failed (Result: exit-code) since Wed 2025-02-05 13:16:55 KST; 4s ago
   Duration: 573ms
       Docs: https://docs.mongodb.org/manual
    Process: 3482681 ExecStart=/usr/bin/mongod --config /etc/mongod.conf (code=exited, status=100)
   Main PID: 3482681 (code=exited, status=100)
        CPU: 240ms

Feb 05 13:16:55 systemd[1]: Started mongod.service - MongoDB Database Server.
Feb 05 13:16:55 mongod[3482681]: {"t":{"$date":"2025-02-05T04:16:55.497Z"},"s":"I",  "c":"CONTROL",  "id":7484500, "ctx":"main","msg":"Environment variable MONGODB_CONFIG_OVERRIDE_NOFORK == 1, overriding \"processManagement.fork\" to false"}
Feb 05 13:16:55 systemd[1]: mongod.service: Main process exited, code=exited, status=100/n/a
Feb 05 13:16:55 systemd[1]: mongod.service: Failed with result 'exit-code'.
```

  </div>
</details><br>

여기서 원인을 제대로 보여주지 않아서 조금 헤맸는데, mongodb의 로그파일을 보면 정확한 원인이 나온다.

```bash
tail -f /var/log/mongodb/mongod.log
```

<details>
  <summary>결과</summary>
  <div markdown="1">

```json
...
{"t":{"$date":"2025-02-05T13:29:29.526+09:00"},"s":"E",  "c":"STORAGE",  "id":20557,   "ctx":"initandlisten","msg":"DBException in initAndListen, terminating","attr":{"error":"Location28596: Unable to determine status of lock file in the data directory /PATH/mongodb/4.4.29: boost::filesystem::status: Permission denied: \"/PATH/mongodb/4.4.29/mongod.lock\""}}
...
```

  </div>
</details><br>

권한이 없어서 mongod.lock 파일에 접근할 수 없다고 한다.

당연히 나는 디렉토리의 권한을 전부 바꿔줬었고, 이게 도대체 왜 오류가 생기는지 납득을 할 수 없었다. 구글링을 하면서 시도해본 내용은 대략 이렇다.

- 정상 실행 후 생성된 /var/lib/mongodb 폴더를 '그대로' copy해서 HDD 경로에 복사 후 conf 파일 갱신 후 실행
- 여러가지 오만가지 방법으로 삭제 후 재설치
- "도대체 왜" 외치기

## 실마리 - service 파일

mongod.conf 파일을 불러올 것으로 예상되는 mongod.service 파일을 열어보았다.

```bash
nano /lib/systemd/system/mongod.service
```

<details>
  <summary>결과</summary>
  <div markdown="1">

```text
[Unit]
Description=MongoDB Database Server
Documentation=https://docs.mongodb.org/manual
After=network-online.target
Wants=network-online.target

[Service]
User=mongodb
Group=mongodb
EnvironmentFile=-/etc/default/mongod
Environment="MONGODB_CONFIG_OVERRIDE_NOFORK=1"
ExecStart=/usr/bin/mongod --config /etc/mongod.conf
RuntimeDirectory=mongodb
# file size
LimitFSIZE=infinity
# cpu time
LimitCPU=infinity
# virtual memory size
LimitAS=infinity
# open files
LimitNOFILE=64000
# processes/threads
LimitNPROC=64000
# locked memory
LimitMEMLOCK=infinity
# total threads (user+kernel)
TasksMax=infinity
TasksAccounting=false

# Recommended limits for mongod as specified in
# https://docs.mongodb.com/manual/reference/ulimit/#recommended-ulimit-settings

[Install]
WantedBy=multi-user.target
```

  </div>
</details><br>

EnvironmentFile에 적힌 경로에는 파일 자체가 없었고, config파일이 문제인가 싶어서 기본 설정으로 실행되라 하고 service 파일의 ExecStart에 적힌 mongod를 config 없이 그냥 돌렸다. 그리고 당연히 성공. dbPath를 바꾸기 전에는 잘 실행이 되었으니 당연한 결과였다.

그렇다면 진짜 권한 문제인데... 권한... chmod chown 다 했는데... 어?

... 과거 OMV라는 것을 처음 썼을 때 기억이 스쳐 지나갔다.

## 문제 해결 - Linux Access Control Lists

OMV에서는 HDD를 추가하고 공유 폴더를 만들 때 접근 제어 목록을 설정할 수 있다. 당시 무지했던 나는 '무슨 권한을 계정마다 또 재귀적으로 주는거지?' 하고 기능을 의심했었는데, 거기에 분명 CLI에서 보이지 않던 권한이 있었다!!

OMV WebGUI로 가서 기억이 이끄는 대로 기능을 찾고(버전이 올라가서 GUI가 바뀐 관계로 바로는 못찾았다...), **File access control lists** 설정을 찾아낸 후 mongodb 유저에게 해당 폴더에 대한 권한을 줬다.

지금 시점에서 권한을 주면서 다시 보니 '아... 이거 그건데... 뭐였지...'라는 생각이 드는 것도 신기했고, 결과적으로 권한을 준 후 실행이 되었다!!

- mongod --dbpath /PATH/mongodb/4.4.29/
  - 먼저 이걸로 명령어를 내린 계정으로 파일이 생기는 것을 확인했다.

그리고...

```bash
sudo systemctl start mongod
sudo systemctl status mongod
```

<details>
  <summary>결과</summary>
  <div markdown="1">

```text
● mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; enabled; preset: enabled)
     Active: active (running) since Wed 2025-02-05 15:06:29 KST; 1s ago
       Docs: https://docs.mongodb.org/manual
   Main PID: 3498750 (mongod)
     Memory: 56.9M
        CPU: 1.167s
     CGroup: /system.slice/mongod.service
             └─3498750 /usr/bin/mongod --config /etc/mongod.conf

Feb 05 15:06:29 systemd[1]: Started mongod.service - MongoDB Database Server.
Feb 05 15:06:29 mongod[3498750]: {"t":{"$date":"2025-02-05T06:06:29.406Z"},"s":"I",  "c":"CONTROL",  "id":7484500, "ctx":"main","msg":"Environ>
```

  </div>
</details><br>

드디어 성공했다!

이후에 Linux ACLs를 찾아보면서 계속 권한 문제라고 외쳤던 mongod에게 미안함을 느꼈다... 완전히 잊고 있었던 Linux ACLs에 대해서도 한번 정리를 해야 할 것 같다.

## 번외 - Failed to unlink socket file

마구 실행시키고 어쩌고 하다 보면 갑자기 이런 오류도 생긴다.

```json
{"t":{"$date":"2025-02-05T14:49:02.510+09:00"},"s":"E",  "c":"NETWORK",  "id":23024,   "ctx":"initandlisten","msg":"Failed to unlink socket file","attr":{"path":"/tmp/mongodb-27017.sock","error":"Operation not permitted"}}
```

왜 찌꺼기를 남기는 지 모르겠지만 가차없이 쳐내면 된다.

```bash
sudo rm /tmp/mongodb-27017.sock
```

## 참고

- [Ubuntu Community Help Wiki - FilePermissionsACLs](https://help.ubuntu.com/community/FilePermissionsACLs){:target="_blank"}
