# 10주차 — 범위 확인과 피벗

## 이번 주 목표

해시·명령줄·도메인·IP·사용자·행위를 다른 호스트로 확장해 단일 단말 사건인지 전사 범위 사건인지 판단합니다. 조사 보고서 2개와 피벗 표를 만듭니다.

## IOC와 행위의 차이

- IOC 피벗: 동일 해시, IP, 도메인처럼 정확히 같은 값을 찾습니다.
- 행위 피벗: 인코딩 PowerShell, LOLBin의 원격 URL 실행처럼 변형되어도 남는 패턴을 찾습니다.
- IOC가 없다고 같은 공격이 없는 것은 아닙니다. 해시와 주소는 쉽게 바뀔 수 있습니다.

## 주야비비 8일 실행표

| 일차 | 근무 | 시간 | 할 일 | 결과물 |
|---|---:|---:|---|---|
| 1 | 주간 | 1시간 | 피벗 키 분류 | 피벗 표 초안 |
| 2 | 야간 | 30분 | IOC·행위 차이 복원 | 카드 |
| 3 | 비번 | 3시간 | 해시·명령줄 범위 검색 | 보고서 1 |
| 4 | 비번 | 7시간 | IP·도메인·행위 범위 검색 | 보고서 2 |
| 5 | 주간 | 1시간 | 단일/전사 판정 기준 | 기준표 |
| 6 | 야간 | 30분 | 피벗 순서 말하기 | 점검 |
| 7 | 비번 | 3시간 | 음성 결과 해석 | 한계 노트 |
| 8 | 비번 | 7시간 | 결과 정리·통과 시험 | 피벗 표 완성 |

## 피벗 표

| 피벗 키 | 값 | 검색 범위 | 결과 | 해석 |
|---|---|---|---:|---|
| SHA256 | `aaaa...` | 모든 EDR 호스트 | 직접 확인 | 같은 파일 범위 |
| 명령줄 | `EncodedCommand` | Windows+EDR | 직접 확인 | 유사 실행 행위 |
| 목적지 | `203.0.113.45` | 네트워크 필드 | 직접 확인 | 통신 범위 |
| 프로세스 | `powershell.exe` | 모든 EDR 호스트 | 직접 확인 | 너무 넓어 조건 추가 필요 |

## 동일 해시 범위

```spl
index=security_lab sourcetype="soc:edr" sha256="aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
| stats count min(_time) as first_seen max(_time) as last_seen values(user) as users values(command_line) as commands values(destination) as destinations by extracted_host
```

현재 데이터에서는 LAB-WS01에만 나타납니다.

## 동일 목적지 범위

```spl
index=security_lab sourcetype="soc:edr" destination="203.0.113.45:443"
| stats count values(process) as processes values(command_line) as commands values(user) as users by extracted_host
```

Windows 네트워크 이벤트까지 확대합니다.

```spl
index=security_lab sourcetype="soc:windows" DestinationIp="203.0.113.45"
| table _time extracted_host user Image CommandLine DestinationIp DestinationPort
```

## 유사 명령줄 범위

```spl
index=security_lab sourcetype IN ("soc:windows","soc:edr")
| eval command=coalesce(command_line,CommandLine)
| where match(command,"(?i)(encodedcommand|rundll32\.exe\s+javascript|mshta\.exe\s+https?://)")
| stats values(sourcetype) as sources values(command) as commands count by extracted_host user
```

## 최초·최종 관찰 시각

```spl
index=security_lab sourcetype="soc:edr" process IN ("powershell.exe","rundll32.exe","mshta.exe")
| stats earliest(_time) as first_seen latest(_time) as last_seen values(command_line) as commands values(sha256) as hashes by process extracted_host
| convert ctime(first_seen) ctime(last_seen)
```

## 범위 판정 문장

```text
현재 제공된 7건의 EDR 합성 데이터에서는 PowerShell 계열 행위가 LAB-WS01,
rundll32→mshta 계열 행위가 LAB-WS02에 각각 한정된다. 다만 데이터 보존 범위와
수집 대상 전체를 확인하지 못했으므로 전사 미발생을 확정할 수 없다.
```

## 통과 시험

1. IOC 피벗과 행위 피벗을 각각 두 가지 제시한다.
2. PowerShell 해시·IP·명령줄을 다른 호스트로 확장한다.
3. 0건 결과가 안전을 증명하지 못하는 이유를 설명한다.
4. 단일 호스트와 전사 범위 판정에 필요한 수집 범위를 말한다.
5. 피벗 결과를 과장 없이 한 문장으로 요약한다.

## 최종 체크리스트

- [ ] 조사 보고서 2개에 범위 확인 절을 추가했다.
- [ ] 해시·IP·도메인·명령줄·행위 피벗을 수행했다.
- [ ] 피벗 표에 검색 범위와 결과를 기록했다.
- [ ] 데이터 보존 기간과 수집 범위 한계를 적었다.
- [ ] 단일 단말과 전사 범위를 구분해 표현했다.

