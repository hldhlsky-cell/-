# -2주차: 셸 스크립팅 (Shell Scripting) 및 환경 구축1. 주요 학습 및 실습 내용자동화의 필요성: 반복적인 작업(로그 분석, 백업, 삭제 등)을 스크립트 파일로 작성하여 오타 및 실행 실수를 방지하고 처리 속도를 극대화.Qwen & Ollama 로컬 LLM 환경:Ollama: 로컬 환경에서 대규모 언어 모델(LLM)을 실행하고 관리하는 오픈소스 플랫폼 및 백엔드 서버 (기본 포트: 11434).Qwen3:0.6b: 알리바바 클라우드에서 개발한 매개변수 약 6억 개(0.6B) 규모의 경량 대형 언어 모델.파이썬 기반 웹 인터페이스(chat.py)와 Ollama API를 연동하여 로컬 대화 서비스 구성.셸 스크립트 구성 요소:Shebang (#!/bin/bash): 해당 스크립트를 해석하고 실행할 스크립트 언어 인터프리터 경로를 OS에 명시.경로 고정 (cd "$(dirname "$0")" || exit 1): 스크립트가 실행되는 위치와 관계없이 스크립트 파일이 위치한 디렉터리를 기준(절대/상대 경로)으로 실행되도록 안전하게 이동.exec 명령어: 새로운 프로세스를 생성하지 않고 현재 셸 프로세스(PID)를 해당 프로그램으로 교체하여 실행.환경 변수 전달 (export): MODEL, WEB_PORT, APP_NAME 등을 설정하여 코드 수정 없이 실행 옵션 주입.조건문을 통한 오류 예외 처리: if ! command -v python3 >/dev/null 2>&1; then ... fi 구조를 활용하여 필수 실행 프로그램 존재 여부 검증 및 표준 오류(>&2) 출력.2. 주요 실행 명령어 및 설명Bash# Ollama 서비스 및 모델 확인
curl -fsS http://127.0.0.1:11434/api/tags    # Ollama 서버 상태 및 모델 목록 조회
ollama pull qwen3:0.6b                       # Qwen 0.6B 모델 다운로드
ollama list                                  # 다운로드된 로컬 모델 리스트 확인

# 스크립트 실행 권한 부여 및 실행
chmod u+x start.sh                           # 소유자에게 실행 권한(x) 부여
./start.sh                                   # 현재 디렉터리의 셸 스크립트 실행

# 환경 변수 전달 스크립트 예시
export MODEL="qwen3:0.6b"
export WEB_PORT="8080"
exec ./start.sh
3주차: 리눅스 자동화 기초 및 Git 협업 워크플로우1. 주요 학습 및 실습 내용리눅스 디렉터리 구조 및 파일 검색:주요 디렉터리: /etc (전역 설정 파일), /var/log (시스템 및 서비스 로그), ~ (사용자 홈 디렉터리).find: 파일 유형(-type f/d), 이름(-name) 기반의 파일시스템 검색.grep: 텍스트 내 특정 패턴/문자열 검색 (-R 하위디렉터리, -n 줄번호, -i 대소문자 무시, -v 반전).파이프 (|) & 리다이렉션 (>, >>): 앞 명령어의 표준 출력을 뒤 명령어의 표준 입력으로 전달하거나 파일로 저장/추가.프로세스 관리 및 자동화 스크립트:프로그램 vs 프로세스: 디스크에 저장된 실행 코드(프로그램)와 메모리에 로드되어 실행 중인 인스턴스(프로세스).ps aux / htop: 프로세스 목록 및 자원 사용량 실시간 모니터링.kill -15 (SIGTERM) (정상 종료) vs kill -9 (SIGKILL) (강제 종료).tar -czf (압축) 및 tar -xzf (해제)를 활용한 백업 자동화 스크립트 구축..gitignore: tar.gz 백업 파일, 로그 등 추적이 불필요한 파일의 Git 커밋 제외.Git/GitHub 협업 및 Conflict 해결:브랜치 전략: 독립적인 작업 공간(feature/기능명, fix/버그명)을 생성하여 메인 브랜치(main)의 안정성 유지.PR (Pull Request) & Merge: 코드 리뷰 후 메인 브랜치에 병합 (--squash 옵션으로 복수 커밋을 단일 커밋으로 깔끔하게 정리).Conflict (충돌): 동일 파일의 같은 위치가 서로 다르게 수정되었을 때 발생. 로컬에서 충돌 기호(<<<<<<<, =======, >>>>>>>) 확인 후 수동 편집, git add 및 커밋/푸시로 해결.2. 주요 실행 명령어 및 설명Bash# 파일 검색 및 로그 필터링
find . -type f -name "*.sh"                  # .sh 확장자 파일 전체 검색
grep -Rn "ERROR" /var/log 2>/dev/null        # 에러 로그 위치 및 줄 번호 검색
grep ERROR server.log | wc -l                # ERROR 발생 횟수 카운트

# 프로세스 확인 및 종료
ps aux | grep python                         # 실행 중인 파이썬 프로세스 검색
kill -9 <PID>                                # 특정 프로세스 강제 종료

# Git 브랜치 작업 및 충돌 해결
git switch -c feature/backup-log             # 새 기능 브랜치 생성 및 이동
git push -u origin feature/backup-log        # 원격 저장소에 브랜치 푸시
gh pr create --title "기능" --body "설명"     # CLI를 통한 PR 생성
gh pr merge --squash --delete-branch         # Squash Merge 수행 및 병합된 브랜치 삭제
4주차: 네트워크 기본 개념 및 웹 서버 실습1. 주요 학습 및 실습 내용IP 주소와 포트(Port):IP 주소: 네트워크상에서 장치를 식별하는 주소 (공인 IP vs 사설 IP: 192.168.x.x, 10.x.x.x, 172.16~31.x.x). 127.0.0.1 / localhost는 자기 자신(Loopback).포트(Port): 한 컴퓨터 안에서 실행 중인 특정 서비스/프로그램을 구분하는 번호 (예: SSH 22, HTTP 80, HTTPS 443, MySQL 3306, Redis 6379).0.0.0.0 바인딩: 외부의 모든 네트워크 인터페이스로부터 들어오는 연결 요청을 수신하도록 설정.DNS 및 네트워크 진단 도구:DNS (Domain Name System): 사람이 읽을 수 있는 도메인 이름을 컴퓨터가 사용하는 IP 주소로 변환.ip addr / hostname -I: 내 컴퓨터의 사설 IP 확인.ping: ICMP 패킷을 통한 네트워크 연결 가능 여부 및 응답 속도 측정.dig / nslookup: DNS 서버 질의 및 도메인 IP 정보 조회.curl: HTTP/HTTPS 요청을 보내 응답 상태 코드(200 OK, 404 Not Found, 500 Server Error 등) 및 헤더/본문 확인.ss -tlnp: 현재 시스템에서 LISTEN(대기) 상태로 포트를 점유 중인 프로세스 및 포트 번호 조회.포트 충돌 및 해결: 동일한 포트를 두 개 이상의 프로세스가 동시에 점유하려고 할 때 Address already in use 오류 발생. ss로 PID 확인 후 kill 처리하거나 다른 포트를 부여하여 해결.2. 주요 실행 명령어 및 설명Bash# IP 및 네트워크 조회
ip addr show eth0                            # 네트워크 인터페이스 IP 조회
dig cs.inhatc.ac.kr +short                   # 도메인의 IP 주소만 간단 조회
curl -I https://google.com                   # HTTP 응답 헤더 및 상태 코드 확인

# 포트 점유 상태 확인 및 프로세스 정리
ss -tlnp | grep 8080                         # 8080 포트를 점유 중인 프로세스 확인
pkill -f "http.server"                       # 파이썬 웹 서버 프로세스 전체 일괄 종료

# Python 간단 웹 서버 구동
python3 -m http.server 8080 --bind 0.0.0.0   # 8080 포트로 외부 접속 허용 웹 서버 실행
5주차: Docker 기초 및 GitHub Issue 기반 워크플로우1. 주요 학습 및 실습 내용가상머신(VM) vs Docker 컨테이너:가상머신(VM): 하이퍼바이저 기반 위에 각 VM마다 Guest OS를 전체 설치하여 구동 (무겁고 느림, 완전 격리).Docker 컨테이너: 호스트 OS의 커널(Kernel)을 공유하며, Namespace(시야/프로세스/네트워크 격리)와 cgroup(CPU/메모리 자원 제한)을 통해 프로세스 단위로 가볍고 빠르게 격리 구동.Docker 핵심 구성 요소 및 관계:Image (붕어빵 틀): 컨테이너를 생성하기 위한 실행 환경, 코드, 라이브러리가 포함된 읽기 전용(Read-Only) 템플릿.Container (붕어빵): 이미지로부터 생성되어 독립된 영역에서 실제 실행 중인 인스턴스 (쓰기 가능 계층이 추가됨, 컨테이너 삭제 시 쓰기 계층 데이터는 소멸).Registry: 이미지를 저장하고 공유하는 저장소 (Docker Hub, GHCR).Docker 주요 옵션:-d (Detached): 백그라운드 모드 실행.-p <호스트 포트>:<컨테이너 포트>: 포트 포워딩/개시 (예: -p 8080:80).-e <KEY=VALUE>: 컨테이너 내부 환경 변수 주입.-v <호스트 경로>:<컨테이너 경로>: 디렉터리/볼륨 마운트.--rm: 컨테이너 종료 시 자동 삭제.GitHub Issue 중심 DevOps 워크플로우:Issue 생성 (목표 및 체크리스트 작성) $\rightarrow$ 작업 브랜치 생성 (docs/issue-6-docker-start) $\rightarrow$ 실습 수행 및 README.md 기록 $\rightarrow$ PR 생성 본문에 Close #이슈번호 명시 $\rightarrow$ Squash Merge 후 Issue 자동 종료.2. 주요 실행 명령어 및 설명Bash# Nginx 컨테이너 실행 및 관리
docker run -d -p 8080:80 --name web nginx     # Nginx 컨테이너 백그라운드 구동
docker ps                                     # 현재 실행 중인 컨테이너 목록 조회
docker ps -a                                  # 종료된 컨테이너를 포함한 전체 목록 조회
docker logs -f web                            # 컨테이너 실시간 실행 로그 확인

# 컨테이너 내부 진입 및 삭제
docker exec -it web bash                      # 실행 중인 컨테이너 내부 Bash 셸 진입
docker rm -f web                              # 컨테이너 강제 정지 및 삭제

# 일회용 컨테이너 및 환경 변수 주입
docker run --rm python:3.12-slim python --version # 3.12 파이썬 일회용 실행
docker run -d -p 8083:80 -e MY_NAME=Kam --name env_test nginx # 환경변수 주입
6주차: Dockerfile 작성을 통한 이미지 빌드 및 배포1. 주요 학습 및 실습 내용Dockerfile 개념 및 명령어:애플리케이션 실행 환경을 코드로서 정의(IaC)하여 동일한 상태로 재현 가능하도록 만드는 레시피 파일.FROM: 기반(Base) 이미지 지정 (예: python:3.12-slim).WORKDIR: 컨테이너 내 작업 디렉터리 설정.COPY <원경로> <목적경로>: 호스트의 파일을 이미지 내부로 복사.RUN: 이미지 빌드 시점에 실행되어 레이어를 생성하는 명령어 (라이브러리 설치 등).CMD: 컨테이너 시작 시점에 기본으로 실행할 명령 (마지막 CMD만 적용).ENV: 컨테이너 내부 환경 변수 기본값 설정.EXPOSE: 사용 포트에 대한 문서화용 선언 (실제 포트 개시는 docker run -p로 수행).레이어 캐시(Layer Cache) 최적화:Dockerfile은 위에서부터 한 줄씩 레이어로 생성 및 캐싱됨.소스 코드는 자주 변경되는 반면, 패키지 목록(requirements.txt)은 거의 바뀌지 않으므로 COPY requirements.txt 및 RUN pip install을 코드 복사(COPY .)보다 앞에 배치하여 빌드 시간을 획기적으로 단축.설정 우선순위:docker run -e (컨테이너 실행 시 주입 - 최우선)Dockerfile 내 ENV (이미지 빌드 시 설정)애플리케이션 코드 내 기본값 (가장 낮음)이미지 레지스트리 공유 (GHCR):ghcr.io (GitHub Container Registry) 및 Docker Hub 로그인 후 docker tag, docker push로 이미지 배포. Package settings에서 Public 전환 필요.Multi-Architecture 빌드: docker buildx를 사용하여 x86_64(amd64) 및 Apple Silicon(arm64)을 동시 지원하는 이미지 작성.2. 주요 실행 명령어 및 설명Bash# Dockerfile 작성 및 이미지 빌드
docker build -t guestbook:v1 .                # 현재 디렉터리 기반 이미지 빌드
docker images                                 # 보유 중인 로컬 이미지 조회
docker rmi guestbook:bad                      # 불필요한 이미지 삭제

# GHCR 푸시 및 로그인
gh auth token | docker login ghcr.io -u <GitHub_ID> --password-stdin
docker tag guestbook:v1 ghcr.io/<GitHub_ID>/guestbook:v1
docker push ghcr.io/<GitHub_ID>/guestbook:v1

# Multi-Arch 이미지 빌드 및 푸시
docker buildx build --platform linux/amd64,linux/arm64 \
  -t ghcr.io/<GitHub_ID>/guestbook:v1 --push .
7주차: Docker Compose 멀티 컨테이너 오케스트레이션1. 주요 학습 및 실습 내용Docker Compose 필요성:단일 컨테이너 방식의 한계(단일 역할 원칙: Single Responsibility) 극복.앱(Web Server)과 DB(Redis) 등 다중 컨테이너 환경의 네트워크, 볼륨, 환경 변수를 compose.yaml 하나의 코드로 정의 및 관리.컨테이너 간 네트워크 통신 및 볼륨:도커 내부 DNS: 동일한 Compose 가상 네트워크 안에서 서비스 이름(web, redis)이 곧 DNS 주소가 되어 IP가 변경되어도 자동 추적/통신 가능.네트워크 격리: DB 컨테이너는 외부 ports 노출을 제거하고 internal expose만 선언하여 보안 강화.볼륨 (Volume): 컨테이너 제거 시 영구 데이터 소실을 막기 위해 외부 호스트에 데이터(redis-data) 보관. docker compose down 시 볼륨은 유지되며, docker compose down -v 시 데이터까지 삭제.컴포즈 구성 고도화:.env 파일 분리: Git 이력에 민감 데이터가 남지 않도록 .gitignore 처리 후 ${REDIS_PASSWORD} 형식을 통해 변수 분리.Healthcheck: 단순 컨테이너 시작(depends_on)을 넘어 서비스 수신 준비 상태(service_healthy)를 검증 후 연관 서비스 실행.Scale-Out & Nginx Load Balancing: --scale web=3 명령으로 웹 컨테이너 수평 확장. 앞단에 Nginx 리버스 프록시를 배치하여 8080 단일 포트 요청을 3개의 Web 컨테이너로 라운드로빈 분산.Watch (docker compose watch): 코드 수정 시 자동으로 빌드 및 컨테이너를 재배치.2. 주요 실행 명령어 및 설명Bash# Docker Compose 구동 및 관리
docker compose up -d                          # 백그라운드 멀티 컨테이너 구동
docker compose ps                             # 서비스 상태 조회 (healthy 여부 포함)
docker compose logs -f web                    # web 서비스 실시간 로그 확인
docker compose restart web                    # web 서비스만 재시작

# 스케일링 및 검증
docker compose up -d --scale web=3            # web 컨테이너 3개로 수평 확장
docker compose config --quiet                 # compose.yaml 파일 문법 및 설정 검증

# 서비스 종료 및 볼륨 삭제
docker compose down                           # 컨테이너 및 네트워크 정지/삭제 (데이터 유지)
docker compose down -v                        # 컨테이너, 네트워크 및 볼륨까지 완전 삭제
