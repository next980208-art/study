# 5주차 — Windows 인증 탐지

## 이번 주 목표

Windows 4624·4625 이벤트를 계정, 출발지, 단말, 시간 순서로 상관분석해 탐지 규칙 2개와 분석 보고서 1개를 만듭니다.

## 알아야 할 질문

- 누가 로그인했거나 실패했는가?
- 어느 단말에서 관찰되었는가?
- 출발지 IP와 대상 포트는 무엇인가?
- 실패가 성공으로 바뀌었는가?
- 같은 계정이 여러 단말에서 보였는가?

현재 합성 데이터에는 `LogonType`이 없습니다. 없는 필드를 추측하지 말고 제한사항에 기록합니다.

## 주야비비 8일 실행표

| 일차 | 근무 | 시간 | 할 일 | 결과물 |
|---|---:|---:|---|---|
| 1 | 주간 | 1시간 | 4624·4625 필드 비교 | 필드 표 |
| 2 | 야간 | 30분 | 정상 인증 패턴 설명 | 기준 메모 |
| 3 | 비번 | 3시간 | 반복 실패 탐지 작성 | 규칙 1 초안 |
| 4 | 비번 | 7시간 | 성공 전환·다중 호스트 분석 | 규칙 2 초안 |
| 5 | 주간 | 1시간 | 임계치와 시간창 검토 | 튜닝 표 |
| 6 | 야간 | 30분 | 조사 순서 복원 | 체크카드 |
| 7 | 비번 | 3시간 | 보고서 작성 | 보고서 초안 |
| 8 | 비번 | 7시간 | 재현·통과 시험 | 규칙 2개·보고서 |

## 탐지 1 — 반복 로그인 실패

```spl
index=security_lab sourcetype="soc:windows" EventCode=4625
| bin _time span=5m
| stats count min(_time) as first_seen max(_time) as last_seen values(extracted_host) as hosts values(DestinationPort) as ports by _time user DestinationIp
| where count>=3
```

예상은 `test-admin`, `198.51.100.24`, 실패 5건입니다.

튜닝 시 검토할 항목:

- 취약점 점검 계정과 승인된 스캐너 IP
- 비밀번호 변경 직후 발생한 사용자 실수
- 서비스 계정의 저장된 이전 비밀번호
- 인터넷 노출 시스템인지 내부 시스템인지
- 3회·5분이라는 임계치가 환경에 맞는지

## 탐지 2 — 실패 후 성공 전환 설계

현재 데이터에는 동일 사용자·출발지의 후속 성공이 없으므로 아래 규칙은 **0건이 정상**입니다.

```spl
index=security_lab sourcetype="soc:windows" EventCode IN (4624,4625)
| eval failure_time=if(EventCode=4625,_time,null())
| eval success_time=if(EventCode=4624,_time,null())
| stats count(eval(EventCode=4625)) as failures count(eval(EventCode=4624)) as successes min(failure_time) as first_failure max(success_time) as last_success by user DestinationIp extracted_host
| where failures>=3 AND successes>=1 AND last_success>first_failure
```

0건일 때 규칙 실패로 결론내리지 않습니다. 필요한 양성 테스트 데이터가 무엇인지 문서화합니다.

## 다중 호스트 로그인 확인

```spl
index=security_lab sourcetype="soc:windows" EventCode=4624
| stats dc(extracted_host) as host_count values(extracted_host) as hosts count by user
| where host_count>=2
```

현재 데이터에서는 0건입니다. 운영에서는 관리 계정, VPN·VDI, 점프 서버 사용을 예외 후보로 봅니다.

## 원본 검증

```spl
index=security_lab sourcetype="soc:windows" EventCode=4625 user="test-admin"
| sort _time
| table _time extracted_host user DestinationIp DestinationPort Outcome _raw
```

보고서에는 `발생 간격`, `단일 IP 집중`, `후속 성공 여부`, `추가 필요한 로그`를 적습니다.

## 결과물 작성

탐지 규칙은 `templates/detection-rule.md`, 조사는 `templates/investigation-report.md`를 복사해 작성합니다. 각 규칙에 다음을 반드시 포함합니다.

- 데이터 소스와 필수 필드
- 시간창·임계치와 선정 근거
- 예상 정상 행위와 제외 조건
- 0건·양성 테스트 방법
- 알림 후 조사 순서

## 통과 시험

1. 4624와 4625의 의미와 필수 문맥을 설명한다.
2. 반복 실패 5건을 원본까지 재현한다.
3. 실패 후 성공 규칙이 0건인 이유를 설명한다.
4. 임계치를 낮추거나 높일 때 장단점을 말한다.
5. `LogonType` 부재를 제한사항으로 식별한다.

## 최종 체크리스트

- [ ] 탐지 규칙 2개를 작성했다.
- [ ] 임계치 근거와 오탐 조건을 각 규칙에 적었다.
- [ ] 양성·음성 테스트를 구분했다.
- [ ] 조사 보고서 1개를 완성했다.
- [ ] 없는 필드를 추측하지 않고 추가 수집 항목으로 기록했다.

