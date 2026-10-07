# 17주차 — AWS 기본 구조와 IAM

## 이번 주 목표

AWS 계정·리전·IAM 사용자·역할·정책·MFA·최소 권한을 설명하고 합성 계정 구조도와 용어 노트를 만듭니다. 실제 AWS 리소스를 만들지 않고 CloudTrail 합성 데이터로 학습합니다.

## 합성 시나리오

```text
AWS Account 111122223333
├─ IAM User: lab-admin
├─ IAM User: lab-analyst
├─ IAM User: temp-support
└─ IAM Role: LabReadOnly
   └─ AssumedRole session: session-01
```

## 주야비비 8일 실행표

| 일차 | 근무 | 시간 | 할 일 | 결과물 |
|---|---:|---:|---|---|
| 1 | 주간 | 1시간 | 계정·리전·공유 책임 | 용어 노트 |
| 2 | 야간 | 30분 | User와 Role 차이 복원 | 카드 |
| 3 | 비번 | 3시간 | 정책 구조·최소 권한 | 정책 해석표 |
| 4 | 비번 | 7시간 | 합성 계정 구조도 작성 | 구조도 |
| 5 | 주간 | 1시간 | MFA·자격증명 위험 | 체크리스트 |
| 6 | 야간 | 30분 | IAM 조사 질문 복원 | 점검 |
| 7 | 비번 | 3시간 | 권한 변경 이벤트 검색 | 분석 노트 |
| 8 | 비번 | 7시간 | 설명·통과 시험 | 완성본 |

## 필수 용어

| 용어 | 설명 질문 |
|---|---|
| Account | 보안·결제·리소스의 기본 경계는 무엇인가? |
| Region | 리소스와 로그가 어느 지역에 있는가? |
| IAM User | 장기 자격증명을 가질 수 있는 주체인가? |
| IAM Role | 누가 어떤 조건으로 임시 자격증명을 받는가? |
| Policy | 어떤 Action을 어떤 Resource에 어떤 조건으로 허용하는가? |
| MFA | 비밀번호 외 추가 인증이 적용되었는가? |
| Least Privilege | 업무에 필요한 최소 권한만 부여했는가? |

## User와 Role 비교

| 기준 | IAM User | IAM Role |
|---|---|---|
| 대표 용도 | 특정 사람·레거시 워크로드 | AWS 서비스·연동·임시 접근 |
| 자격증명 | 장기 키 보유 가능 | STS 임시 자격증명 |
| CloudTrail 식별 | `IAMUser`, `userName` | `AssumedRole`, `arn`, session |
| 권장 방향 | 필요 최소화·MFA·키 관리 | 짧은 세션·신뢰 정책·최소 권한 |

## CloudTrail에서 주체 확인

```spl
index=cloud_security_lab sourcetype="aws:cloudtrail"
| eval actor=coalesce('userIdentity.userName','userIdentity.arn')
| table _time actor userIdentity.type userIdentity.accountId awsRegion eventSource eventName sourceIPAddress
| sort _time
```

## 고권한 정책 연결 확인

```spl
index=cloud_security_lab sourcetype="aws:cloudtrail" eventName=AttachUserPolicy
| table _time userIdentity.userName sourceIPAddress requestParameters.userName requestParameters.policyArn awsRegion
```

확인할 질문:

1. 누가 정책을 연결했는가?
2. 누구에게 연결했는가?
3. 어떤 정책인가?
4. 어느 IP와 리전인가?
5. 승인된 변경인가?
6. 이후 그 사용자가 무엇을 했는가?

## 공유 책임 설명 연습

다음 문장을 자신의 말로 완성합니다.

```text
AWS는 클라우드 자체의 __________을 책임지고,
고객은 IAM 권한, 데이터 분류, 리소스 구성, 로그 모니터링 등 __________을 책임진다.
서비스 유형에 따라 고객 책임 범위는 달라진다.
```

## 구조도에 표시할 것

- 계정 ID와 리전
- IAM User 3개와 Role 1개
- `lab-admin → temp-support 생성`
- `lab-admin → AdministratorAccess 연결`
- `temp-support → StopLogging`
- `LabReadOnly/session-01 → S3 GetObject`

이는 시간상 연결된 합성 시나리오이며, 인과관계는 추가 증거 없이 단정하지 않습니다.

## 통과 시험

1. Account와 Region의 차이를 설명한다.
2. User와 Role 사용 차이를 예시로 설명한다.
3. 정책의 Action·Resource·Condition 의미를 말한다.
4. 최소 권한과 MFA가 필요한 이유를 설명한다.
5. CloudTrail에서 IAM User와 AssumedRole을 구분한다.

## 최종 체크리스트

- [ ] 합성 IAM 구조도를 작성했다.
- [ ] 핵심 용어를 자신의 말로 정리했다.
- [ ] 권한 연결 이벤트의 주체·대상·정책을 확인했다.
- [ ] User와 Role의 차이를 설명했다.
- [ ] 실계정이나 비용 발생 리소스를 만들지 않았다.

