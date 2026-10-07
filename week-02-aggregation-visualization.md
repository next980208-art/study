# 2주차 — Splunk 집계와 시각화

## 이번 주 목표

원시 이벤트를 한 줄씩 보는 단계에서 벗어나, 여러 이벤트를 의미 있는 숫자와 추세로 요약하는 것이 목표입니다.

```text
질문 정의
→ 데이터 범위 지정(index, sourcetype, 시간)
→ 필요한 이벤트만 필터링
→ 집계(stats, chart, timechart, top, rare)
→ 원시 이벤트로 결과 검증
→ 질문에 맞는 시각화 선택
```

완료 결과물은 다음 3개입니다.

1. 직접 작성하고 설명할 수 있는 집계 SPL 5개
2. 단일 값·막대 차트·시간 추세 차트로 구성한 패널 3개
3. 각 패널의 질문·필터·집계 기준·검증 방법을 적은 분석 노트

## 시작 전 10분

1. `splunk` 폴더에서 `docker compose up -d`를 실행합니다.
2. `http://127.0.0.1:8000`에서 `admin`으로 로그인합니다.
3. **Search & Reporting** 앱을 엽니다.
4. 시간 범위를 **All time(전체 시간)** 으로 바꿉니다.
5. 아래 확인 검색을 실행합니다.

```spl
(index=security_lab sourcetype IN ("soc:windows","soc:edr"))
OR (index=cloud_security_lab sourcetype="aws:cloudtrail")
| stats count by index sourcetype
```

예상 결과는 `soc:windows=16`, `soc:edr=7`, `aws:cloudtrail=5`입니다. 이번 주의 Windows 집계는 주로 `security_lab`의 `soc:windows` 16건을 사용합니다.

## 먼저 이해할 핵심

### 집계 전과 집계 후

```spl
index=security_lab sourcetype="soc:windows"
| stats count by EventCode
```

- 파이프 앞에는 원본 이벤트 16건이 있습니다.
- `stats`가 같은 `EventCode`끼리 이벤트를 묶습니다.
- 파이프 뒤에는 `EventCode`와 `count`만 남습니다.
- `Image`, `user`, `_raw`처럼 집계에 사용하지 않은 필드는 결과에서 사라집니다.

`stats` 결과에서 원래 필드가 보이지 않는 것은 오류가 아닙니다. 집계 결과만 남기는 변환 명령의 정상 동작입니다. 원본을 다시 보려면 `stats` 앞까지만 실행하거나, 집계 결과의 조건을 원래 검색에 다시 넣어 확인합니다.

### 이번 주 명령어 지도

| 명령어 | 언제 쓰는가 | 결과 모양 | 대표 질문 |
|---|---|---|---|
| `stats` | 자유롭게 통계를 계산할 때 | 일반 표 | EventCode별 이벤트는 몇 건인가? |
| `chart` | 행과 열로 범주를 비교할 때 | 교차표 | 단말별 성공·실패 건수는? |
| `timechart` | 시간에 따른 변화를 볼 때 | 시간표 | 시간대별 이벤트 추세는? |
| `top` | 가장 많이 나온 값을 볼 때 | 빈도·비율 표 | 가장 빈번한 EventCode는? |
| `rare` | 가장 적게 나온 값을 볼 때 | 빈도·비율 표 | 드물게 나온 EventCode는? |
| `sort` | 결과 순서를 정할 때 | 정렬된 표 | 건수가 큰 순서로 보면? |

### 자주 쓰는 집계 함수

| 함수 | 의미 | 예시 |
|---|---|---|
| `count` | 이벤트 수 | `stats count by EventCode` |
| `dc(field)` | 중복을 제거한 값의 개수 | `dc(user) as distinct_users` |
| `values(field)` | 중복을 제거한 값 목록 | `values(user) as users` |
| `sum(field)` | 숫자 필드의 합계 | `sum(bytes) as total_bytes` |
| `avg(field)` | 숫자 필드의 평균 | `avg(duration) as avg_duration` |
| `min(field)` / `max(field)` | 최솟값 / 최댓값 | `min(_time)`, `max(_time)` |
| `earliest(field)` / `latest(field)` | 시간순 최초 / 최신 이벤트의 값 | `earliest(user)`, `latest(user)` |

`as`는 출력 열의 이름을 읽기 쉽게 바꾸는 별칭입니다. 예를 들어 `dc(user) as distinct_users`는 고유 사용자 수를 `distinct_users`라는 열로 보여줍니다.

## 주야비비 8일 실행표

| 일차 | 근무 | 시간 | 해야 할 일 | 그날 남길 것 |
|---|---:|---:|---|---|
| 1일 | 주간 | 1시간 | 데이터 건수 확인, `stats count by` 구조 익히기 | 집계 전·후 차이 3문장 |
| 2일 | 야간 | 30분 | `count`, `dc`, `values`, `as`, `by`를 보지 않고 설명 | 함수 암기 카드 5개 |
| 3일 | 비번·회복 | 3시간 | 단계별 필수 검색 1–4번 실행, 예상 건수와 비교 | 결과 건수 표 |
| 4일 | 비번·집중 | 7시간 | `stats`, `chart`, `top`, `rare` 비교 실습 | 집계 SPL 5개 초안 |
| 5일 | 주간 | 1시간 | `timechart span=1h`, 시간 버킷 개념 익히기 | 시간 집계 설명 1장 |
| 6일 | 야간 | 30분 | 빈 검색창에서 집계 SPL 3개 복원 | 막힌 문법 1개 |
| 7일 | 비번·회복 | 3시간 | 패널 3개 생성하고 시각화 종류 비교 | 패널 화면 캡처 3장 |
| 8일 | 비번·집중 | 7시간 | 결과물 정리, 원시 이벤트 검증, 통과 시험 | SPL 5개 + 패널 3개 + 분석 노트 |

## 단계별 필수 검색

### 1. 전체 이벤트 수를 하나의 숫자로 집계

```spl
index=security_lab sourcetype="soc:windows"
| stats count as total_events
```

예상: `total_events=16`입니다. `stats count`만 사용하면 전체 이벤트 수가 한 행으로 나옵니다.

검증할 때는 `| stats ...`를 지우고 검색 결과 상단의 이벤트 건수가 16인지 확인합니다.

### 2. EventCode별 건수 집계

```spl
index=security_lab sourcetype="soc:windows"
| stats count by EventCode
| sort - count
```

예상 결과:

| EventCode | count | 의미 |
|---:|---:|---|
| 4625 | 5 | 로그인 실패 |
| 1 | 4 | 프로세스 생성 샘플 |
| 4624 | 3 | 로그인 성공 |
| 3 | 2 | 네트워크 연결 샘플 |
| 22 | 1 | DNS 질의 샘플 |
| 4688 | 1 | Windows 프로세스 생성 |

`sort - count`의 `-`는 내림차순입니다. `sort count`는 오름차순입니다.

### 3. 단말별 이벤트와 고유 사용자 집계

```spl
index=security_lab sourcetype="soc:windows"
| stats count as events dc(user) as distinct_users values(user) as users by extracted_host
| sort - events
```

예상 결과:

| extracted_host | events | distinct_users | users |
|---|---:|---:|---|
| LAB-DC01 | 5 | 1 | test-admin |
| LAB-WS01 | 5 | 1 | kim |
| LAB-WS02 | 3 | 1 | lee |
| LAB-WS03 | 2 | 1 | park |
| LAB-WS04 | 1 | 1 | choi |

`dc(user)`는 사용자 종류의 수, `values(user)`는 실제 사용자 값의 목록입니다. 둘은 질문이 다릅니다.

### 4. 로그인 성공·실패를 단말별 교차표로 비교

```spl
index=security_lab sourcetype="soc:windows" EventCode IN (4624,4625)
| chart count over extracted_host by Outcome
```

예상 결과:

| extracted_host | failure | success |
|---|---:|---:|
| LAB-DC01 | 5 | 0 |
| LAB-WS01 | 0 | 1 |
| LAB-WS02 | 0 | 1 |
| LAB-WS04 | 0 | 1 |

`over extracted_host`는 표의 행, `by Outcome`은 열을 만듭니다. 없는 조합은 화면 설정이나 명령 결과에 따라 빈칸 또는 0으로 보일 수 있습니다.

### 5. 조건별 건수를 한 행으로 만들기

```spl
index=security_lab sourcetype="soc:windows" EventCode IN (4624,4625)
| stats count(eval(EventCode=4624)) as success_count
        count(eval(EventCode=4625)) as failure_count
```

예상: `success_count=3`, `failure_count=5`입니다.

`count(eval(조건))`은 해당 조건을 만족한 이벤트만 셉니다. 검색 조건에서 먼저 로그인 이벤트만 남겼는지, `eval` 안에서 어떤 EventCode를 세는지 각각 설명할 수 있어야 합니다.

### 6. 가장 자주 등장한 값 보기

```spl
index=security_lab sourcetype="soc:windows"
| top limit=5 EventCode
```

`top`은 기본적으로 `EventCode`, `count`, `percent`를 보여줍니다. 이 데이터에서는 4625가 5건으로 가장 많습니다.

`percent`는 전체 16건 중 해당 값이 차지하는 비율입니다. 작은 표본이므로 비율만 보고 위험도를 판단하지 말고 실제 이벤트도 함께 봅니다.

### 7. 드물게 등장한 값 보기

```spl
index=security_lab sourcetype="soc:windows"
| rare limit=3 EventCode
```

`rare`는 빈도가 낮은 값을 먼저 보여줍니다. 이 데이터에는 1건짜리 EventCode가 두 개 있으므로 22와 4688이 희귀 값에 포함됩니다. 동률 값의 표시 순서는 달라질 수 있습니다.

희귀하다는 이유만으로 악성은 아닙니다. 정상적으로 드문 이벤트인지, 탐지 가치가 있는 이벤트인지 원시 로그와 주변 이벤트로 판단합니다.

### 8. 시간대별 이벤트 추세 보기

```spl
index=security_lab sourcetype="soc:windows"
| timechart span=1h count by Outcome
```

`timechart`는 `_time`을 기준으로 자동 집계합니다. `span=1h`는 이벤트를 1시간 단위 바구니에 넣는다는 뜻입니다. 이 실습 데이터는 2026-09-01 하루에 있으므로 반드시 시간 범위를 **All time**으로 둡니다.

다음 검색으로 시간 버킷의 원리를 표 형태로 비교합니다.

```spl
index=security_lab sourcetype="soc:windows"
| bin _time span=1h
| stats count by _time Outcome
| sort _time Outcome
```

`timechart span=1h count by Outcome`은 위의 `bin + stats` 조합을 시간 시각화에 적합한 형태로 만들어 주는 명령이라고 이해하면 됩니다.

### 9. 집계 결과에서 이상 후보 찾고 원본으로 돌아가기

1단계 — 어떤 사용자에게 로그인 실패가 집중되었는지 집계합니다.

```spl
index=security_lab sourcetype="soc:windows" EventCode=4625
| stats count min(_time) as first_seen max(_time) as last_seen by user DestinationIp extracted_host
| where count >= 3
| convert ctime(first_seen) ctime(last_seen)
```

예상: `test-admin`, `198.51.100.24`, `LAB-DC01`, `count=5`인 한 행이 나옵니다.

2단계 — 찾은 값으로 원시 이벤트를 다시 검색합니다.

```spl
index=security_lab sourcetype="soc:windows"
EventCode=4625 user="test-admin" DestinationIp="198.51.100.24" extracted_host="LAB-DC01"
| table _time extracted_host user EventCode DestinationIp DestinationPort Outcome _raw
| sort _time
```

예상: 약 12초 사이에 발생한 로그인 실패 5건입니다. 이 과정이 **집계 → 이상 후보 발견 → 원본 검증**입니다.

## 패널 3개 만들기

### 패널 1 — 로그인 실패 건수

질문: 전체 실습 데이터에서 로그인 실패는 몇 건인가?

```spl
index=security_lab sourcetype="soc:windows" EventCode=4625
| stats count as login_failures
```

- 예상 값: `5`
- 시각화: **Single Value**
- 패널 제목: `로그인 실패 건수`
- 검증: `stats`를 제거했을 때 원시 이벤트가 5건인지 확인

### 패널 2 — EventCode별 이벤트 분포

질문: 어떤 EventCode가 가장 많이 발생했는가?

```spl
index=security_lab sourcetype="soc:windows"
| stats count by EventCode
| sort - count
```

- 시각화: **Column Chart** 또는 **Bar Chart**
- 패널 제목: `EventCode별 이벤트 분포`
- X축: `EventCode`
- Y축: `count`
- 검증: 모든 막대의 합이 16인지 확인

### 패널 3 — 시간대별 결과 추세

질문: 이벤트 결과가 어느 시간대에 집중되었는가?

```spl
index=security_lab sourcetype="soc:windows"
| timechart span=1h count by Outcome
```

- 시각화: **Line Chart**
- 패널 제목: `시간대별 이벤트 결과 추세`
- X축: `_time`
- 계열: `Outcome`
- 검증: 각 시간대와 결과 값을 클릭해 원시 이벤트를 확인

### Splunk 화면에서 저장하는 순서

1. Search & Reporting에서 SPL을 실행합니다.
2. **Visualization** 탭을 열고 지정된 시각화를 선택합니다.
3. 축·범례·제목이 질문을 이해하는 데 필요한지 확인합니다.
4. **Save As → Dashboard Panel**을 선택합니다.
5. 새 대시보드 이름을 `SOC 학습 - 2주차`로 지정합니다.
6. 나머지 두 검색도 같은 대시보드에 추가합니다.
7. 패널 제목만 읽어도 무엇을 보여주는지 알 수 있는지 확인합니다.

## 집계 SPL 5개 결과물 형식

아래 다섯 주제를 직접 작성합니다.

1. EventCode별 이벤트 수
2. 단말별 이벤트 수와 고유 사용자 수
3. 단말별 로그인 성공·실패 교차표
4. 시간대별 Outcome 추세
5. 사용자·출발지 IP별 반복 로그인 실패

각 검색은 아래 형식으로 기록합니다.

```text
검색 이름:
질문: 이 숫자로 무엇을 판단하려는가?
검색 대상: index / sourcetype / 시간 범위
필터 조건:
SPL:
그룹 기준(by / over):
집계 함수:
예상 결과:
실제 결과:
원시 이벤트 검증 방법:
적합한 시각화와 선택 이유:
한 문장 해석:
```

좋은 해석은 숫자를 반복하지 않고 의미를 말합니다.

```text
나쁜 예: test-admin의 실패는 5건이다.
좋은 예: 전체 로그인 실패 5건이 test-admin과 단일 출발지 IP에 집중되어 있어 반복 인증 시도로 우선 조사한다.
```

## 통과 시험

아래 여덟 항목을 모두 할 수 있으면 2주차 완료입니다.

1. `stats count`와 `stats count by EventCode`의 결과 차이를 설명한다.
2. `count`, `dc`, `values`의 차이를 예시와 함께 설명한다.
3. 빈 검색창에서 EventCode별 정확한 건수를 만든다.
4. `chart ... over ... by ...`에서 행과 열이 무엇인지 설명한다.
5. `timechart span=1h`가 이벤트를 어떻게 묶는지 설명한다.
6. `top`과 `rare` 결과를 악성 여부와 동일시하면 안 되는 이유를 설명한다.
7. 집계 결과에서 `test-admin`의 실패 5건을 찾고 원본 이벤트까지 돌아간다.
8. 패널 3개를 화면 없이 다시 만들고 각 시각화를 선택한 이유를 말한다.

## 자주 막히는 원인

- **시간 차트가 비어 있음:** 고정 실습 데이터는 2026-09-01 이벤트입니다. 시간 범위를 All time으로 바꿉니다.
- **`stats` 뒤에 원래 필드가 사라짐:** 집계 함수 또는 `by`에 넣지 않은 필드는 결과에 남지 않습니다. 원본 검색과 집계 검색을 별도로 비교합니다.
- **건수 합계가 16이 아님:** 검색 조건이 Windows 16건만 대상으로 하는지, 특정 EventCode나 Outcome 필터가 남아 있는지 확인합니다.
- **`chart`의 빈칸:** 해당 단말과 Outcome 조합의 이벤트가 없다는 뜻일 수 있습니다. 원본 검색으로 실제 0건인지 검증합니다.
- **`timechart` 선이 너무 많음:** `by` 뒤 필드의 값 종류가 많으면 계열이 많아집니다. 질문에 필요한 필드만 사용합니다.
- **백분율만 보고 결론을 냄:** 표본 수와 절대 건수를 함께 확인합니다. 50%라도 전체가 2건이면 의미가 다릅니다.
- **대시보드부터 꾸밈:** 먼저 질문과 SPL을 검증하고 마지막에 시각화를 선택합니다.
- **집계 결과를 곧바로 사고로 판단:** 집계는 조사 대상을 줄이는 도구입니다. 원시 이벤트·시간·사용자·IP·프로세스 문맥을 다시 확인합니다.

## 2주차 최종 체크리스트

- [ ] 데이터 확인 검색에서 Windows 16건, EDR 7건, CloudTrail 5건을 확인했다.
- [ ] EventCode별 예상 건수 `5, 4, 3, 2, 1, 1`을 재현했다.
- [ ] `stats`, `chart`, `timechart`, `top`, `rare`를 각각 한 번 이상 직접 작성했다.
- [ ] 집계 SPL 5개를 답안 없이 작성했다.
- [ ] 모든 집계 SPL을 원시 이벤트와 비교해 검증했다.
- [ ] Single Value, Bar/Column, Line 패널을 하나씩 만들었다.
- [ ] 반복 로그인 실패 5건을 집계에서 원본까지 추적했다.
- [ ] 각 패널의 질문과 선택 이유를 한 문장으로 설명했다.

## 공식 참고 자료

- 통계 및 차트 명령 개요: https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/calculate-statistics/about-statistical-and-charting-functions
- `stats` 명령: https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.4/search-commands/stats
- `chart` 명령: https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.4/search-commands/chart
- `timechart` 명령: https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.4/search-commands/timechart
- `top` 명령: https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.4/search-commands/top
- `rare` 명령: https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.4/search-commands/rare
