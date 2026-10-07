# 19주차 — GuardDuty·네트워크

## 이번 주 목표

GuardDuty Finding이 원본 로그를 대신하는 것이 아니라 조사 시작점이라는 점을 이해하고, CloudTrail·VPC Flow Logs·DNS·S3 로그를 이용한 조사 계획과 보고서 2개를 만듭니다.

> 현재 실습 폴더에는 실제 GuardDuty Finding과 VPC Flow Logs가 없습니다. 이번 주는 가상 Finding을 설계하고, 기존 CloudTrail 증거로 확인 가능한 것과 추가 수집이 필요한 것을 명확히 구분합니다.

## 가상 Finding A

```text
type: CredentialAccess:IAMUser/AnomalousBehavior
severity: 8.0
resource: IAMUser/temp-support
remoteIp: 203.0.113.89
firstSeen: 2026-09-01T01:18:19Z
summary: 새 계정에서 이례적인 CloudTrail StopLogging 호출
```

## 가상 Finding B

```text
type: Exfiltration:S3/AnomalousBehavior
severity: 5.0
resource: S3/lab-audit-bucket
principal: assumed-role/LabReadOnly/session-01
remoteIp: 192.0.2.80
firstSeen: 2026-09-01T02:01:55Z
summary: 평소와 다른 세션의 객체 접근
```

## 주야비비 8일 실행표

| 일차 | 근무 | 시간 | 할 일 | 결과물 |
|---|---:|---:|---|---|
| 1 | 주간 | 1시간 | Finding 필드와 의미 | 필드 표 |
| 2 | 야간 | 30분 | 탐지·원본 차이 복원 | 카드 |
| 3 | 비번 | 3시간 | Finding A 조사 계획 | 보고서 1 |
| 4 | 비번 | 7시간 | Finding B 조사 계획 | 보고서 2 |
| 5 | 주간 | 1시간 | VPC Flow Logs 필드 | 노트 |
| 6 | 야간 | 30분 | 증거 소스 순서 복원 | 점검 |
| 7 | 비번 | 3시간 | 수집 공백·한계 작성 | 한계 표 |
| 8 | 비번 | 7시간 | 보고서·통과 시험 | 완성본 |

## Finding A에서 확인 가능한 CloudTrail

```spl
index=cloud_security_lab sourcetype="aws:cloudtrail" userIdentity.userName="temp-support"
| table _time userIdentity.type userIdentity.userName sourceIPAddress awsRegion eventSource eventName requestParameters.*
| sort _time
```

추가로 필요한 증거:

- 사용자 생성과 정책 연결의 승인 기록
- 해당 자격증명의 Access Key 사용 내역
- 같은 IP의 다른 API 호출
- CloudTrail 실제 중지 여부와 로그 공백
- IAM 자격증명 보고서와 MFA 상태

## Finding B에서 확인 가능한 CloudTrail

```spl
index=cloud_security_lab sourcetype="aws:cloudtrail" eventName=GetObject
| eval actor=coalesce('userIdentity.userName','userIdentity.arn')
| table _time actor sourceIPAddress awsRegion requestParameters.bucketName requestParameters.key
```

추가로 필요한 증거:

- Role 신뢰 정책과 세션 발급 주체
- S3 객체 민감도·크기·버전
- 같은 세션의 List/Get 요청
- VPC Endpoint·프록시·Flow Logs
- 정상 기준선과 데이터 소유자 확인

## VPC Flow Logs에서 볼 필드

```text
srcaddr, dstaddr, srcport, dstport, protocol,
packets, bytes, start, end, action, log-status,
interface-id, vpc-id, subnet-id, instance-id
```

`ACCEPT`는 보안상 정상이라는 뜻이 아니라 네트워크 정책상 허용되었다는 뜻입니다. 패킷 내용과 사용자 행위는 Flow Logs만으로 알 수 없습니다.

## 탐지와 원본 로그 비교

| 구분 | GuardDuty Finding | 원본 로그 |
|---|---|---|
| 역할 | 위험 신호와 요약 | 실제 이벤트 증거 |
| 장점 | 우선순위·맥락 제공 | 세부 필드·타임라인 |
| 한계 | 모델 논리·집계가 추상화됨 | 분석가가 의미를 연결해야 함 |
| 조사 | 시작점 | 판정과 범위 확인 근거 |

## 보고서 결론 수준

현재 데이터만으로 Finding B를 유출로 확정하지 않습니다. `GetObject 1건 관찰`, `AssumedRole 세션`, `객체명`은 사실이고, 실제 유출 여부는 객체 크기·민감도·정상 기준선·네트워크 전송 증거가 필요하다고 적습니다.

## 통과 시험

1. Finding과 원본 로그의 차이를 설명한다.
2. 각 Finding에서 확인 가능한 사실과 추정을 나눈다.
3. CloudTrail·Flow Logs·S3 로그의 역할을 구분한다.
4. `ACCEPT`의 의미를 정확히 설명한다.
5. 두 사례에 필요한 추가 증거를 5개씩 제시한다.

## 최종 체크리스트

- [ ] 가상 Finding 2개의 조사 보고서를 작성했다.
- [ ] Finding 필드와 원본 필드를 연결했다.
- [ ] 확인 가능한 사실과 수집 공백을 구분했다.
- [ ] 네트워크·S3 추가 조사 계획을 적었다.
- [ ] 탐지만으로 사고를 확정하지 않았다.

