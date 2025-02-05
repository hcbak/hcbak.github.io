---
title: "[MongoDB] Illegal instruction (core dumped) 오류"
author: hcbak
date: 2025-02-05 21:40:28 +0900
categories: [DBMS, MongoDB]
---

## 환경

CPU|Intel® Celeron® Processor J1900
OS|OpenMediaVault 7.4.13-1

## 발단

부트캠프 때 프로젝트를 진행하는데 MongoDB가 필요했다. CI 파이프라인 용도로 작성하는 Docker Compose를 그대로 써도 되지만 어차피 배포 환경에서 DBMS 역할을 맡아주는 NAS가 있기 때문에 이 친구에게 MongoDB를 설치하고 개발이나 배포 때 사용하면 되겠다 싶어서 MongoDB 설치를 진행했다.

## 문제 발생

당시 MongoDB의 최신 버전은 8까지 나온 상태였다. 당연히 8버전으로 공식 홈페이지에서 제공하는 deb 파일을 다운로드 받아서 설치를 진행했다.

그런데 어라... 실행이 안된다..?

```bash
sudo systemctl start mongod
sudo systemctl status mongod
```

<details>
  <summary>결과</summary>
  <div markdown="1">

```text
● mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; disabled; vendor preset: enabled)
     Active: failed (Result: core-dump)
       Docs: https://docs.mongodb.org/manual
    Process: 2656736 ExecStart=/usr/bin/mongod --config /etc/mongod.conf (code=dumped, signal=ILL)
   Main PID: 2656736 (code=dumped, signal=ILL)

systemd[1]: Started MongoDB Database Server.
systemd[1]: mongod.service: Main process exited, code=dumped, status=4/ILL
systemd[1]: mongod.service: Failed with result 'core-dump'.
```

  </div>
</details><br>

```bash
journalctl -xe
```

<details>
  <summary>결과</summary>
  <div markdown="1">

```text
systemd[1]: Started MongoDB Database Server.
-- Subject: A start job for unit mongod.service has finished successfully
-- Defined-By: systemd
-- Support: http://www.ubuntu.com/support
-- 
-- A start job for unit mongod.service has finished successfully.
-- 
-- The job identifier is 1558071.
sudo[2656733]: pam_unix(sudo:session): session closed for user root
kernel: traps: mongod[2656736] trap invalid opcode ip:5621ba8d075d sp:7fff5bb29b90 error:0 in mongod[5621b5f0a000+4af3000]
systemd[1]: mongod.service: Main process exited, code=dumped, status=4/ILL
-- Subject: Unit process exited
-- Defined-By: systemd
-- Support: http://www.ubuntu.com/support
-- 
-- An ExecStart= process belonging to unit mongod.service has exited.
-- 
-- The process' exit code is 'dumped' and its exit status is 4.
systemd[1]: mongod.service: Failed with result 'core-dump'.
-- Subject: Unit failed
-- Defined-By: systemd
-- Support: http://www.ubuntu.com/support
-- 
-- The unit mongod.service has entered the 'failed' state with result 'core-dump'.
```

  </div>
</details><br>

```bash
mongod
```

<details>
  <summary>결과</summary>
  <div markdown="1">

```text
Illegal instruction (core dumped)
```

  </div>
</details><br>

명령어에 대한 결과들에서 동일한 문구(core-dump)가 보인다.

그대로 검색하면 나오는데, 안타깝게도 MongoDB와 CPU의 호환성 문제였다.

MongoDB가 5버전 이상부터 뭐가 많이 추가된 것으로 보이는데, 내 문제는 AVX라는 CPU 단위에서 제공하는 기능이 내 CPU(J1900)에 없어서 생긴 문제였다.

Intel은 2011년 1분기 이후 장비에서 지원한다고 한다.  
(J1900은 13년도 4분기 출시인데 왜...)

## 해결

하드웨어에서 지원하지 않는다면 해결 방법은 사실상 없다. 성능이고 뭐고 에뮬레이터 마냥 어떻게든 해내면 될지도 모르지만 그럴 능력도 없고~

다만 선택지는 있다.

- 4.4.29 버전을 설치한다.
  - 4.x 버전의 마지막이고 **지원이 종료(2024.2.)되었다.**
  - 바로 뒤 4.4.28 버전도 취약점(CVE-2024-1351)이 있어 사용하면 안되는 만큼 **위험을 감수해야 한다.**
  - MongoDB는 현 시점 8버전이 최신일 정도로 최근 정말 빠르게 발전하고 있다.
- 장비를 교체한다.
  - 돈이 깨지고 시간이 깨진다.

나는 아직 대놓고 취약점이 없기도 하고 완전한 내부망이기 때문에 4.4.29 버전을 설치하려고 한다.

하지만 대놓고 취약점이 발생한다 하면 답이 없기 때문에 NAS의 HDD를 NFS 등으로 메인 서버에서 Mount하고 MongoDB를 메인 서버에 설치해서 데이터 경로만 Mount 된 HDD 경로에 잡아서 사용해야 할 것 같다.

이미 장비 다 들여놓고 계속 드는 생각이지만 단순히 코어 숫자나 클럭 등의 수치도 중요하지만 세대가 올라가면서 추가되는 최적화나 기능들이 무시할 수 없을 정도로 많은 것 같다. 인코딩/디코딩도 그렇고 OS 설치도 그렇고... 등등 너무 사례를 많이 겪었다.

## 참고

- [MongoDB - 자체 관리 배포서버를 위한 프로덕션 노트 - 플랫폼 지원 참고 사항
](https://www.mongodb.com/ko-kr/docs/manual/administration/production-notes/#platform-support-notes){:target="_blank"}
- [MongoDB - Install MongoDB Community Edition on Linux(v4.4)](https://www.mongodb.com/docs/v4.4/administration/install-on-linux/){:target="_blank"}
- [MongoDB - MongoDB Software Lifecycle Schedules](https://www.mongodb.com/legal/support-policy/lifecycles){:target="_blank"}
- [NIST - CVE-2024-1351](https://nvd.nist.gov/vuln/detail/CVE-2024-1351){:target="_blank"}
