# 16주차 — 테스트와 도구 완성

## 이번 주 목표

IOC 추출기와 로그 요약기를 함수·CLI로 분리하고 `argparse`, `unittest`, README를 갖춘 재현 가능한 도구로 완성합니다. 새 PC에서 README만 보고 실행할 수 있어야 합니다.

## 완성 기준

| 항목 | IOC 추출기 | 로그 요약기 |
|---|---|---|
| 입력 검증 | 파일·인코딩 | 파일·헤더·시간 |
| 함수 분리 | 추출·보고서 | 읽기·집계·출력 |
| CLI | input/output/format | input/output/format/top |
| 테스트 | 정상·오류·중복 | 정상·빈 파일·열 누락 |
| 문서 | 예시 입력·출력 | 예시 입력·출력 |

## 주야비비 8일 실행표

| 일차 | 근무 | 시간 | 할 일 | 결과물 |
|---|---:|---:|---|---|
| 1 | 주간 | 1시간 | 함수·CLI 경계 설계 | 구조도 |
| 2 | 야간 | 30분 | 테스트 Arrange/Act/Assert | 카드 |
| 3 | 비번 | 3시간 | IOC 테스트 확장 | 테스트 1 |
| 4 | 비번 | 7시간 | 로그 요약 테스트 작성 | 테스트 2 |
| 5 | 주간 | 1시간 | argparse 도움말 개선 | CLI UX |
| 6 | 야간 | 30분 | 깨끗한 실행 순서 복원 | 점검 |
| 7 | 비번 | 3시간 | README 설치·예제 | 문서 |
| 8 | 비번 | 7시간 | 전체 재현·최종 QA | 도구 2개 |

## 권장 함수 구조

```text
ioc_extractor.py
├─ extract_iocs(text)
├─ build_report(path)
├─ render_report(report, format)
├─ parse_args()
└─ main()

log_summary.py
├─ read_rows(path)
├─ summarize(rows)
├─ render_text(summary)
├─ write_output(...)
├─ parse_args()
└─ main()
```

테스트는 `main()`보다 입력과 출력이 명확한 작은 함수를 우선 검사합니다.

## 테스트 실행

```powershell
Set-Location python
python -m unittest discover -v
```

최소 테스트 목록:

1. 유효 IP·도메인·해시 추출
2. 잘못된 IP 제외
3. URL·이메일 정규화와 중복 제거
4. Windows CSV 전체 16건 집계
5. EventCode별 정확한 분포
6. 의심 명령줄 3건
7. 빈 파일 오류
8. 필수 열 누락 오류
9. 존재하지 않는 파일 오류
10. JSON 출력이 다시 파싱됨

## 임시 파일 테스트

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as directory:
    sample = Path(directory) / "sample.csv"
    sample.write_text("_time,EventCode,user,CommandLine\n", encoding="utf-8")
    # 함수 실행과 결과 검증
```

테스트가 실제 작업 파일을 덮어쓰지 않도록 임시 폴더를 사용합니다.

## README 필수 항목

```markdown
# 도구 이름과 한 줄 목적
## 요구 환경
## 폴더 구조
## 빠른 시작
## CLI 옵션
## 입력 형식
## 출력 예시
## 테스트 실행
## 오류 해결
## 보안·개인정보 주의
## 제한사항
```

명령은 작업 폴더 기준으로 그대로 복사해 실행 가능해야 합니다.

## 새 PC 재현 시험

1. README만 열고 필요한 Python 버전을 확인합니다.
2. 테스트를 실행합니다.
3. 합성 입력으로 두 CLI를 실행합니다.
4. 출력 파일을 생성하고 다시 엽니다.
5. 잘못된 경로를 넣어 오류 메시지를 확인합니다.

## 통과 시험

1. 두 도구의 함수 구조를 설명한다.
2. 모든 테스트를 한 명령으로 실행한다.
3. 실패 테스트를 보고 원인을 찾는다.
4. 새 PC 절차를 README만으로 재현한다.
5. 회사 데이터 없이 결과를 시연한다.

## 최종 체크리스트

- [ ] Python 도구 2개가 CLI로 동작한다.
- [ ] 최소 10개의 단위 테스트가 통과한다.
- [ ] 테스트가 실제 데이터를 수정하지 않는다.
- [ ] `--help`와 오류 메시지가 이해하기 쉽다.
- [ ] README에 설치·실행·테스트·제한사항이 있다.
