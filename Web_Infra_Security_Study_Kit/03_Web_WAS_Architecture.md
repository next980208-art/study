# 🏗️ 3. Web & WAS Architecture

3-Tier 아키텍처(Web - WAS - DB)의 동작 구조와 연동 방식입니다.

## 1. Web Server (Apache, Nginx)
- **역할:** 정적 콘텐츠(HTML, 이미지) 처리, 리버스 프록시, 로드 밸런싱, SSL 터미네이션.
- **Nginx Reverse Proxy 설정 예시 (기억 환기용):**
  ```nginx
  server {
      listen 80;
      server_name example.com;
      
      location / {
          proxy_pass http://127.0.0.1:8080; # Tomcat으로 포워딩
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
      }
  }
  ```

## 2. WAS - Web Application Server (Tomcat, JBoss)
- **역할:** 동적 로직(JSP, Spring 등) 처리, DB 커넥션 풀(DBCP) 관리.
- **주요 포인트:** 
  - `server.xml`: Connector 포트(8080) 설정.
  - OOM(Out of Memory) 방지를 위한 JVM Heap 메모리 설정 (`-Xms`, `-Xmx`).

## 3. 시스템 간 연동 (API)
- **REST API:** HTTP URI로 리소스를 명시하고 HTTP Method(GET, POST, PUT, DELETE)로 행위 정의.
- **JSON:** Key-Value 형태의 경량 데이터 교환 포맷. (대부분의 REST API 응답 포맷)
