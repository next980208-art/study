# 1주차 — Splunk 검색 구조

## 이번 주 목표

검색을 많이 복사하는 것이 아니라 아래 구조를 보고 직접 조립하고 설명할 수 있게 되는 것이 목표입니다.

```text
시간 범위 + 데이터 범위(index, sourcetype)
→ 이벤트 조건(field=value, 키워드, AND/OR)
→ 파이프(|) 뒤 처리(table, fields, sort, head)
→ 결과 검증(건수, 원시 이벤트, 필드)
```

완료 결과물은 다음 3개입니다.

1. 직접 작성한 기본 SPL 5개
2. `index`, `sourcetype`, `source`, `host`, `extracted_host`, `_time`, `_raw` 필드 사전
3. 각 SPL의 목적·예상 건수·실제 건수·틀렸을 때 확인할 점을 적은 설명 노트

## 시작 전 10분

1. 작업 폴더의 `Splunk 실습환경 시작.cmd`를 실행합니다.
2. `http://127.0.0.1:8000`에서 `admin`으로 로그인합니다.
3. **Search & Reporting** 앱을 엽니다.
4. 시간 범위를 **All time(전체 시간)** 으로 바꿉니다.
5. 아래 확인 검색을 실행합니다.

```spl
(index=security_lab sourcetype IN ("soc:windows","soc:edr"))
OR (index=cloud_security_lab sourcetype="aws:cloudtrail")
| stats count by index sourcetype
```

예상 결과는 `soc:windows=16`, `soc:edr=7`, `aws:cloudtrail=5`입니다.

## 반드시 구분할 필드

| 필드 | 뜻 | 이 실습에서 볼 값 |
|---|---|---|
| `index` | 이벤트가 저장된 논리적 저장 공간 | `security_lab`, `cloud_security_lab` |
| `sourcetype` | 로그의 형식과 필드 구조 | `soc:windows`, `soc:edr`, `aws:cloudtrail` |
| `source` | 로그가 들어온 파일·경로·입력 | `...windows_events_lab.csv` 등 |
| `host` | Splunk 입력에 지정된 원본 호스트 | 이 실습에서는 `security-lab` |
| `extracted_host` | CSV의 `host` 열에서 추출된 분석 대상 단말 | `LAB-WS01`, `LAB-DC01` 등 |
| `_time` | Splunk가 인식한 이벤트 시각 | 시간순 정렬과 구간 분석에 사용 |
| `_raw` | 가공 전 원본 이벤트 한 줄 | 필드가 이상할 때 반드시 비교 |

`host`와 `extracted_host`가 다른 이유를 설명할 수 있어야 합니다. Splunk의 기본 필드 `host`가 이미 존재하므로 CSV 안의 동명 열은 `extracted_host`로 보존됩니다. 이 실습의 `host`는 Docker 입력 호스트이고 `extracted_host`는 분석 대상 단말입니다.

## 주야비비 8일 실행표

| 일차 | 근무 | 시간 | 해야 할 일 | 그날 남길 것 |
|---|---:|---:|---|---|
| 1일 | 주간 | 1시간 | Splunk 접속, All time 설정, 확인 검색, 화면 구성 익히기 | 기본 필드 4개 메모 |
| 2일 | 야간 | 30분 | `index/source/sourcetype/host`를 보지 않고 설명 | 용어 오답 1장 |
| 3일 | 비번·회복 | 3시간 | 범위 검색 1–5번 실행, 각 단계의 결과 건수 비교 | 검색 결과 표 |
| 4일 | 비번·집중 | 7시간 | 조건식과 파이프, `table/fields/sort/head` 실습 | 기본 SPL 5개 초안 |
| 5일 | 주간 | 1시간 | `AND/OR/IN`, 괄호, 따옴표, 와일드카드 비교 | 조건식 예제 5개 |
| 6일 | 야간 | 30분 | 노트 없이 검색 3개 복원하고 소리 내 설명 | 막힌 지점 1개 |
| 7일 | 비번·회복 | 3시간 | `labs.md` 1–5번을 답안 없이 해결 | 오답과 수정 이유 |
| 8일 | 비번·집중 | 7시간 | 결과물 정리, 통과 시험, 화면 캡처 3장 | SPL 5개 + 필드 사전 |

## 단계별 필수 검색

### 1. 전체 범위 확인

```spl
index=security_lab
```

예상: Windows 16건과 EDR 7건을 합쳐 23건입니다. 결과가 0건이면 먼저 시간 범위와 인덱스 이름을 확인합니다.

### 2. 데이터 형식으로 좁히기

```spl
index=security_lab sourcetype="soc:windows"
```

예상: 16건입니다. `index`는 저장 위치, `sourcetype`은 로그 형식입니다.

### 3. 이벤트 코드로 좁히기

```spl
index=security_lab sourcetype="soc:windows" EventCode=4625
```

예상: 로그인 실패 5건입니다.

### 4. 여러 값을 한 번에 찾기

```spl
index=security_lab sourcetype="soc:windows" EventCode IN (4624,4625)
```

예상: 성공 3건과 실패 5건을 합쳐 8건입니다.

### 5. 단말과 사용자 조건 추가

```spl
index=security_lab sourcetype="soc:windows" extracted_host="LAB-WS01" user="kim"
```

예상: 5건입니다. 조건 사이에는 `AND`가 생략되어 있습니다.

### 6. 필요한 필드만 표로 보기

```spl
index=security_lab sourcetype="soc:windows"
| table _time extracted_host user EventCode Outcome CommandLine
```

`table` 앞은 어떤 이벤트를 가져올지 정하고, 뒤는 결과를 어떻게 보여줄지 정합니다.

### 7. 최신 이벤트 10건만 보기

```spl
index=security_lab sourcetype="soc:windows"
| sort - _time
| head 10
```

`sort - _time`은 최신순, `head 10`은 앞의 10건만 남깁니다.

### 8. 기본 필드와 원본 비교

```spl
index=security_lab sourcetype="soc:windows"
| table _time index source sourcetype host extracted_host _raw
| head 5
```

필드 값이 예상과 다르면 `_raw`를 보고 필드가 원본에서 어떻게 만들어졌는지 확인합니다.

## 기본 SPL 5개 결과물 형식

각 검색을 아래 형식으로 기록합니다.

```text
검색 이름:
질문: 이 검색으로 무엇을 확인하려는가?
SPL:
예상 건수:
실제 건수:
중요 필드:
결과가 0건일 때 확인할 것:
한 문장 설명:
```

추천 주제는 전체 Windows 이벤트, 로그인 실패, 로그인 성공·실패, 특정 단말·사용자, 필요한 필드만 표시하기입니다.

## 통과 시험

아래 여섯 항목을 모두 할 수 있으면 1주차 완료입니다.

1. `index`, `sourcetype`, `source`, `host`의 차이를 예시와 함께 설명한다.
2. 빈 검색창에서 Windows 이벤트 16건을 찾는다.
3. 로그인 실패 5건을 찾고 왜 5건인지 원본 이벤트로 확인한다.
4. `|` 왼쪽과 오른쪽의 역할을 설명한다.
5. 검색 결과가 0건일 때 시간 범위 → 인덱스 → 소스타입 → 필드명 순서로 점검한다.
6. 저장한 SPL 5개를 화면 없이 말로 설명한다.

## 자주 막히는 원인

- **결과 0건:** 시간 범위가 최근 24시간인지 먼저 확인합니다. 고정 데이터 실습은 All time을 사용합니다.
- **필드가 안 보임:** 이벤트를 펼쳐 `_raw`와 Selected Fields를 비교합니다.
- **건수가 예상과 다름:** 검색 조건을 하나씩 빼면서 어느 조건에서 줄었는지 확인합니다.
- **OR 결과가 이상함:** 조건을 괄호로 묶습니다. 예: `(EventCode=4624 OR EventCode=4625) user="kim"`.
- **처음부터 `index=*`:** 실무에서는 검색 비용과 노이즈가 커지므로 알고 있는 인덱스와 소스타입부터 지정합니다.

## 공식 참고 자료

- Search 앱: https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.1/using-the-search-app/about-the-search-app
- 기본 필드와 검색 최적화: https://help.splunk.com/en/splunk-enterprise/search/search-manual/10.4/optimize-searches/quick-tips-for-optimization
- 기본 필드 설명: https://help.splunk.com/en/splunk-enterprise/get-started/get-data-in/9.4/configure-indexed-field-extraction/about-default-fields-host-source-sourcetype-and-more
