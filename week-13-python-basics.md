# 13주차 — Python 기초 복구

## 이번 주 목표

자료형·조건·반복·함수·파일 입출력·`pathlib`을 다시 익혀 CSV와 텍스트에서 필요한 행을 스스로 출력합니다. 짧은 스크립트 5개를 만듭니다.

## 준비

저장소 최상위 폴더에서 가상환경을 만든 뒤 Python 버전을 확인합니다.

```powershell
python -m venv python\.venv
python\.venv\Scripts\Activate.ps1
python --version
```

실습 입력은 다음 두 파일입니다.

- `datasets\windows_events.csv`
- `datasets\ioc_sample.txt`

## 주야비비 8일 실행표

| 일차 | 근무 | 시간 | 할 일 | 결과물 |
|---|---:|---:|---|---|
| 1 | 주간 | 1시간 | 문자열·숫자·리스트·딕셔너리 | 예제 1 |
| 2 | 야간 | 30분 | 조건문과 반복문 복원 | 예제 2 |
| 3 | 비번 | 3시간 | 함수와 예외 기본 | 예제 3 |
| 4 | 비번 | 7시간 | pathlib로 텍스트·CSV 읽기 | 예제 4·5 |
| 5 | 주간 | 1시간 | 함수 입력·반환값 정리 | 함수 노트 |
| 6 | 야간 | 30분 | 빈 파일 읽기 함수 작성 | 점검 |
| 7 | 비번 | 3시간 | 다섯 스크립트 다시 작성 | 재현 기록 |
| 8 | 비번 | 7시간 | 오류 처리·README | 완성본 |

## 스크립트 1 — 자료형과 조건

```python
event = {"EventCode": "4625", "user": "test-admin", "count": 5}
severity = "high" if event["count"] >= 5 else "medium"
print(event["user"], severity)
```

직접 바꿔 볼 값은 `count=2`, `EventCode=4624`, 빈 사용자입니다.

## 스크립트 2 — 목록 반복과 집계

```python
event_codes = ["4624", "4625", "4625", "1"]
counts = {}
for code in event_codes:
    counts[code] = counts.get(code, 0) + 1
print(counts)
```

`collections.Counter`를 쓰기 전에 직접 집계 원리를 이해합니다.

## 스크립트 3 — 함수 분리

```python
def is_failed_logon(row: dict[str, str]) -> bool:
    return row.get("EventCode") == "4625"
```

입력, 반환값, 부작용을 자신의 말로 설명합니다.

## 스크립트 4 — 텍스트 파일 읽기

```python
from pathlib import Path

def read_nonempty_lines(path: Path) -> list[str]:
    return [line.strip() for line in path.read_text(encoding="utf-8").splitlines() if line.strip()]
```

존재하지 않는 파일, UTF-8이 아닌 파일, 빈 파일일 때 무엇이 일어나는지 확인합니다.

## 스크립트 5 — CSV에서 로그인 실패 출력

```python
import csv
from pathlib import Path

def failed_logons(path: Path) -> list[dict[str, str]]:
    with path.open(encoding="utf-8", newline="") as stream:
        return [row for row in csv.DictReader(stream) if row.get("EventCode") == "4625"]
```

각 행에서 `_time`, `host`, `user`, `DestinationIp`만 출력하도록 확장합니다. 예상 실패는 5건입니다.

## 반드시 연습할 오류

```python
try:
    rows = failed_logons(Path("missing.csv"))
except FileNotFoundError as error:
    print(f"입력 파일을 찾을 수 없습니다: {error.filename}")
```

오류를 무조건 숨기는 `except Exception: pass`는 사용하지 않습니다.

## 결과물 규칙

각 스크립트 위에 다음을 적습니다.

```text
입력:
출력:
핵심 문법:
실패할 수 있는 조건:
직접 바꿔 본 부분:
```

## 통과 시험

1. 리스트와 딕셔너리의 사용 차이를 설명한다.
2. 복사 없이 UTF-8 파일 읽기 함수를 작성한다.
3. CSV에서 4625 5건을 필터링한다.
4. 함수의 입력과 반환값을 설명한다.
5. 파일 없음 오류를 안전하게 처리한다.

## 최종 체크리스트

- [ ] 짧은 스크립트 5개를 직접 작성했다.
- [ ] 모든 경로를 `pathlib.Path`로 처리했다.
- [ ] 파일을 UTF-8로 명시해 읽었다.
- [ ] 적어도 한 함수를 입력·반환 구조로 분리했다.
- [ ] 노트 없이 파일 읽기 함수를 다시 작성했다.
