# 🌐 2. Network Protocols & Packet Analysis

웹 서비스 통신의 기반이 되는 TCP/IP 및 패킷 분석 도구 리마인드입니다.

## 1. TCP/IP 통신 흐름
- **3-Way Handshake (연결 수립)**
  1. `SYN`: 클라이언트 -> 서버 (연결 요청)
  2. `SYN-ACK`: 서버 -> 클라이언트 (수락 및 연결 요청)
  3. `ACK`: 클라이언트 -> 서버 (수락 확인)
- **4-Way Handshake (연결 종료)**
  - `FIN` 패킷을 주고받으며 연결을 안전하게 종료 (`TIME_WAIT` 상태 발생 원인)

## 2. HTTP / HTTPS 기본
- **HTTP 구조:** Request Line(메서드, URI, 버전) + Headers + Body
- **주요 상태 코드:**
  - `200` OK, `301/302` Redirect
  - `400` Bad Request, `403` Forbidden, `404` Not Found
  - `500` Internal Server Error, `502` Bad Gateway, `504` Gateway Timeout
- **HTTPS:** HTTP + SSL/TLS. 443 포트 사용, 인증서 기반 암호화 통신.

## 3. 패킷 분석 (Packet Capture)
서버에 콘솔 접속 후 통신이 안 될 때 직접 패킷을 떠서 확인합니다.
- **tcpdump 명령어**
  - `tcpdump -i eth0`: eth0 인터페이스 전체 캡처
  - `tcpdump -i eth0 port 80 -w capture.pcap`: 80포트 패킷을 파일로 저장
  - `tcpdump -i eth0 host 192.168.1.10`: 특정 IP 통신 캡처
- **Wireshark (PC에서 분석)**
  - 필터: `ip.addr == 192.168.1.10`, `tcp.port == 80`, `http.request.method == "GET"`
