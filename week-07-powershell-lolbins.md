# 7주차 — PowerShell·LOLBins

## 이번 주 목표

PowerShell, rundll32, mshta의 명령줄을 행위 중심으로 탐지하고 정상 관리 행위와 공격 행위를 구분합니다. 탐지 규칙 3개와 MITRE ATT&CK 매핑을 만듭니다.

## 핵심 원칙

- 파일명 하나만으로 악성 판정하지 않습니다.
- 전체 명령줄, 부모, 사용자, 서명, 네트워크, 후속 행위를 함께 봅니다.
- 인코딩·URL·스크립트 실행 같은 위험 신호를 조합합니다.
- MITRE 매핑은 규칙 이름이 아니라 실제 탐지 행위를 근거로 합니다.

## 주야비비 8일 실행표

| 일차 | 근무 | 시간 | 할 일 | 결과물 |
|---|---:|---:|---|---|
| 1 | 주간 | 1시간 | 세 도구의 정상 용도 조사 | 비교표 |
| 2 | 야간 | 30분 | 위험 신호 5개 복원 | 카드 |
| 3 | 비번 | 3시간 | PowerShell 규칙 작성 | 규칙 1 |
| 4 | 비번 | 7시간 | rundll32·mshta 규칙 작성 | 규칙 2·3 |
| 5 | 주간 | 1시간 | MITRE 근거 정리 | 매핑 표 |
| 6 | 야간 | 30분 | 예외 3개 말하기 | 오탐 카드 |
| 7 | 비번 | 3시간 | EDR 데이터와 교차 확인 | 증거 표 |
| 8 | 비번 | 7시간 | 튜닝·통과 시험 | 최종 규칙 3개 |

## 규칙 1 — 인코딩 PowerShell

```spl
index=security_lab sourcetype="soc:windows" EventCode IN (1,4688) Image="*powershell.exe"
| where match(CommandLine,"(?i)(-enc|-encodedcommand)")
| rex field=CommandLine "(?i)(?:-enc|-encodedcommand)\s+(?<encoded_payload>\S+)"
| table _time extracted_host user ParentImage Image CommandLine encoded_payload
```

MITRE 후보: `T1059.001 PowerShell`.

## 규칙 2 — rundll32의 비정상 인자

```spl
index=security_lab sourcetype="soc:windows" EventCode IN (1,4688) Image="*rundll32.exe"
| where match(CommandLine,"(?i)(javascript:|https?://|\\\\)")
| table _time extracted_host user ParentImage Image CommandLine
```

MITRE 후보: `T1218.011 Rundll32`.

## 규칙 3 — mshta의 원격 콘텐츠

```spl
index=security_lab sourcetype="soc:windows" EventCode IN (1,4688) Image="*mshta.exe"
| rex field=CommandLine "(?<url>https?://[^\s]+)"
| where isnotnull(url)
| table _time extracted_host user ParentImage Image CommandLine url
```

MITRE 후보: `T1218.005 Mshta`.

## EDR 교차 검증

```spl
index=security_lab sourcetype="soc:edr" process IN ("powershell.exe","rundll32.exe","mshta.exe")
| table _time extracted_host user pid parent_pid parent_process process command_line sha256 destination disposition
| sort _time
```

Windows 로그의 `Image·ParentImage·CommandLine`과 EDR의 `process·parent_process·command_line`이 같은 행위를 가리키는지 비교합니다.

## 정상 예외 후보 3개 이상

1. 승인된 배포·관리 도구가 실행한 PowerShell.
2. 서명된 사내 스크립트의 정해진 경로와 해시.
3. 소프트웨어 설치 프로그램이 사용하는 rundll32.
4. 레거시 업무 앱이 승인된 내부 HTA를 실행하는 경우.

예외는 프로세스명 전체 제외가 아니라 `승인된 부모 + 경로 + 서명/해시 + 계정 + 시간`을 조합해 좁게 설계합니다.

## 추가 확인 로그

- PowerShell Script Block Logging 4104
- Sysmon ProcessGuid와 파일 생성 이벤트
- EDR 프로세스 트리·해시 평판
- 프록시·DNS·방화벽 로그
- 사용자 로그인과 변경 작업 기록

## 통과 시험

1. 세 규칙을 빈 검색창에서 작성한다.
2. 파일명만으로 차단하면 안 되는 이유를 설명한다.
3. 각 MITRE 기술과 탐지 근거를 연결한다.
4. 정상 예외 3개와 추가 로그 3개를 말한다.
5. Windows와 EDR 이벤트를 같은 타임라인에 설명한다.

## 최종 체크리스트

- [ ] 탐지 규칙 3개를 템플릿으로 문서화했다.
- [ ] 각 규칙에 MITRE ID와 근거를 적었다.
- [ ] 넓은 예외 대신 좁은 예외 조건을 설계했다.
- [ ] EDR 데이터로 명령줄과 부모를 교차 검증했다.
- [ ] 추가 수집이 필요한 로그를 기록했다.

