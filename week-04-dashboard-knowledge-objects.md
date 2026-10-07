# 4주차 — 대시보드와 지식 객체

## 이번 주 목표

1–3주차 SPL을 재사용 가능한 저장 검색·룩업·대시보드·알림으로 바꿉니다. 최종 산출물은 SPL 20개 목록과 인증 모니터링 대시보드 1개입니다.

## 핵심 개념

| 객체 | 목적 | 이번 주 결과 |
|---|---|---|
| Lookup | IP·계정 같은 값을 설명 정보와 결합 | 실습용 IP 분류표 |
| Saved Search | 검증된 SPL 재사용 | 핵심 검색 5개 저장 |
| Dashboard | 질문별 현황을 한 화면에 표시 | 인증 모니터링 1개 |
| Alert | 조건 충족 시 조사 시작 | 반복 실패 알림 1개 |

## 주야비비 8일 실행표

| 일차 | 근무 | 시간 | 할 일 | 결과물 |
|---|---:|---:|---|---|
| 1 | 주간 | 1시간 | 기존 SPL 20개 후보 정리 | 검색 목록 |
| 2 | 야간 | 30분 | 패널 질문 4개 작성 | 설계 메모 |
| 3 | 비번 | 3시간 | CSV lookup 생성·적용 | 룩업 1개 |
| 4 | 비번 | 7시간 | 저장 검색과 패널 구성 | 대시보드 초안 |
| 5 | 주간 | 1시간 | 반복 실패 기준 정의 | 알림 명세 |
| 6 | 야간 | 30분 | 5분 시연 순서 복원 | 발표 카드 |
| 7 | 비번 | 3시간 | 알림 실행·오탐 검토 | 테스트 기록 |
| 8 | 비번 | 7시간 | 문서화와 시연 연습 | 완성본 |

## 1. SPL 20개 목록 만들기

1–3주차의 검색을 다음 범주로 정리합니다.

- 범위·필터 5개
- 집계·시각화 5개
- 필드 가공·추출 5개
- 인증·프로세스 조사 5개

각 검색에 `이름 / 질문 / SPL / 예상 결과 / 검증법`을 기록합니다. 숫자를 채우기 위한 유사 검색 복제는 하지 않습니다.

## 2. Lookup 실습

다음 내용을 `ip_context.csv`로 직접 만듭니다.

```csv
ip,classification,owner,note
198.51.100.24,external,training,failed-logon-source
203.0.113.45,external,training,powershell-destination
192.0.2.50,external,training,browser-destination
```

Splunk의 **Settings → Lookups → Lookup table files**에서 업로드한 뒤 정의 이름을 `ip_context`로 만듭니다.

```spl
index=security_lab sourcetype="soc:windows" DestinationIp!=""
| lookup ip_context ip as DestinationIp OUTPUT classification owner note
| table _time extracted_host user EventCode DestinationIp DestinationPort classification note
```

매칭되지 않은 값이 비어 있는 것은 정상입니다. Lookup의 키 필드와 검색 필드의 값이 정확히 같은지 확인합니다.

## 3. 인증 대시보드 패널

### 전체 로그인 실패

```spl
index=security_lab sourcetype="soc:windows" EventCode=4625
| stats count as failures
```

### 사용자·IP별 반복 실패

```spl
index=security_lab sourcetype="soc:windows" EventCode=4625
| stats count min(_time) as first_seen max(_time) as last_seen by user DestinationIp extracted_host
| where count>=3
| convert ctime(first_seen) ctime(last_seen)
```

### 단말별 성공·실패

```spl
index=security_lab sourcetype="soc:windows" EventCode IN (4624,4625)
| chart count over extracted_host by Outcome
```

### 시간대별 인증 추세

```spl
index=security_lab sourcetype="soc:windows" EventCode IN (4624,4625)
| timechart span=5m count by Outcome
```

패널마다 제목 아래에 `무엇을 보면 되는지` 한 문장을 설명으로 적습니다.

## 4. 반복 실패 알림

```spl
index=security_lab sourcetype="soc:windows" EventCode=4625
| bin _time span=5m
| stats count values(extracted_host) as hosts by _time user DestinationIp
| where count>=3
```

실습에서는 고정 데이터이므로 All time 검색으로 검증합니다. 운영 설계 문서에는 `최근 5분을 5분마다 실행`, 결과 1개 이상일 때 발생으로 적습니다. 실제 메일·외부 전송은 설정하지 않습니다.

알림에 포함할 필드는 `_time`, `user`, `DestinationIp`, `hosts`, `count`입니다.

## 5분 시연 순서

1. 데이터 소스와 시간 범위를 30초 안에 설명합니다.
2. 대시보드의 네 패널 질문을 설명합니다.
3. 반복 실패 5건을 클릭해 원본 이벤트로 이동합니다.
4. Lookup이 IP 문맥을 추가하는 모습을 보여줍니다.
5. 알림 임계치와 예상 오탐을 설명합니다.

## 통과 시험

1. 저장 검색과 알림의 차이를 설명한다.
2. Lookup 키가 맞지 않을 때 원인을 찾는다.
3. 대시보드를 5분 안에 시연한다.
4. `count>=3`, 5분 구간의 근거와 한계를 말한다.
5. 원시 이벤트로 드릴다운해 질문에 답한다.

## 최종 체크리스트

- [ ] 서로 다른 목적의 SPL 20개를 정리했다.
- [ ] Lookup 1개와 저장 검색 5개를 만들었다.
- [ ] 인증 대시보드에 패널 4개 이상을 만들었다.
- [ ] 반복 실패 알림과 필수 증거 필드를 정의했다.
- [ ] 5분 시연 노트를 작성했다.

