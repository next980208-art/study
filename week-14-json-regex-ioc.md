# 14주차 — JSON·정규식·IOC

## 이번 주 목표

`json`, `re`, `ipaddress`, `set`을 사용해 중첩 JSON을 읽고 IP·도메인·URL·이메일·해시를 검증·중복 제거하는 IOC CLI를 완성합니다.

## 기준 파일

- 코드: `study-lab\python\ioc_extractor.py`
- 입력: `study-lab\datasets\ioc_sample.txt`
- 테스트: `study-lab\python\test_ioc_extractor.py`

현재 도구는 IPv4, 도메인, SHA256, MD5를 지원합니다. 이번 주에는 URL과 이메일을 추가합니다.

## 주야비비 8일 실행표

| 일차 | 근무 | 시간 | 할 일 | 결과물 |
|---|---:|---:|---|---|
| 1 | 주간 | 1시간 | JSON 읽기·쓰기 | 예제 |
| 2 | 야간 | 30분 | 정규식 기호 복원 | 카드 |
| 3 | 비번 | 3시간 | IP 후보와 유효성 검증 | 함수 |
| 4 | 비번 | 7시간 | URL·이메일 추출 추가 | CLI 초안 |
| 5 | 주간 | 1시간 | set 중복 제거·정렬 | 검증표 |
| 6 | 야간 | 30분 | 잘못된 IP 테스트 | 점검 |
| 7 | 비번 | 3시간 | 중첩 CloudTrail JSON 읽기 | JSON 노트 |
| 8 | 비번 | 7시간 | 테스트·README·통과 시험 | IOC CLI |

## 현재 도구 실행

```powershell
python python\ioc_extractor.py datasets\ioc_sample.txt
```

예상 핵심 결과:

- IPv4: `192.0.2.50`, `203.0.113.45`
- 도메인: `training.example.test`
- SHA256 1개, MD5 1개
- 잘못된 `999.999.999.999`는 제외

## 정규식과 검증 분리

정규식은 IP처럼 보이는 **후보**를 찾고, `ipaddress.ip_address()`가 실제 유효성을 검사합니다. 정규식 하나로 0–255 범위를 완벽히 해결하려 하지 않습니다.

```python
for candidate in IP_CANDIDATE.findall(text):
    try:
        ips.add(str(ipaddress.ip_address(candidate)))
    except ValueError:
        continue
```

## URL 지원 추가 요구사항

```python
URL = re.compile(r"https?://[^\s<>\"']+")
```

- 문장 끝 마침표와 쉼표를 제거합니다.
- `http`와 `https`만 허용합니다.
- 출력 키 이름은 `urls`로 통일합니다.
- 중복을 제거하고 정렬합니다.

## 이메일 지원 추가 요구사항

```python
EMAIL = re.compile(r"\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,63}\b")
```

- 소문자로 정규화합니다.
- 이메일의 도메인이 별도 도메인 IOC로 중복 집계될지 정책을 정합니다.
- 정규식만으로 실제 메일함 존재를 보장할 수 없음을 README에 적습니다.

## CloudTrail 중첩 JSON 읽기

```python
import json
from pathlib import Path

events = json.loads(Path("cloudtrail_events.json").read_text(encoding="utf-8"))
for event in events:
    actor = event.get("userIdentity", {}).get("userName") or event.get("userIdentity", {}).get("arn")
    print(event.get("eventTime"), actor, event.get("eventName"))
```

실제 파일의 최상위 구조가 리스트인지 딕셔너리인지 먼저 출력해 확인합니다.

## 추가할 테스트

1. URL 2개를 추출하고 중복을 제거한다.
2. 대문자 이메일을 소문자로 정규화한다.
3. 잘못된 IP를 제외한다.
4. 같은 IOC가 두 번 나와도 한 번만 출력한다.
5. IOC가 없는 빈 문자열에서 빈 목록을 반환한다.

## 통과 시험

1. 후보 추출과 유효성 검증의 차이를 설명한다.
2. `set`을 사용하는 이유와 정렬 시점을 설명한다.
3. URL·이메일 정규식의 한계를 말한다.
4. 중첩 JSON에서 안전하게 값을 꺼낸다.
5. 잘못된 IP와 중복 IOC 테스트를 통과시킨다.

## 최종 체크리스트

- [ ] IOC CLI에 URL과 이메일을 추가했다.
- [ ] 잘못된 IP를 제외했다.
- [ ] 모든 IOC를 중복 제거·정규화·정렬했다.
- [ ] JSON 출력 파일 옵션이 정상 동작한다.
- [ ] 단위 테스트를 5개 이상 만들었다.
