# ☁️ 7. Virtualization & Cloud (Docker/AWS)

과거 VM(VMware) 환경에서 컨테이너와 클라우드로 넘어온 인프라 트렌드 리마인드입니다.

## 1. Docker (컨테이너 가상화)
- OS 전체를 가상화하는 VM과 달리, 애플리케이션과 라이브러리만 격리하여 가볍고 빠름.
- **핵심 명령어:**
  - `docker ps`: 실행 중인 컨테이너 확인
  - `docker run -d -p 80:80 nginx`: Nginx 컨테이너 백그라운드 실행 및 포트 맵핑
  - `docker logs [컨테이너ID]`: 컨테이너 내부 로그 확인
  - `docker exec -it [컨테이너ID] /bin/bash`: 컨테이너 내부로 쉘 접속

## 2. AWS 핵심 개념
- **VPC (Virtual Private Cloud):** 논리적으로 격리된 가상 네트워크 (Subnet, Routing Table 구성)
- **EC2 (Elastic Compute Cloud):** 가상 서버 인스턴스 (Linux VM이라고 보면 됨)
- **Security Group (보안 그룹):** 인스턴스 단위의 가상 방화벽 (Inbound/Outbound 포트 허용 제어)
