---
title: "Ceph PGs autoscaling 고장"
author: hcbak
date: 2026-04-18 11:54:15 +0900
categories: [Cloud, Ceph]
---

## 환경

Ceph|20.2(tentacle)
Deploy|cephadm

## 문제

RGW Pool(rgw.buckets.data)이 일정 수준 이상의 데이터가 쌓였음에도 PGs를 자동으로 관리하지 않고 변화가 없는 상태가 유지되었다. PGs 수가 고정이라도 복제는 되고 있으니 당장은 문제가 없지만, 복구를 PGs 단위로 하기 때문에 장기적으로 해소해야 할 문제이다.

최신 버전의 Ceph은 Pool에 대한 PGs를 직접 조정하지 않고 autoscaling으로 자동으로 조정한다. 따라서 해당 모듈이 살아있는지 먼저 확인한다.

```bash
sudo ceph mgr module ls | grep pg_autoscaler
pg_autoscaler         on (always on)
```

상태를 확인한다.

```bash
sudo ceph osd pool autoscale-status
```

내 경우에는 결과가 아무것도 나오지 않았다. 어떠한 pool도 autoscaling으로 관리하지 않는다.

도대체 무슨 일인가 싶었는데, 사례가 잦았는지 Ceph 공식문서에 해당 상황에 대한 내용이 나온다.

> `autoscale-status`에서 아무런 결과도 출력되지 않으면, CRUSH 적용 문제일 수 있다.

무슨 말이냐 하면, 예를 들어 저장장치 유형(HDD, SSD 등)으로 CRUSH Rule을 나눠놓고 특정 pool만 원래의 CRUSH Rule(저장장치 유형 미정)로 남겨 놓으면, 원래의 CRUSH Rule이 어느 저장장치 유형을 사용해야 할 지 길을 잃게 되고, 이것에 대한 해결이 없는 상태에서는 autoscaling이 전체적으로 작동을 하지 않게 되는 것이다.

나의 경우에는 과거 NVMe OSD를 추가하면서 전체적으로 CRUSH Rule을 분리(HDD와 NVMe)한 적이 있었는데, 그 시점에서 `.mgr` pool이 새로 분리한 어떠한 Rule에도 속하지 않은 상태로 기본 룰(`replicated_rule`)로 남아 있어 모든 pool의 autoscaling이 스킵된 사례였다.

## 해결

따라서 마지막 남은 `.mgr` pool도 새로 분리한 CRUSH Rule로 저장장치 유형을 지정해줘야 한다.

```bash
sudo ceph osd pool set .mgr crush_rule replicated_nvme
```

이렇게 나머지 하나까지 CRUSH Rule을 지정하면 즉시 `autoscale-status`에 모든 pool이 보이게 되고, 그동안 autoscaling이 멈춰 있던 pool에 `NEW PG_NUM`이 활성화되면서 새로운 `PG_NUM`이  반영된다.

```bash
sudo ceph osd pool autoscale-status
```

```
POOL                          SIZE  TARGET SIZE  RATE  RAW CAPACITY   RATIO  TARGET RATIO  EFFECTIVE RATIO  BIAS  PG_NUM  NEW PG_NUM  AUTOSCALE  BULK
.mgr                                              2.0                0.0000                                  1.0       1              on         False
.rgw.root                                         2.0                0.0000                                  1.0      32              on         False
default.rgw.log                                   2.0                0.0000                                  1.0      32              on         False
default.rgw.control                               2.0                0.0000                                  1.0      32              on         False
default.rgw.meta                                  2.0                0.0000                                  4.0      32              on         False
default.rgw.buckets.index                         2.0                0.0000                                  4.0      32              on         False
default.rgw.buckets.non-ec                        2.0                0.0000                                  1.0      32              on         False
default.rgw.buckets.data                          2.0                0.0875                                  1.0      32         128  on         True
```

```
POOL                          SIZE  TARGET SIZE  RATE  RAW CAPACITY   RATIO  TARGET RATIO  EFFECTIVE RATIO  BIAS  PG_NUM  NEW PG_NUM  AUTOSCALE  BULK
.mgr                                              2.0                0.0000                                  1.0       1              on         False
.rgw.root                                         2.0                0.0000                                  1.0      32              on         False
default.rgw.log                                   2.0                0.0000                                  1.0      32              on         False
default.rgw.control                               2.0                0.0000                                  1.0      32              on         False
default.rgw.meta                                  2.0                0.0000                                  4.0      32              on         False
default.rgw.buckets.index                         2.0                0.0000                                  4.0      32              on         False
default.rgw.buckets.non-ec                        2.0                0.0000                                  1.0      32              on         False
default.rgw.buckets.data                          2.0                0.0875                                  1.0     128              on         True
```

autoscaling이 작동하지 않은 지 오랜 시간이 지났다면 거대해진 PGs를 쪼개기 위해 `active+remapped+backfill_wait`상태의 PGs가 대량으로 발생하여 많은 recovery작업이 쌓일 수 있다. 따라서 I/O 성능 저하가 있을 수 있으므로 사용량이 적은 시간대에 작업을 하는 것을 권장한다.

## 참고

- Ceph Documentation
  - Ceph Storage Cluster
    - Operations
      - Placement Groups
        - [Viewing PG scaling recommendations](https://docs.ceph.com/en/tentacle/rados/operations/placement-groups/#viewing-pg-scaling-recommendations){:target="_blank"}
