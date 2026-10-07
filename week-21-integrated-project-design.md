# 21주차 — 통합 프로젝트 설계

## 이번 주 목표

Splunk + Python + AWS 로그를 하나의 문제 해결 흐름으로 연결하는 프로젝트를 설계합니다. 코드를 먼저 만들지 않고 문제, 사용자, 데이터 흐름, 성공 기준, 제한사항을 확정합니다.

## 권장 프로젝트

**프로젝트명:** CloudTrail 보안 이벤트 탐지·요약 대시보드

**한 줄 목표:** 합성 CloudTrail 로그에서 고위험 IAM·로깅 이벤트를 Splunk로 탐지하고 Python으로 재현 가능한 요약 보고서를 생성한다.

## 주야비비 8일 실행표

| 일차 | 근무 | 시간 | 할 일 | 결과물 |
|---|---:|---:|---|---|
| 1 | 주간 | 1시간 | 문제·사용자 정의 | 한 줄 목표 |
| 2 | 야간 | 30분 | 입력·출력 복원 | 카드 |
| 3 | 비번 | 3시간 | 데이터 흐름과 폴더 설계 | 구조도 |
| 4 | 비번 | 7시간 | 탐지·CLI·대시보드 요구사항 | 설계서 초안 |
| 5 | 주간 | 1시간 | 성공·실패 기준 | 검증표 |
| 6 | 야간 | 30분 | 1분 설명 연습 | 발표 카드 |
| 7 | 비번 | 3시간 | 샘플 입력·예상 출력 고정 | 테스트 자료 |
| 8 | 비번 | 7시간 | 설계 리뷰·범위 축소 | 최종 설계서 |

## 문제 정의

```text
대상 사용자: 보안관제 분석가
문제: CloudTrail 원본 JSON만으로는 고위험 IAM 변경과 로깅 중지 흐름을 빠르게 파악하기 어렵다.
입력: 회사 정보가 없는 CloudTrail 합성 로그 5건
처리: Splunk 탐지·타임라인 + Python 요약·검증
출력: 대시보드, 탐지 3개 이상, JSON/Markdown 요약 보고서
성공: 새 PC에서 README만으로 재현하고 10분 안에 시연
```

## 데이터 흐름

```text
CloudTrail NDJSON
├─ Splunk cloud_security_lab
│  ├─ IAM 고권한 변경 탐지
│  ├─ StopLogging 탐지
│  └─ 대시보드·원본 피벗
└─ Python cloudtrail_summary.py
   ├─ actor/target 정규화
   ├─ 위험 이벤트 분류
   └─ JSON 또는 Markdown 보고서
```

## 권장 폴더 구조

```text
portfolio/cloudtrail-security-project/
├─ README.md
├─ data/
│  └─ cloudtrail_events.ndjson
├─ splunk/
│  ├─ detections.spl
│  └─ dashboard-notes.md
├─ python/
│  ├─ cloudtrail_summary.py
│  └─ test_cloudtrail_summary.py
├─ reports/
│  └─ expected-summary.md
└─ docs/
   ├─ architecture.md
   ├─ limitations.md
   └─ demo-script.md
```

## 기능 범위

### 반드시 구현

- NDJSON 5건 읽기
- actor·target·IP·리전·API·시각 정규화
- ConsoleLogin 실패, 고권한 정책 연결, StopLogging 분류
- 시간순 타임라인
- Splunk 패널 3개 이상
- Python 결과와 Splunk 건수 교차 검증
- 단위 테스트와 README

### 이번 프로젝트에서 제외

- 실제 AWS 계정 연결
- 자동 차단·IAM 변경
- 외부 위협정보 API
- 운영용 실시간 수집
- 회사 로그·고객 정보

범위를 명확히 줄이는 것도 설계 능력입니다.

## 성공 기준

| ID | 기준 | 검증 방법 |
|---|---|---|
| S1 | 입력 5건을 모두 읽음 | Python/Splunk count 비교 |
| S2 | 세 고위험 행위를 식별 | 예상 eventName 비교 |
| S3 | 시간순 타임라인 생성 | 첫·마지막 시각 확인 |
| S4 | actor와 target 구분 | lab-admin/temp-support 비교 |
| S5 | 오류 입력을 안전 처리 | 빈 파일·잘못된 JSON 테스트 |
| S6 | 새 PC 재현 가능 | README 절차만 실행 |
| S7 | 10분 시연 가능 | 데모 리허설 |

## 설계서 필수 항목

1. 한 줄 목표와 대상 사용자
2. 문제와 기존 방식의 불편
3. 입력·처리·출력
4. 아키텍처와 폴더 구조
5. 탐지 요구사항
6. Python 요구사항
7. 테스트와 성공 기준
8. 보안·개인정보 원칙
9. 제한사항과 제외 범위
10. 22주차 작업 순서

## 통과 시험

1. 프로젝트 목표를 한 문장으로 설명한다.
2. 데이터 흐름을 1분 안에 그린다.
3. Splunk와 Python을 함께 쓰는 이유를 설명한다.
4. 성공 기준을 숫자와 테스트로 제시한다.
5. 제외 범위를 말하고 기능 확장을 거절할 수 있다.

## 최종 체크리스트

- [ ] 프로젝트 설계서를 완성했다.
- [ ] 폴더·샘플 입력·예상 출력을 정의했다.
- [ ] 기능 범위와 제외 범위를 분리했다.
- [ ] 성공 기준 7개를 검증 가능하게 작성했다.
- [ ] 22주차 첫 작업을 30분 단위로 적었다.

