# 9주차 — Falcon 조사 기본기

## 이번 주 목표

EDR 탐지에서 호스트·사용자·프로세스 트리·명령줄·IOC를 순서대로 확인하는 조사 습관을 만듭니다. 합성 데이터로 조사 보고서 2개를 작성합니다.

> Falcon 콘솔 메뉴와 필드명은 권한·버전에 따라 달라질 수 있습니다. 이번 주 핵심은 버튼 암기가 아니라 `알림 → 호스트 → 프로세스 → IOC → 범위` 조사 흐름입니다.

## 데이터 확인

```spl
index=security_lab sourcetype="soc:edr"
| stats count by extracted_host
```

예상은 LAB-WS01 4건, LAB-WS02 3건으로 총 7건입니다.

## 주야비비 8일 실행표

| 일차 | 근무 | 시간 | 할 일 | 결과물 |
|---|---:|---:|---|---|
| 1 | 주간 | 1시간 | EDR 필드 사전 작성 | 필드 표 |
| 2 | 야간 | 30분 | 조사 순서 복원 | 카드 |
| 3 | 비번 | 3시간 | LAB-WS01 트리 재구성 | 보고서 1 초안 |
| 4 | 비번 | 7시간 | LAB-WS02 트리 재구성 | 보고서 2 초안 |
| 5 | 주간 | 1시간 | Windows 로그 교차 검증 | 증거 표 |
| 6 | 야간 | 30분 | 최초 실행점 말하기 | 오답 카드 |
| 7 | 비번 | 3시간 | IOC·후속 행위 정리 | 피벗 표 |
| 8 | 비번 | 7시간 | 보고서 완성·통과 시험 | 보고서 2개 |

## EDR 필드 사전

| 필드 | 조사 질문 |
|---|---|
| `extracted_host` | 어느 단말인가? |
| `user` | 어떤 사용자 문맥인가? |
| `pid`, `parent_pid` | 프로세스 관계는 무엇인가? |
| `process`, `parent_process` | 어떤 실행 계보인가? |
| `command_line` | 실제 동작과 인자는 무엇인가? |
| `sha256` | 같은 파일을 어디서 더 찾을 수 있는가? |
| `destination` | 어느 IP·도메인·포트로 연결했는가? |
| `disposition` | 탐지·차단·관찰 중 무엇인가? |

## 사례 1 — PowerShell 프로세스 트리

```spl
index=security_lab sourcetype="soc:edr" extracted_host="LAB-WS01"
| sort _time
| table _time user pid parent_pid parent_process process command_line sha256 destination disposition
```

재구성할 계보:

```text
userinit.exe
└─ explorer.exe (PID 3120)
   └─ powershell.exe (PID 4880, EncodedCommand)
      ├─ 203.0.113.45:443 연결
      └─ cmd.exe (PID 5024, whoami)
```

최초 정상 실행점은 `explorer.exe`, 최초 의심 실행점은 인코딩 옵션을 사용한 `powershell.exe`로 구분합니다.

## 사례 2 — rundll32에서 mshta 실행

```spl
index=security_lab sourcetype="soc:edr" extracted_host="LAB-WS02"
| sort _time
| table _time user pid parent_pid parent_process process command_line sha256 destination disposition
```

재구성할 계보:

```text
userinit.exe
└─ explorer.exe (PID 2200)
   └─ rundll32.exe (PID 5300, javascript 인자)
      └─ mshta.exe (PID 5412, 원격 HTA, blocked)
```

`blocked`는 해당 행위가 차단되었다는 증거이지, 앞선 행위와 전체 침해가 없었다는 뜻은 아닙니다.

## Windows 로그 교차 확인

```spl
index=security_lab sourcetype IN ("soc:windows","soc:edr") extracted_host IN ("LAB-WS01","LAB-WS02")
| eval process_name=coalesce(process,replace(Image,"^.*\\\\",""))
| eval command=coalesce(command_line,CommandLine)
| table _time sourcetype extracted_host user process_name parent_process ParentImage command destination DestinationIp disposition Outcome
| sort _time
```

## 보고서에 답할 질문

1. 알림의 핵심 행위는 무엇인가?
2. 최초 의심 프로세스와 부모는 무엇인가?
3. 자식·네트워크 행위는 무엇인가?
4. 해시·명령줄·목적지는 무엇인가?
5. EDR이 차단했는가, 관찰만 했는가?
6. 다른 호스트 범위를 어떻게 확인할 것인가?
7. 현재 증거로 가능한 판정과 부족한 증거는 무엇인가?

## 통과 시험

1. 두 프로세스 트리를 화면 없이 그린다.
2. 최초 실행점과 최초 의심 실행점을 구분한다.
3. 차단 이벤트 이후에도 조사할 항목을 3개 말한다.
4. Windows와 EDR 필드의 대응을 설명한다.
5. 알림→호스트→프로세스→IOC 순서를 재현한다.

## 최종 체크리스트

- [ ] 조사 보고서 2개를 템플릿으로 작성했다.
- [ ] 프로세스 트리 2개를 부모·PID와 함께 그렸다.
- [ ] 명령줄·해시·목적지를 증거로 기록했다.
- [ ] EDR과 Windows 이벤트를 교차 검증했다.
- [ ] 판정에 부족한 로그를 명시했다.

