# 🐍 6. Scripting & Regex (PCRE)

반복 업무 자동화와 로그 파싱을 위한 도구들입니다.

## 1. 정규표현식 (PCRE) 리마인드
로그에서 원하는 정보만 추출할 때 필수적입니다.
- `\d`: 숫자, `\w`: 문자+숫자, `\s`: 공백
- `+`: 1개 이상, `*`: 0개 이상, `?`: 0개 또는 1개
- **IP 주소 정규식 예시:** `(?:\d{1,3}\.){3}\d{1,3}`
- **로그 취약점 패턴 예시:** `(?i)(union|select|insert|drop).*` (대소문자 무시 SQLi 탐지)

## 2. Shell Script 기반 로그 통계 (One-liner)
access.log에서 접속이 가장 많은 IP Top 10 뽑기:
```bash
cat /var/log/nginx/access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -10
```

## 3. Python 스크립트 예시 (로그 파일 읽기)
오류가 난 줄만 찾아서 출력하는 간단한 파이썬 로직.
```python
import re

log_file = "access.log"
error_pattern = re.compile(r" 404 | 500 ")

with open(log_file, "r") as f:
    for line in f:
        if error_pattern.search(line):
            print(line.strip())
```
