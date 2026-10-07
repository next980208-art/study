# 6주차 — Sysmon 프로세스 분석

## 이번 주 목표

Event ID 1·3·22의 부모·자식 프로세스, 명령줄, 네트워크, DNS를 시간순으로 연결합니다. 탐지 규칙 2개와 PowerShell 타임라인 1개를 만듭니다.

> 현재 합성 CSV에는 `ProcessGuid`가 없습니다. `extracted_host + user + Image + 시간`으로 제한적인 연결을 수행하고, 실제 Sysmon 조사에서는 ProcessGuid가 필요한 이유를 기록합니다.

## 주야비비 8일 실행표

| 일차 | 근무 | 시간 | 할 일 | 결과물 |
|---|---:|---:|---|---|
| 1 | 주간 | 1시간 | Event ID 1·3·22 필드 비교 | 필드 사전 |
| 2 | 야간 | 30분 | 부모→자식 개념 복원 | 계보 메모 |
| 3 | 비번 | 3시간 | 프로세스 생성 탐색 | 규칙 1 초안 |
| 4 | 비번 | 7시간 | PowerShell→네트워크→DNS 타임라인 | 타임라인 |
| 5 | 주간 | 1시간 | ProcessGuid 필요성 정리 | 제한사항 |
| 6 | 야간 | 30분 | 피벗 순서 암기 점검 | 카드 |
| 7 | 비번 | 3시간 | 외부 연결 규칙 작성 | 규칙 2 초안 |
| 8 | 비번 | 7시간 | 재현·문서화 | 완성본 |

## 데이터 훑기

```spl
index=security_lab sourcetype="soc:windows" EventCode IN (1,3,22)
| table _time extracted_host user EventCode Image ParentImage CommandLine DestinationIp DestinationPort Outcome
| sort _time
```

예상은 EventCode 1이 4건, 3이 2건, 22가 1건입니다.

## 탐지 1 — 의심 프로세스 생성

```spl
index=security_lab sourcetype="soc:windows" EventCode IN (1,4688)
| eval suspicious=if(match(CommandLine,"(?i)(encodedcommand|rundll32\.exe\s+javascript|mshta\.exe\s+https?://)"),1,0)
| where suspicious=1
| table _time extracted_host user Image ParentImage CommandLine
```

예상은 PowerShell, rundll32, mshta 관련 3건입니다.

## 탐지 2 — 프로세스의 외부 연결

```spl
index=security_lab sourcetype="soc:windows" EventCode=3 DestinationIp!=""
| eval is_training_external=if(cidrmatch("203.0.113.0/24",DestinationIp) OR cidrmatch("192.0.2.0/24",DestinationIp),1,0)
| table _time extracted_host user Image DestinationIp DestinationPort is_training_external
```

두 IP 대역은 문서용 합성 주소입니다. 실제 운영에서는 자산·프록시·위협정보 문맥이 필요합니다.

## PowerShell 타임라인

```spl
index=security_lab sourcetype="soc:windows" extracted_host="LAB-WS01" user="kim" earliest="09/01/2026:09:15:00" latest="09/01/2026:09:17:00"
| eval activity=case(EventCode=1,"process_create",EventCode=3,"network_connect",EventCode=22,"dns_query",true(),"other")
| table _time activity EventCode Image ParentImage CommandLine DestinationIp DestinationPort Outcome
| sort _time
```

타임라인 해석:

1. 09:15:42 PowerShell이 `explorer.exe`에서 실행됩니다.
2. 09:16:03 같은 단말·사용자에서 `203.0.113.45:443` 연결이 관찰됩니다.
3. 09:16:04 DNS 질의 샘플이 관찰됩니다.

정확히 같은 프로세스라고 단정할 `ProcessGuid`가 없으므로 `시간적으로 인접한 관련 행위`라고 표현합니다.

## ProcessGuid 기반 실제 피벗 순서

1. Event ID 1에서 의심 프로세스의 `ProcessGuid`를 확보합니다.
2. 같은 `ProcessGuid`의 Event ID 3·22·11 등을 검색합니다.
3. `ParentProcessGuid`로 부모 프로세스를 찾습니다.
4. 자식 이벤트의 `ParentProcessGuid`가 대상 `ProcessGuid`인지 검색합니다.
5. 사용자·해시·서명·목적지를 다른 호스트로 확장합니다.

## 통과 시험

1. Event ID 1·3·22의 역할을 설명한다.
2. PowerShell 실행부터 외부 연결까지 순서대로 재현한다.
3. 부모와 자식 필드의 차이를 말한다.
4. 현재 데이터로 같은 프로세스를 단정할 수 없는 이유를 설명한다.
5. ProcessGuid 기반 피벗 순서를 화면 없이 말한다.

## 최종 체크리스트

- [ ] 탐지 규칙 2개를 작성했다.
- [ ] 타임라인에 시각·행위·근거를 기록했다.
- [ ] 정상 브라우저 연결과 의심 PowerShell 연결을 비교했다.
- [ ] ProcessGuid 부재를 제한사항으로 적었다.
- [ ] 추가 확인 로그를 3개 이상 제시했다.

