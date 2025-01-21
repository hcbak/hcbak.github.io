---
title: "[AWS Skill Builder] AWS Cloud Practitioner Essentials 정리"
author: hcbak
date: 2025-01-21 22:47:13 +0900
categories: [Cloud, AWS]
---

## 보안(Security)

### 공동 책임 모델  
Shared Responsibility Model [(Link)](https://aws.amazon.com/ko/compliance/shared-responsibility-model/){:target="_blank"}

Customer|클라우드 '내부'의 보안을 책임
AWS|클라우드 '자체'의 보안을 책임

### 사용자 권한 및 액세스

AWS 계정 루트 사용자: 계정에 있는 모든 리소스에 엑세스하고 제어함
생성 즉시 MFA(Multi-factor authentication) 활성화

### AWS IAM  
AWS Identify and Access Management

권한을 명시적으로 부여하기 전에는 로그인을 포함한 모든 작업이 불가능

#### 최소 권한의 원칙

사용자에게 필요한 것에만 엑세스 부여

#### 정책

Json 파일로 이루어져 있는 권한 명시

Effect|Allow 또는 Deny
Action|아무 AWS API 호출
Resource|API 호출이 필요한 AWS 리소스

#### 그룹

그룹에 정책을 할당하면 해당 그룹에 속한 모든 사용자에게 정책 적용

#### 역할

일시적으로 할당하고, 사용자 이름 또는 암호 없음

이전에 부여받은 권한을 모두 포기해야 역할 변경 가능

### AWS Organizations

여러 AWS 계정을 관리하는 중앙 위치

- 중앙 집중식 관리
- 통합 결제 (사용량 기준 할인 가능)
- 계정의 계층적 그룹화
- AWS 서비스 및 API 작업 액세스 제어

#### 서비스 제어 정책(SCP)

각 계정의 사용자 및 역할이 액세스할 수 있는 AWS 서비스, 리소스 및 개별 API 작업을 제한

### 규정 준수  

AWS는 다양한 국가의 규정을 준수하고 있고, 이에 대한 보고서를 제공받을 수 있음

#### AWS Artifact

AWS의 규정 준수 문서에 접근할 수 있음

### 분산 서비스 거부(DDoS)

UDP FLOOD|보안 그룹
SLOWLORIS|ELB

기본적으로 AWS 규모의 이점으로 위 공격 등은 성공하려면 AWS 리전 전체를 압도해야 함

그 외에 AWS Shield를 사용하여 DDoS 보호 서비스 이용 가능

### 추가 보안 서비스

#### 암호화

권한 있는 당사자만 액세스 할 수 있는 방식으로 메시지 또는 데이터 보호

- 저장중 암호화
  - DynamoDB - AWS KMS와 연동 가능
- 전송중 암호화
  - AWS 서비스 ↔ 클라이언트 - TLS

전송중 암호화는 사실상 거의 모든 서비스에서 제공

#### AWS Key Management Service

암호화 작업을 수행할 암호화 키를 생성, 관리 및 사용할 수 있음

#### Amazon Inspector

인스턴스 대상 자동화된 보안 평가

- 네트워크 구성 연결성 부분
- Amazon 에이전트
- 보안 평가 서비스

#### Amazon GuardDuty

AWS CloudTrail 이벤트, Amazon VPC Flow Logs, DNS 로그에서 발견된 네트워크 활동을 알려진 악성 IP 주소, 이상 탐지, 기계 학습 등 통합 위협 인텔리전스로 분석하는 위협 탐지 시스템

독립적으로 실행되므로 기존 인프라 및 워크로드의 성능 및 가용성에 영향을 미치지 않음

## 모니터링

시스템을 관찰하고 지표를 수집한 다음 데이터를 사용하여 의사 결정

### Amazon CloudWatch

CloudWatch 경보

임계를 설정하고 알림을 받을 수 있음.

지표나 지표의 변동 사항을 그래프로 보여주는 대시보드 기능 제공

- 중앙 위치에서 모든 지표에 엑세스
- 애플리케이션, 인프라 및 서비스에 대한 가시성 확보
- MTTR 절감 및 TCO 개선
- 애플리케이션 및 운영 리소스 최적화를 위한 인사이트 확보

### AWS CloudTrail

어떤 요청이든 관계 없이 AWS에 대한 모든 요청은 CloudTrail 엔진에 기록됨

### AWS Trusted Advisor

자동화된 조언 도구

5대 핵심 원칙을 기준으로 리소스를 평가해줌

- 비용 최적화
- 성능
- 보안성
- 내결함성
- 서비스 한도

고객이 이해할 수 있도록 결과를 범주화해서 제공

## 요금 및 지원

### AWS 프리 티어

|유형|예시|
|-|-|
|상시 무료|AWS Lambda는 매월 100만 건의 무료 호출 허용|
|12개월 무료|Amazon S3는 최대 5GB의 표준 스토리지에 대해 12개월 동안 무료로 제공됨|
|평가판|AWS Lightsail은 최대 750시간 사용량의 1개월 평가판 제공

### AWS 요금 적용 방식

- 실제 사용한 만큼만 지불합니다.
- 예약하는 경우 비용이 감소합니다.
- 많이 사용할수록 볼륨 기반 할인이 적용되어 비용이 감소합니다.

### 결제 대시보드

- AWS 청구서를 결제
- 사용량 모니터링
- 비용을 분석 및 제어

### 통합 결제

AWS Organizations에서 통합 결제도 가능함

- 청구 프로세스 간소화
- 계정 간 절감 효과 공유
- 무료 기능

### AWS 예산 (AWS Budgets)

사전에 리소스에 배정한 예산이 정해진 임계치 이상으로 사용되었을 때 알림 설정 가능

### AWS Cost Explorer

AWS에서 지출하는 비용을 시각적으로 확인하고 분석할 수 있음

지난 12개월 데이터 제공

사용자 정의 보고서 작성 가능

### AWS Support 플랜

#### Basic Support

- 24/7 고객 서비스
- 설명서
- 백서
- 지원 포럼
- AWS Trusted Advisor
- AWS Personal Health Dashboard

#### Developer Support

- Basic Support
- 이메일로 고객 지원 엑세스

#### Business Support

- Basic 및 Developer Support
- 모든 AWS Trusted Advisor 검사
- 클라우드 지원 엔지니어와 직접 통화

#### AWS Enterprise On-Ramp Support

- Basic, Developer 및 Business Support
- 비즈니스 크리티컬 워크로드에 대한 30분 응답 시간
- Technical Account Manager(TAM)풀 사용

#### AWS Enterprise Support

- Basic, Developer 및 Business Support
- 비즈니스 크리티컬 워크로드에 대한 15분 응답 시간
- 전담 Technical Account Manager(TAM)

### AWS Marketplace

AWS 아키텍처에서 실행되는 타사 소프트웨어를 검색, 배포, 관리하는 단계를 간소화하는 큐레이트된 디지털 카탈로그

- 사용자 지정 약관 및 가격
- Private marketplace
- 조달 시스템 통합
- 비용 관리 도구

## 마이그레이션 및 혁신

### AWS Cloud Adoption Framework (AWS CAF)

AWS로 신속하고 원활하게 마이그레이션할 수 있도록 조언을 제공

- 비즈니스
- 인력
- 거버너스
- 플랫폼
- 보안
- 운영

### 마이그레이션 전략

6R 전략

- 리호스팅(Rehosting)
- 리플랫포밍(Replatforming)
- 폐기(Retire)
- 유지(Retain)
- 재구매(Repurchasing)
- 리팩터링(Refactoring)

### AWS Snow 패밀리

네트워크가 아닌 방법으로 외부에서 AWS로 데이터를 운반하는 경우의 솔루션

#### AWS Snowcone

최대 14TB 운반 가능

#### AWS Snowball

디바이스를 인프라에 연결하면 AWS Lambda, EC2 호환 AMI 등을 바로 사용할 수 있음

#### AWS Snowmobile

컨테이너 한 대에 100PB

데이터 센터를 통째로 AWS로 옮기는 경우

### AWS를 통한 혁신

#### 기계 학습

- Amazon SageMaker

#### 인공 지능

- Amazon CodeWhisperer
  - 코드를 작성하는 동안 코드 제안을 얻고 코드에서 보안 문제를 식별
- Amazon Transcribe
  - 음성을 텍스트로 변환
- Amazon Comprehend
  - 텍스트에서 패턴을 검색
- Amazon Fraud Detector
  - 잠재적인 온라인 사기 행위를 식별
- Amazon Lex
  - 음성 및 텍스트 챗봇 구축

## 참고

[AWS Cloud Practitioner Essentials (Korean)](https://explore.skillbuilder.aws/learn/courses/13522/aws-cloud-practitioner-essentials-hangug-eo){:target="_blank"}
