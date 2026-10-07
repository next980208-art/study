# 15주차 — 로그 집계 자동화

## 이번 주 목표

`csv`, `Counter`, `datetime`, 정렬과 예외 처리를 사용해 EventCode·사용자·의심 명령줄을 요약하는 CLI 도구를 완성합니다.

## 기준 파일

- 코드: `study-lab\python\log_summary.py`
- 입력: `study-lab\datasets\windows_events.csv`

## 먼저 실행

```powershell
python python\log_summary.py datasets\windows_events.csv
```

예상 핵심 값은 전체 16건, 4625 5건, 의심 명령줄 3건입니다.

## 주야비비 8일 실행표

| 일차 | 근무 | 시간 | 할 일 | 결과물 |
|---|---:|---:|---|---|
| 1 | 주간 | 1시간 | DictReader·Counter 이해 | 노트 |
| 2 | 야간 | 30분 | 집계 함수 복원 | 카드 |
| 3 | 비번 | 3시간 | EventCode·사용자 집계 | 기능 1 |
| 4 | 비번 | 7시간 | 시간 파싱·정렬·의심 명령 | 기능 2 |
| 5 | 주간 | 1시간 | 오류 사례 설계 | 오류 표 |
| 6 | 야간 | 30분 | 빈 파일 처리 작성 | 점검 |
| 7 | 비번 | 3시간 | JSON·CSV 출력 추가 | 기능 3 |
| 8 | 비번 | 7시간 | 테스트·README | 완성 도구 |

## 핵심 집계

```python
from collections import Counter

event_counts = Counter(row["EventCode"] for row in rows)
failed_users = Counter(row["user"] for row in rows if row["EventCode"] == "4625")
```

예상 EventCode 순위는 4625=5, 1=4, 4624=3, 3=2, 22=1, 4688=1입니다.

## 시간 처리

```python
from datetime import datetime

def parse_time(value: str) -> datetime:
    return datetime.fromisoformat(value.replace("Z", "+00:00"))
```

다음을 추가합니다.

- 최초·최종 이벤트 시각
- 사용자별 최초·최종 시각
- 모든 출력의 시간순 또는 건수순 정렬
- 잘못된 시간 문자열의 행 번호와 오류 메시지

## 의심 명령줄 규칙

다음 토큰을 대소문자 없이 찾습니다.

- `encodedcommand`
- `rundll32.exe javascript`
- `mshta.exe http`

각 결과에 `_time`, `host`, `user`, `Image`, `ParentImage`, `CommandLine`을 포함합니다.

## 안전한 입력 검증

반드시 처리할 오류:

1. 입력 파일이 없음
2. 빈 CSV
3. 헤더가 없음
4. `EventCode` 또는 `CommandLine` 필드가 없음
5. UTF-8 디코딩 실패
6. 잘못된 시간 형식

오류 메시지에는 사용자에게 필요한 해결 방법을 포함하고 비정상 종료 코드를 반환합니다.

## 출력 옵션 요구사항

```text
기본: 사람이 읽는 텍스트 요약
--format json: 자동화 가능한 JSON
--output PATH: 파일로 저장
--top N: 상위 N개만 표시
```

## 결과 검증

Python 결과를 Splunk와 비교합니다.

```spl
index=security_lab sourcetype="soc:windows"
| stats count by EventCode
| sort - count
```

두 도구의 데이터 파일과 시간 범위가 같은지 먼저 확인한 뒤 건수를 비교합니다.

## 통과 시험

1. `Counter` 없이 직접 집계 원리를 설명한다.
2. 전체 16건과 EventCode 분포를 정확히 출력한다.
3. 의심 명령줄 3건을 찾는다.
4. 입력 파일 오류와 필수 열 누락을 안전하게 처리한다.
5. Python 결과와 Splunk 결과를 교차 검증한다.

## 최종 체크리스트

- [ ] 로그 요약 CLI를 완성했다.
- [ ] EventCode·실패 사용자·의심 명령줄을 요약한다.
- [ ] 최초·최종 시각과 정렬을 구현했다.
- [ ] JSON 또는 파일 출력 옵션을 추가했다.
- [ ] 여섯 오류 조건을 처리했다.
