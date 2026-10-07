# 22주차 — 통합 프로젝트 완성

## 이번 주 목표

21주차 설계대로 CloudTrail 탐지, Python 요약, Splunk 시각화를 구현하고 누구나 재현할 수 있는 통합 프로젝트 1개를 완성합니다. 10분 라이브 시연이 가능해야 합니다.

## 주야비비 8일 실행표

| 일차 | 근무 | 시간 | 할 일 | 결과물 |
|---|---:|---:|---|---|
| 1 | 주간 | 1시간 | 폴더·입력·테스트 뼈대 | 프로젝트 구조 |
| 2 | 야간 | 30분 | 성공 기준 복원 | 카드 |
| 3 | 비번 | 3시간 | Python 파서·요약 구현 | CLI 초안 |
| 4 | 비번 | 7시간 | Splunk 탐지·대시보드 | 화면 초안 |
| 5 | 주간 | 1시간 | 두 결과 건수 비교 | 검증표 |
| 6 | 야간 | 30분 | 10분 데모 순서 복원 | 발표 카드 |
| 7 | 비번 | 3시간 | 테스트·오류 처리·캡처 | QA |
| 8 | 비번 | 7시간 | README·리허설·완성 | 통합 프로젝트 |

## 구현 순서

1. 샘플 NDJSON 5건을 프로젝트 `data/`에 복사합니다.
2. Python에서 한 줄씩 JSON을 읽는 함수를 만듭니다.
3. actor, target, event, IP, region, time을 정규화합니다.
4. 세 위험 규칙을 순수 함수로 구현합니다.
5. 예상 결과 테스트를 먼저 고정합니다.
6. Splunk에서 같은 규칙을 SPL로 작성합니다.
7. Python과 Splunk 결과를 표로 비교합니다.
8. 패널을 만들고 README와 데모 대본을 완성합니다.

## Python 출력 예시

```json
{
  "total_events": 5,
  "high_risk_events": 3,
  "timeline": [
    {
      "actor": "lab-admin",
      "event": "AttachUserPolicy",
      "target": "temp-support",
      "severity": "high"
    }
  ]
}
```

심각도는 학습 프로젝트의 분류 기준임을 명시하고 AWS 공식 심각도로 표현하지 않습니다.

## Splunk 패널

### 위험 이벤트 수

```spl
index=cloud_security_lab sourcetype="aws:cloudtrail" eventName IN (ConsoleLogin,AttachUserPolicy,StopLogging)
| stats count as reviewed_events
```

### 주체별 API 활동

```spl
index=cloud_security_lab sourcetype="aws:cloudtrail"
| eval actor=coalesce('userIdentity.userName','userIdentity.arn')
| chart count over actor by eventName
```

### IAM 변경 타임라인

```spl
index=cloud_security_lab sourcetype="aws:cloudtrail" eventName IN (CreateUser,AttachUserPolicy,StopLogging)
| eval actor=coalesce('userIdentity.userName','userIdentity.arn')
| eval target=coalesce('requestParameters.userName','requestParameters.name')
| table _time actor sourceIPAddress eventName target requestParameters.policyArn
| sort _time
```

## 교차 검증 표

| 항목 | Python | Splunk | 일치 여부 |
|---|---:|---:|---|
| 전체 이벤트 | 5 | 5 | |
| ConsoleLogin | 1 | 1 | |
| CreateUser | 1 | 1 | |
| AttachUserPolicy | 1 | 1 | |
| StopLogging | 1 | 1 | |
| GetObject | 1 | 1 | |

## 10분 라이브 시연

1. 1분: 문제와 한 줄 목표
2. 1분: 데이터 흐름과 안전한 합성 데이터
3. 2분: Python CLI 실행과 결과
4. 3분: Splunk 대시보드와 원본 피벗
5. 1분: 두 결과의 교차 검증
6. 1분: 테스트와 오류 처리
7. 1분: 제한사항과 다음 개선

## 최종 QA

- 새 폴더에서 README 순서대로 실행합니다.
- 하드코딩된 개인 PC 절대경로를 제거합니다.
- 출력에 비밀번호·토큰·실제 회사 정보가 없는지 확인합니다.
- 테스트가 입력 파일을 수정하지 않는지 확인합니다.
- 캡처에 브라우저 즐겨찾기·계정명이 노출되지 않게 합니다.
- 실패 사례와 제한사항을 숨기지 않습니다.

## 통과 시험

1. 전체 프로젝트를 10분 안에 시연한다.
2. Python과 Splunk의 역할을 구분한다.
3. 동일 건수를 두 방식으로 재현한다.
4. 빈 파일·잘못된 JSON을 안전하게 처리한다.
5. 제한사항과 운영 확장 방안을 설명한다.

## 최종 체크리스트

- [ ] 통합 프로젝트 1개를 완성했다.
- [ ] 탐지·자동화·시각화가 연결된다.
- [ ] 테스트와 README가 있다.
- [ ] 화면 캡처와 예상 출력이 있다.
- [ ] 10분 라이브 시연을 두 번 이상 연습했다.

