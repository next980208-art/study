# 🐧 1. Linux OS & System Deep Dive

과거에 다루었던 리눅스 서버 리소스 분석 및 핵심 명령어 리마인드입니다. 장애 발생 시 가장 먼저 확인해야 할 지표들입니다.

## 1. 프로세스 (Process) 분석
- `ps -ef | grep [프로세스명]`: 실행 중인 프로세스 확인 (PID, PPID 확인)
- `top` / `htop`: 실시간 시스템 리소스 사용량 (CPU, Memory)
  - `top` 실행 후 `Shift + P` (CPU 정렬), `Shift + M` (메모리 정렬)
- `lsof -p [PID]`: 특정 프로세스가 열고 있는 파일 및 네트워크 소켓 확인
- `/proc/`: 리눅스는 모든 것을 파일로 관리. `/proc/cpuinfo`, `/proc/meminfo` 등 확인 가능.

## 2. 메모리 (Memory) 분석
- `free -h`: 메모리 및 Swap 사용량 확인. (Available 메모리가 실제 가용 메모리)
- `vmstat 1`: 1초 간격으로 가상 메모리, 프로세스, CPU 활동 모니터링 (병목현상 파악)

## 3. 디스크 및 파일시스템 (Filesystem)
- `df -h`: 마운트된 파티션별 디스크 사용량 확인 (100% 차면 장애 발생)
- `du -sh *`: 현재 디렉토리 내 파일/폴더별 용량 확인 (로그 파일이 꽉 찼을 때 범인 찾기)
- `iostat -x 1`: 디스크 I/O 상태 확인 (%util이 100%에 가까우면 디스크 병목)

## 4. 네트워크 (Network) 상태
- `netstat -tulpn`: 현재 서버에서 LISTEN 중인 포트 및 PID 확인
- `ss -antp`: netstat보다 빠르고 상세한 소켓 정보 출력
- `ip a`: 네트워크 인터페이스 및 IP 주소 확인 (ifconfig 대체)
