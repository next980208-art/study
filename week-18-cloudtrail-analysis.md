# 18주차 — CloudTrail 분석

## 이번 주 목표

`userIdentity`, `eventName`, `sourceIPAddress`, 리전과 요청 파라미터를 사용해 누가·언제·어디서·무엇을 했는지 답합니다. CloudTrail 탐지 3개를 만듭니다.

## 데이터 확인

```spl
index=cloud_security_lab sourcetype="aws:cloudtrail"
| stats count by eventName
```

ConsoleLogin, CreateUser, AttachUserPolicy, StopLogging, GetObject가 각 1건입니다.

## 주야비비 8일 실행표

| 일차 | 근무 | 시간 | 할 일 | 결과물 |
|---|---:|---:|---|---|
| 1 | 주간 | 1시간 | CloudTrail 공통 필드 | 필드 사전 |
| 2 | 야간 | 30분 | 5W 질문 복원 | 카드 |
| 3 | 비번 | 3시간 | 콘솔 로그인 탐지 | 규칙 1 |
| 4 | 비번 | 7시간 | 사용자 생성·정책 연결 타임라인 | 규칙 2 |
| 5 | 주간 | 1시간 | StopLogging 탐지 | 규칙 3 |
| 6 | 야간 | 30분 | actor 표현 복원 | 점검 |
| 7 | 비번 | 3시간 | 관리·데이터 이벤트 구분 | 노트 |
| 8 | 비번 | 7시간 | 문서화·통과 시험 | 탐지 3개 |

## 공통 타임라인

```spl
index=cloud_security_lab sourcetype="aws:cloudtrail"
| eval actor=coalesce('userIdentity.userName','userIdentity.arn')
| table _time actor userIdentity.type sourceIPAddress awsRegion eventSource eventName requestParameters.* responseElements.*
| sort _time
```

## 탐지 1 — MFA 없는 콘솔 로그인 실패

```spl
index=cloud_security_lab sourcetype="aws:cloudtrail" eventName=ConsoleLogin responseElements.ConsoleLogin=Failure additionalEventData.MFAUsed=No
| table _time userIdentity.userName sourceIPAddress awsRegion responseElements.ConsoleLogin additionalEventData.MFAUsed
```

예상은 `lab-analyst`, `198.51.100.24`, 실패 1건입니다.

## 탐지 2 — 사용자 생성 후 고권한 정책 연결

```spl
index=cloud_security_lab sourcetype="aws:cloudtrail" eventName IN (CreateUser,AttachUserPolicy)
| eval target_user=coalesce('requestParameters.userName','responseElements.user.userName')
| stats values(eventName) as actions min(_time) as first_seen max(_time) as last_seen values(userIdentity.userName) as actors values(sourceIPAddress) as source_ips values(requestParameters.policyArn) as policies by target_user
| where mvfind(actions,"CreateUser")>=0 AND mvfind(actions,"AttachUserPolicy")>=0
| convert ctime(first_seen) ctime(last_seen)
```

예상 대상은 `temp-support`입니다. 두 이벤트가 같은 주체·IP에서 약 1분 21초 간격으로 발생합니다.

## 탐지 3 — 감사 로깅 중지

```spl
index=cloud_security_lab sourcetype="aws:cloudtrail" eventName=StopLogging
| eval severity="critical"
| table _time severity userIdentity.userName sourceIPAddress awsRegion requestParameters.name
```

예상은 `temp-support`가 `lab-trail`을 중지한 1건입니다.

## 관리 이벤트와 데이터 이벤트

- `CreateUser`, `AttachUserPolicy`, `StopLogging`은 제어 영역의 변경을 보여주는 관리 이벤트입니다.
- `GetObject`는 S3 객체 접근이라는 데이터 영역 행위입니다.
- 실제 CloudTrail 설정에서는 데이터 이벤트가 별도 설정·비용 대상일 수 있으므로 수집 여부를 확인해야 합니다.

## 5W 답변 형식

```text
누가: userIdentity.type + userName/arn
언제: eventTime/_time
어디서: sourceIPAddress + awsRegion
무엇을: eventSource + eventName
대상: requestParameters의 사용자·정책·Trail·버킷·키
결과: responseElements + errorCode/errorMessage
```

## 통과 시험

1. 다섯 이벤트를 시간순으로 설명한다.
2. IAM User와 AssumedRole actor를 한 필드로 만든다.
3. 사용자 생성과 정책 연결을 같은 대상 기준으로 묶는다.
4. StopLogging이 중요한 이유를 설명한다.
5. 각 이벤트에 대해 5W 질문에 답한다.

## 최종 체크리스트

- [ ] CloudTrail 탐지 3개를 문서화했다.
- [ ] actor와 target을 구분했다.
- [ ] 이벤트 결과와 요청 파라미터를 확인했다.
- [ ] 관리 이벤트와 데이터 이벤트를 구분했다.
- [ ] 탐지 결과를 원본 JSON과 비교했다.

