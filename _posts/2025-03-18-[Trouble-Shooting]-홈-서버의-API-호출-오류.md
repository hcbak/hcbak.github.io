---
title: "[Trouble Shooting] 홈 서버의 API 호출 오류"
author: hcbak
date: 2025-03-18 17:34:19 +0900
categories: [Server]
img_path: /assets/img/posts/blog/2025-03-18-[Trouble-Shooting]-홈-서버의-API-호출-오류/
---

## 발단

나는 집에서 서버를 운영하고 있다. 그리고 최근, 집에 있는 서버에서 돌릴 새 프로젝트를 진행하고 있었다.

![01](01_네트워크.webp)

대략 방화벽 안에 있는 A서버에 API 서버를 구축하고, 방화벽 밖에 있는 B서버에 Python Script를 작성하여 이벤트가 발생할 때 마다 A서버로 데이터를 보내서, A서버는 요청받은 데이터를 DBMS에 저장하는 구조를 가지고 있다.

## 전개

API 서버와 스크립트가 준비되었고, 모니터링을 하면서 조회 역할을 맡을 새로운 API 서버를 개발하고 있었는데, 스크립트에서 API 서버로 이벤트를 전송할 때 이상하게 403 오류가 한 번씩 발생했다. 반복해서 보내면 보내지길래 retry를 넣고 지나갔는데, 스크립트에서 또 이상한 기류를 포착했다.

![02](02_오류.webp)

이번에는 530 오류였다. 이번 오류는 Retry를 계속 늘려도 오류가 계속 돌아왔다. 잘 시간(23시)이 되어 이미 일기까지 쓰고 잠들기만 하면 됐는데, 통신이 거의(90% 이상) 이루어지지 않아 어떻게든 해결을 해야 했다.

### 범인은 Cloudflare?

나는 Cloudflare라는 CDN 서비스를 이용하고 있다. 위의 방화벽 위에 Cloudflare가 있다고 생각하면 된다.

마침 내가 방문하던 사이트가 느려져서 '아? 이거 보기 드문 Cloudflare 문제인가?!' 하고 인터넷 검색을 먼저 했다.

하지만 검색 결과도 딱히 없었고(Cloudflare 장애는 뉴스가 나올 정도로 드물고 치명적인 문제이다.) 서버의 상태는 [Cloudflare Status](https://www.cloudflarestatus.com/){:target="_blank"}
이쪽에서 확인할 수 있는데 별다른 문제는 보고된 바가 없었다.

### 범인은 API 서버?

이제 500번대 서버 에러를 줄 수 있는 친구는 내 서버 밖에 없다. 하지만 요리 보고 조리 봐도 서버에는 로그가 하나도 생기지 않았고, 로그가 생긴 순간에는 스크립트도 200 ok를 던지고 있었다.

변경이 없는데 중간에 이상해질 이유도 없고...

### 범인은 스크립트?

그렇다면 변경이 있었던 스크립트가 문제인가? 하고 보기는 했는데 사실 그랬다면 반복 요청에 드물게 발생하는 200 ok가 존재해서는 안됐다.

![03](03_오류.webp)

대강 이런 식이었다. 이 때가 되어서는 왜 요청이 반복되면 200 ok가 떨어지는지 궁금해서 1 2 4 3번이 아니라 1초로 10번을 시도하라고 스크립트를 고친 상태였다.

### 범인은 DDNS?

나는 고정 IP를 할당받지 않았지만, 웬만해서는 수 년이 지나도 아이피를 가져가지 않는 킹갓KT를 사용하고 있었다. 하지만 그래도 혹시 모른다. 정전 등으로 KT 장비의 테이블이 날아가서 나의 IP를 바꿔버릴지! A 레코드로 전부 작성했다가는 한 방에 모든 서비스가 죽어버리는 것이다.

그래서 나는 서버를 처음 소유했던 대학생 시절 부터 DDNS 서비스를 이용해왔다. 처음에는 ASUS 라우터에서 제공하는 DDNS 서비스를 이용하다가, 분리된 서버와 방화벽을 장만하면서 **Duck DNS**로 이전했다. 따라서 내 DNS는 거의 대부분이 Duck DNS에서 제공받은 도메인 주로로 CNAME이 걸려 있었다.

검증을 위해 간단하게 내가 서비스하는 다른 것들도 접속했더니 Cloudflare 530 페이지가 나왔고, 여기에서 실마리를 잡았다.

![04](04_오류.webp)

530 에러는 Cloudflare에서 사용하는 비공식 에러코드인데, Cloudflare에서 호스트를 확인할 수 없을 때 발생시킨다고 한다. 그리고 530 에러 페이지에는 1000번대 오류 코드가 추가로 있는데, 1016 에러는 DNS에서 목적지를 확인할 수 없다는 오류였다.

정리하자면 이렇다.

1. Cloutflare가 요청을 받았다.
2. 목적지를 알아내기 위해 DNS를 확인해 보니 CNAME으로 또 도메인이 적혀 있다.
3. 목적지를 알아내기 위해 적힌 도메인으로 DNS 요청을 날린다.
4. **Duck DNS에서 응답을 안 준다!**
5. 목적지를 찾을 수 없으므로 요청을 폐기하고 1016 에러코드를 530 에러에 담아 응답을 보낸다.

그렇다면 내 DDNS에 무언가 문제가 있는 것이 틀림 없었다.

### 범인은 방화벽?

나는 DDNS를 KT 장비 바로 아래에 있는 방화벽에서 관리하고 있다. 웬만해서는 문제를 일으키지 않는 장비라, 접근도 까다롭고 잘 들어가지도 않는다.

이렇게 저렇게 해서 장비에 접근을 했는데, DDNS 설정은 내가 처음에 설정 한 그대로였고, 업데이트에 대한 응답이 이루어지지 않는 것으로 보였다. 잘 작동하는 장비는 만지는 것이 아니니 문제의 파편화를 막기 위해 그대로 두고 나왔다.

아니, 그러면 Duck DNS가 문제야?

### 범인은 Duck DNS!!

DNS 서버가 장애가 생길 수가 있을까? 그렇다. 사실은 이 Duck DNS라는 친구는 가끔 장애가 생기던 친구였다. 다만, 내가 이렇게 지속적인 API 요청을 보내는 서비스를 처음 개발하여, 일시적으로 발생하는 장애를 체감할 수 없었던 것이었다.

검색을 해보니 따로 공식 상태 모니터링 페이지를 운영하지는 않는 것으로 보이고 [Down for Everyone](https://downforeveryoneorjustme.com/duck-dns){:target="_blank"} 라는 사이트에서 사용자들의 보고를 기반으로 그래프를 작성하고 있었다.

![05](05_그래프.webp)

대략 2 ~ 3시간 가량 장애가 있었고, 검색이 되는 것을 보니 한 두 번이 아닌 것으로 보였다. 너무 Cloudflare 의존적으로 변하는 것 같지만 추후에 DDNS도 Cloudflare로 이전해야 할 것 같다.

일단 당장은 다른 방법이 없으므로 CNAME으로 작성된 DNS를 A 레코드로 IP를 직접 찍는 방향으로 수정을 했다. 서버와 클라이언트 양쪽의 DNS 캐시를 비우고 다시 시도하니 **530 에러가 더이상 보이지 않았다!**

### 범인은 Cloudflare!!

![06](06_403.webp)

하지만 200 ok 대신 **403 에러가 엄청나게 발생**했다. 하지만 그래도 10번 retry를 하면 50% 확률로 200 ok가 떨어졌으므로, 2시간 30분 고생을 마치고 일단 자러 갔다.

나는 API 서버에 별다른 보안 조치를 하지 않았기 때문에 403 에러는 Cloudflare가 유력했다. 아침에 눈을 뜨자 마자 핸드폰으로 Cloudflare를 살펴봤다. 그랬더니 어이가 없게도, **유례없는 엄청난 요청**(??: 안 보내지면 10번 반복해서 보내면 되지~)에 놀란 Cloudflare가 **DDoS 방어**를 걸고 있었다.

![07](07_underattack.webp)

아는 사람은 알겠지만 Cloudflare는 엄청난 물량을 기반으로 한 DDoS 방어 능력을 갖추고 있고, 나도 이 서비스를 이용하기 위해 Cloudflare로 왔었다. 내 Real IP를 알지 못하면 DDoS의 방향이 Cloudflare로 향하고, Cloudflare는 공짜로 막아준다고 하니! 비록 Cloudflare의 망 사용료 부담으로 외국을 거치게 되어 딜레이가 발생하기는 했지만, 혼자 쓰는데는 문제가 없었다.

**Under Attack 모드**가 활성화되면 Cloudflare는 요청을 받아서 서버로 보내기 전에 정상 요청인지 확인 절차를 거치는데, 웹 페이지로 보면 캡챠 뱅글뱅글 돌아가는 그게 잠깐 뜨다가 당신 사람 맞다! 하고 넘어가는 그 페이지다. 여기에서 내 스크립트의 요청이 정상적인 요청이 아니라고 판단하여 403 에러를 봔환했던 것이었다.

#### Under Attack 모드 해제

Under Attck 모드를 해제해야 한다.

위의 대시보드 페이지에서 **빠른 동작**에 Under Attack 스위치를 누르면 창이 열리고, **I'm Under Attack!**을 **기본적으로 끔**으로 변경하면 된다.

![08](08_해제.webp)

일단은 WAF 설정의 유효성을 확인하기 위해 아직 변경은 하지 않았다.

#### WAF 예외

왼쪽 사이드바에서 [보안 → Analytics]에 들어가면 WAF의 활동 내역을 볼 수 있다. 추후에도 Under Attack 모드가 활성화되면 안되니 여기에 예외를 걸었다.

![09](09_보안.webp)

**WAF로 완화됨**이 아무래도 이 친구가 위협을 느낀 결과로 보인다.

WAF에 들어가서 스크립트의 아이피를 **건너뛰기**로 설정하고 저장한다.

![10](10_waf.webp)

![11](11_waf.webp)

![12](12_waf.webp)

이후에 들어오는 트래픽이 **완화되지 않음**으로 변경되면 성공적이다. 장기간 스크립트를 운영하여 트래픽 평균이 조정되었을 때, 이 설정을 비활성화하고 다시 상황을 한번 지켜봐야겠다.

![13](13_waf.webp)

**이 시점에서 스크립트의 통신은 완전히 정상화되었다!** 그리고 Under Attack 모드도 해제했다.

## 후기

역시 집에서 서버를 운영한다는 것은 쉬운 일이 아니다. 하지만 발생하는 문제를 해결하다 보면 재미있기도 하고 더 다양한 방향으로 생각할 수 있게 되는 것 같다!

하지만 당분간은 장애가 발생하지 않았으면 좋겠다. ㅎ;

## 참고

- CloudFlare Docs
  - Support
    - Troubleshooting
      - Cloudflare Errors
        - Troubleshooting Cloudflare 1XXX errors
          - [Error 1016: Origin DNS error](https://developers.cloudflare.com/support/troubleshooting/cloudflare-errors/troubleshooting-cloudflare-1xxx-errors/#error-1016-origin-dns-error){:target="_blank"}
        - Troubleshooting Cloudflare 5XX errors
          - [Error 530](https://developers.cloudflare.com/support/troubleshooting/cloudflare-errors/troubleshooting-cloudflare-5xx-errors/#error-530){:target="_blank"}
      - HTTP Status Codes
        - 4xx Client Error
          - 403 Forbidden
            - [Cloudflare-specific information](https://developers.cloudflare.com/support/troubleshooting/http-status-codes/4xx-client-error/#cloudflare-specific-information-2){:target="_blank"}
