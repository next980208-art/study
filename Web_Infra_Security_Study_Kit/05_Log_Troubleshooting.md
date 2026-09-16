# 🔍 5. Log Analysis & Troubleshooting

로그를 기반으로 상황을 유추하고 장애를 해결하는 '엔지니어의 기본기'입니다.

## 1. 주요 시스템 로그 위치 (Linux)
- `/var/log/messages` (또는 `syslog`): 시스템 데몬 및 전반적인 커널 메시지
- `/var/log/secure` (또는 `auth.log`): SSH 로그인 성공/실패 기록 (Brute Force 확인)
- `/var/log/nginx/access.log`, `error.log`: 웹 접속 기록 및 에러 발생 내역

## 2. Syslog & ELK 스택 (Elasticsearch)
- **Syslog:** 여러 시스템의 로그를 중앙 집중식 서버로 보내는 표준 프로토콜 (UDP 514 주로 사용).
- **Elasticsearch (ELK):** 방대한 로그를 JSON 형태로 파싱하여 저장하고, 키워드로 초고속 검색이 가능하게 해주는 검색엔진 기반 로그 시스템.

## 3. 대표적인 웹 장애 트러블슈팅
- **502 Bad Gateway:** 
  - *상황:* Nginx는 살아있으나, 뒤단 WAS(Tomcat)가 죽었거나 응답을 못할 때 발생.
  - *해결:* Nginx `error.log` 확인 -> Tomcat 프로세스(`ps -ef`) 및 Tomcat 로그(`catalina.out`) 확인.
- **504 Gateway Timeout:**
  - *상황:* Nginx가 Tomcat으로 넘겼으나 설정된 시간(Timeout) 내에 응답이 안 옴. (DB 락(Lock)이나 무거운 쿼리로 인한 지연이 주원인)
