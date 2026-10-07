# 3주차 — 필드 가공과 추출

## 이번 주 목표

기존 필드를 조합하고, 원문과 JSON에서 새 필드를 추출해 검색에 사용할 수 있게 합니다.

```text
필드 존재 확인 → eval로 가공 → where로 필터 → rex/spath로 추출 → 원문 검증
```

완료 결과물은 `eval·where·rex·spath·dedup`을 사용한 SPL 5개와 `rex` 예제 3개입니다.

## 시작 전 확인

시간 범위를 **All time**으로 두고 아래 건수를 확인합니다.

```spl
index=security_lab sourcetype="soc:windows" | stats count
```

예상은 16건입니다. CloudTrail JSON 실습은 다음 검색에서 시작합니다.

```spl
index=cloud_security_lab sourcetype="aws:cloudtrail" | stats count
```

예상은 5건입니다.

## 핵심 명령

| 명령 | 역할 | 기억할 점 |
|---|---|---|
| `eval` | 새 필드를 계산하거나 기존 값을 바꿈 | 결과를 만드는 명령 |
| `where` | 계산식으로 이벤트를 필터링 | 숫자 비교·함수 사용에 강함 |
| `rex` | 정규식 캡처 그룹으로 필드 추출 | `(?<field_name>...)` 형식 |
| `spath` | JSON 경로에서 값 추출 | JSON 원문과 경로를 먼저 확인 |
| `dedup` | 지정 필드 기준 중복 제거 | 정렬 순서에 따라 남는 행이 달라짐 |

## 주야비비 8일 실행표

| 일차 | 근무 | 시간 | 할 일 | 결과물 |
|---|---:|---:|---|---|
| 1 | 주간 | 1시간 | `eval`, `where` 기본 예제 | 가공 SPL 2개 |
| 2 | 야간 | 30분 | 함수와 검색 조건 차이 복원 | 오답 카드 |
| 3 | 비번 | 3시간 | `rex` 캡처 3개 따라 하기 | rex 예제 3개 |
| 4 | 비번 | 7시간 | Windows 명령줄 독립 실습 | SPL 5개 초안 |
| 5 | 주간 | 1시간 | CloudTrail `spath` 실습 | JSON 경로 노트 |
| 6 | 야간 | 30분 | 정규식 캡처 이름 없이 다시 작성 | 암기 점검 |
| 7 | 비번 | 3시간 | `dedup` 전후 건수·행 비교 | 검증 표 |
| 8 | 비번 | 7시간 | 결과물 정리와 통과 시험 | SPL 5개·rex 3개 |

## 단계별 실습

### 1. EventCode에 의미 붙이기

```spl
index=security_lab sourcetype="soc:windows"
| eval event_name=case(EventCode=4624,"logon_success", EventCode=4625,"logon_failure", EventCode=1 OR EventCode=4688,"process_create", EventCode=3,"network_connect", EventCode=22,"dns_query", true(),"other")
| table _time extracted_host user EventCode event_name
```

### 2. 계산된 값으로 필터링

```spl
index=security_lab sourcetype="soc:windows"
| eval is_auth=if(EventCode=4624 OR EventCode=4625,1,0)
| where is_auth=1
| stats count by EventCode Outcome
```

예상은 성공 3건, 실패 5건입니다.

### 3. PowerShell 인코딩 옵션 추출

```spl
index=security_lab sourcetype="soc:windows" CommandLine="*EncodedCommand*"
| rex field=CommandLine "(?i)(?:-enc|-encodedcommand)\s+(?<encoded_payload>\S+)"
| table _time extracted_host user CommandLine encoded_payload
```

예상 `encoded_payload`는 `VEhJU19JU19MQUJfREFUQQ==`입니다.

### 4. 명령줄에서 URL 추출

```spl
index=security_lab sourcetype="soc:windows" CommandLine="*http*"
| rex field=CommandLine "(?<url>https?://[^\s]+)"
| table _time extracted_host Image CommandLine url
```

예상 URL은 `https://training.example.test/demo.hta`입니다.

### 5. 실행 파일명 추출

```spl
index=security_lab sourcetype="soc:windows" Image!=""
| rex field=Image "(?<image_name>[^\\]+)$"
| stats count values(CommandLine) as commands by image_name
| sort - count
```

### 6. JSON 경로 확인

```spl
index=cloud_security_lab sourcetype="aws:cloudtrail"
| spath
| eval actor=coalesce('userIdentity.userName','userIdentity.arn')
| table _time actor eventName sourceIPAddress requestParameters.userName requestParameters.policyArn
```

JSON 필드가 이미 검색 시점에 추출되어 있어도 `spath`를 명시해 원리와 경로 표기법을 익힙니다.

### 7. 중복 제거의 영향 확인

```spl
index=security_lab sourcetype="soc:windows"
| sort - _time
| dedup extracted_host
| table _time extracted_host user EventCode Outcome
```

`dedup` 전 16건, 단말 기준 중복 제거 후 5건입니다. 최신 행을 남기고 싶다면 반드시 먼저 최신순으로 정렬합니다.

## 필드가 없을 때 점검 순서

1. 이벤트가 실제로 검색되는지 확인합니다.
2. `_raw`에 찾으려는 문자열이 있는지 확인합니다.
3. 원본 필드명과 대소문자를 확인합니다.
4. `rex`를 짧은 패턴부터 늘립니다.
5. `table _raw 새필드`로 추출 결과를 나란히 봅니다.
6. JSON이면 `| spath | table *`로 실제 경로를 확인합니다.

## 통과 시험

1. `eval`과 `where`의 역할을 구분해 설명한다.
2. 보지 않고 PowerShell payload와 URL을 추출한다.
3. CloudTrail의 사용자 이름과 정책 ARN을 표로 만든다.
4. `dedup` 전에 정렬이 필요한 이유를 설명한다.
5. 필드가 비었을 때 `_raw`부터 원인을 찾아 수정한다.

## 최종 체크리스트

- [ ] 가공 SPL 5개를 저장했다.
- [ ] 이름 있는 캡처 그룹을 사용한 `rex` 3개를 만들었다.
- [ ] `spath`로 중첩 JSON 필드를 확인했다.
- [ ] 각 새 필드를 `_raw`와 비교했다.
- [ ] `dedup` 전후 건수를 기록했다.

