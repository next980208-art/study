# 🛡️ 4. Web Security & Solutions

웹 보안 솔루션의 위치와 주요 취약점 공격 리마인드입니다.

## 1. 보안 솔루션의 차이
- **WAF (Web Application Firewall):** L7 계층(웹) 방어. HTTP 헤더/바디 분석, SQLi/XSS 차단 특화.
- **IPS / IDS (Intrusion Prevention/Detection System):** L3~L4 계층 방어 및 L7 일부 탐지. 네트워크 패킷의 시그니처 기반 비정상 트래픽 탐지 및 차단.

## 2. 주요 웹 공격 기법 및 방어
- **SQL Injection (SQLi)**
  - *원리:* 입력값 검증 누락으로 쿼리문을 조작 (`' OR 1=1 --`)하여 DB 인증 우회 및 데이터 탈취.
  - *방어:* Prepared Statement(바인딩) 사용, WAF 시그니처 차단.
- **XSS (Cross Site Scripting)**
  - *원리:* 악성 스크립트(`<script>`)를 게시판 등에 삽입하여, 열람하는 사용자의 세션 탈취.
  - *방어:* 입력값 및 출력값 필터링(HTML Entity 치환).
- **File Upload 취약점**
  - *원리:* 공격자가 웹쉘(Web Shell, 악성 PHP/JSP 파일)을 업로드 후 실행하여 서버 제어권 획득.
  - *방어:* 업로드 파일 확장자 제한(White List), 업로드 디렉토리 실행 권한 제거.
