---
layout: post
title: "Docker 실전 가이드: 설치와 환경 구성"
subtitle: "Docker 환경 구축과 실전 활용을 위한 종합 가이드"
gh-repo: your-github-username/your-repo-name
gh-badge: [star, fork, follow]
tags: [docker, installation, configuration, devops, containerization]
comments: true
---

# Docker 실전 가이드: 설치와 환경 구성
- 최초 작성일: 2024년 10월 25일 (금)
- Docker 버전: 24.0.6
- Docker Compose 버전: 2.21.0

<br>

## 목차
1. [Docker 설치 가이드](#docker-설치-가이드)
   - Windows 환경
   - Linux 환경
   - Mac 환경
2. [초기 설정](#초기-설정)
   - 서비스 설정
   - 네트워크 설정
   - 저장소 설정
   - 보안 초기 설정
3. [기본 명령어 실습](#기본-명령어-실습)
   - 컨테이너 생명주기 관리
   - 이미지 관리
   - 볼륨과 네트워크 관리
   - 리소스 모니터링
4. [실전 컨테이너 구성](#실전-컨테이너-구성)
   - 웹 애플리케이션 스택
   - 데이터베이스 클러스터
   - 캐시 서버
   - 로드 밸런서
5. [운영 환경 설정](#운영-환경-설정)
   - 모니터링과 로깅
   - CI/CD 파이프라인
   - 보안 강화
   - 백업과 복구
6. [문제 해결 가이드](#문제-해결-가이드)
   - 일반적인 문제 해결
   - 성능 최적화
   - 디버깅 방법

<br>

## Docker 설치 가이드

### Windows 환경 설치
Windows 환경에서 Docker를 설치하기 위해서는 WSL2(Windows Subsystem for Linux 2)가 필요하다. WSL2는 완전한 Linux 커널을 제공하여 Docker가 네이티브 Linux처럼 동작할 수 있게 해준다.

1. **시스템 요구사항 확인**
   - Windows 10 Pro, Enterprise, Education (Build 16299 이상)
   - 또는 Windows 11
   - CPU 가상화 지원 (BIOS에서 활성화 필요)
   - 최소 4GB RAM (8GB 이상 권장)
   - 최소 50GB 여유 디스크 공간

2. **WSL2 설치 및 설정**

```powershell
# PowerShell 관리자 모드에서 실행
wsl --install

# WSL2를 기본 버전으로 설정
wsl --set-default-version 2

# WSL 상태 확인
wsl -l -v
```

3. **Docker Desktop 설치**
   - Docker Hub에서 최신 버전 다운로드
   - 설치 중 'Use WSL 2 instead of Hyper-V' 옵션 선택
   - 설치 완료 후 시스템 재시작

설치 확인:
```powershell
# Docker 버전 확인
docker --version

# Docker Compose 버전 확인
docker-compose --version

# Docker 시스템 정보 확인
docker info
```

```markdown
### Linux(Ubuntu) 환경 설치
Linux 환경에서는 패키지 관리자를 통해 Docker를 설치한다. 여기서는 Ubuntu 22.04 LTS를 기준으로 설명한다.

1. **시스템 업데이트 및 필수 패키지 설치**
   패키지 목록을 최신화하고 HTTPS를 통한 저장소 사용을 위한 패키지들을 설치한다.

```bash
# 시스템 업데이트
sudo apt update
sudo apt upgrade -y

# 필수 패키지 설치
sudo apt install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

2. **Docker 공식 저장소 설정**
   Docker의 공식 GPG 키를 추가하고 저장소를 설정한다. 이는 패키지의 신뢰성을 보장한다.

```bash
# GPG 키 추가
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 저장소 추가
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

3. **Docker 엔진 설치**
   최신 버전의 Docker 엔진과 관련 도구들을 설치한다.

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

4. **사용자 권한 설정**
   일반 사용자가 sudo 없이 Docker를 사용할 수 있도록 설정한다.

```bash
# 현재 사용자를 docker 그룹에 추가
sudo usermod -aG docker $USER

# 변경사항 적용을 위한 재로그인
newgrp docker
```

<br>

### Mac 환경 설치
Mac 환경에서는 Docker Desktop for Mac을 통해 Docker를 설치하고 관리할 수 있다.

1. **시스템 요구사항**
   - macOS 11 이상
   - Apple Silicon(M1/M2) 또는 Intel 프로세서
   - 최소 4GB RAM (8GB 이상 권장)
   - 최소 50GB 여유 디스크 공간

2. **설치 과정**

```bash
# Homebrew를 통한 설치
brew install --cask docker

# 또는 Docker 웹사이트에서 Docker Desktop for Mac 다운로드
# https://www.docker.com/products/docker-desktop
```

3. **설치 확인 및 초기 설정**

```bash
# Docker 버전 확인
docker --version

# 테스트 컨테이너 실행
docker run hello-world
```

<br>

## 초기 설정

### 1. Docker 서비스 설정
Docker 데몬의 기본 설정을 최적화하고 보안을 강화한다.

1. **데몬 설정 파일 생성/수정**

```json
# /etc/docker/daemon.json
{
    "storage-driver": "overlay2",
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "10m",
        "max-file": "3"
    },
    "default-ulimits": {
        "nofile": {
            "Name": "nofile",
            "Hard": 64000,
            "Soft": 64000
        }
    },
    "live-restore": true,
    "max-concurrent-downloads": 10,
    "max-concurrent-uploads": 10,
    "registry-mirrors": [
        "https://mirror.gcr.io"
    ]
}
```

2. **서비스 자동 시작 설정**

```bash
# 서비스 상태 확인
sudo systemctl status docker

# 부팅 시 자동 시작 설정
sudo systemctl enable docker

# 서비스 재시작
sudo systemctl restart docker
```

```markdown
### 2. 네트워크 설정
컨테이너 간 통신과 외부 네트워크 연결을 위한 Docker 네트워크 설정을 구성한다.

1. **기본 네트워크 이해와 확인**
Docker는 기본적으로 bridge, host, none 세 가지 네트워크를 제공한다.

```bash
# 네트워크 목록 확인
docker network ls

# 기본 bridge 네트워크 상세 정보
docker network inspect bridge
```

2. **사용자 정의 네트워크 생성**
애플리케이션별로 격리된 네트워크를 생성하여 보안성을 높인다.

```bash
# 기본 bridge 네트워크 생성
docker network create \
    --driver bridge \
    --subnet 172.18.0.0/16 \
    --gateway 172.18.0.1 \
    app_network

# 네트워크 암호화 옵션 추가 (Swarm mode)
docker network create \
    --driver overlay \
    --opt encrypted \
    --attachable \
    secure_network
```

3. **네트워크 정책 설정**
컨테이너 간 통신 제어를 위한 네트워크 정책을 설정한다.

```bash
# 컨테이너 간 통신 제한
docker run -d \
    --name webapp \
    --network app_network \
    --network-alias webapp \
    --cap-drop NET_RAW \
    nginx

# 특정 포트만 노출
docker run -d \
    --name api \
    --network app_network \
    --publish 8080:8080 \
    api-server
```

<br>

### 3. 볼륨 관리
데이터 영속성과 백업을 위한 Docker 볼륨 설정을 구성한다.

1. **볼륨 종류별 생성 및 관리**

```bash
# 이름이 있는 볼륨 생성
docker volume create \
    --driver local \
    --opt type=nfs \
    --opt o=addr=192.168.1.1,rw \
    --opt device=:/path/to/dir \
    nfs_storage

# 볼륨 상세 정보 확인
docker volume inspect nfs_storage

# 미사용 볼륨 정리
docker volume prune -f
```

2. **백업 전략 설정**
중요 데이터의 안전한 보관을 위한 볼륨 백업 설정.

```bash
# 볼륨 백업
docker run --rm \
    --volumes-from source_container \
    -v $(pwd):/backup \
    alpine \
    tar cvf /backup/backup.tar /data

# 볼륨 복원
docker run --rm \
    --volumes-from target_container \
    -v $(pwd):/backup \
    alpine \
    tar xvf /backup/backup.tar
```

3. **볼륨 권한 관리**

```bash
# 볼륨 마운트 시 권한 설정
docker run -d \
    --name db \
    --mount type=volume,source=db_data,target=/var/lib/mysql,volume-opt=o=uid=999 \
    mysql:8.0

# 읽기 전용 볼륨 마운트
docker run -d \
    --name webapp \
    --mount type=volume,source=config,target=/etc/nginx/conf.d,readonly \
    nginx
```

<br>

### 4. 보안 초기 설정
Docker 환경의 보안을 강화하기 위한 기본 설정을 구성한다.

1. **컨테이너 보안 설정**

```bash
# 권한 제한 실행
docker run -d \
    --name restricted_app \
    --cap-drop ALL \
    --cap-add NET_BIND_SERVICE \
    --security-opt no-new-privileges \
    --read-only \
    --tmpfs /tmp \
    nginx

# 리소스 제한 설정
docker run -d \
    --name limited_app \
    --memory="512m" \
    --memory-swap="512m" \
    --cpus=".5" \
    --pids-limit=200 \
    nginx
```

```markdown
## 기본 명령어 실습

### 1. 컨테이너 생명주기 관리
컨테이너의 생성, 시작, 중지, 삭제 등 기본적인 생명주기를 관리하는 명령어들을 살펴본다.

1. **컨테이너 생성과 실행**
실행 옵션에 따라 컨테이너의 동작이 달라진다는 점을 이해하는 것이 중요하다.

```bash
# 기본 실행 (포그라운드)
docker run nginx

# 백그라운드 실행 (-d)
docker run -d --name webserver nginx

# 환경변수 설정 (-e)
docker run -d \
    --name myapp \
    -e "NODE_ENV=production" \
    -e "PORT=3000" \
    node:16

# 포트 매핑 (-p)
docker run -d \
    --name webapp \
    -p 80:80 \     # 호스트:컨테이너
    -p 443:443 \
    nginx
```

2. **컨테이너 상태 관리**
컨테이너의 현재 상태를 확인하고 제어하는 명령어들이다.

```bash
# 실행 중인 컨테이너 목록
docker ps

# 모든 컨테이너 목록 (중지된 것 포함)
docker ps -a

# 컨테이너 상세 정보
docker inspect webapp

# 컨테이너 로그 확인
docker logs -f --tail=100 webapp

# 컨테이너 리소스 사용량
docker stats webapp
```

3. **컨테이너 제어**
실행 중인 컨테이너의 동작을 제어한다.

```bash
# 컨테이너 중지
docker stop webapp

# 컨테이너 시작
docker start webapp

# 컨테이너 재시작
docker restart webapp

# 컨테이너 일시 중지
docker pause webapp

# 컨테이너 일시 중지 해제
docker unpause webapp

# 컨테이너 강제 종료
docker kill webapp

# 컨테이너 삭제 (중지 상태에서만)
docker rm webapp

# 실행 중인 컨테이너 강제 삭제
docker rm -f webapp
```

### 2. 이미지 관리
Docker 이미지의 다운로드, 생성, 수정, 삭제 등을 관리하는 명령어들을 알아본다.

1. **이미지 검색과 다운로드**

```bash
# Docker Hub 이미지 검색
docker search ubuntu

# 이미지 다운로드 (태그 지정)
docker pull ubuntu:22.04

# 이미지 목록 확인
docker images

# 이미지 상세 정보
docker inspect ubuntu:22.04

# 이미지 히스토리 확인
docker history ubuntu:22.04
```

2. **이미지 생성과 수정**

```bash
# 컨테이너로부터 이미지 생성
docker commit \
    --author "John Doe" \
    --message "Added nginx configuration" \
    running_container \
    my-nginx:v1.0

# Dockerfile을 통한 이미지 빌드
docker build \
    --tag myapp:1.0 \
    --build-arg VERSION=1.0 \
    --no-cache \
    .

# 이미지 태그 변경
docker tag myapp:1.0 registry.example.com/myapp:1.0
```

```markdown
3. **이미지 공유와 저장**

```bash
# 이미지 저장소 푸시
docker push registry.example.com/myapp:1.0

# 이미지를 파일로 저장
docker save -o myapp.tar myapp:1.0

# 저장된 이미지 파일 로드
docker load -i myapp.tar

# 불필요한 이미지 정리
docker image prune -a --filter "until=24h"
```

### 3. 리소스 모니터링

1. **기본 모니터링**

```bash
# 실시간 리소스 사용량
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"

# 특정 컨테이너 상세 정보
docker inspect \
    --format='{{.State.Status}} {{.State.Running}}' \
    container_name
```

2. **고급 모니터링 설정**

```yaml
# docker-compose.monitoring.yml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.retention.time=15d'

  grafana:
    image: grafana/grafana:latest
    volumes:
      - grafana-data:/var/lib/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=secure_password
      - GF_USERS_ALLOW_SIGN_UP=false

volumes:
  grafana-data:
```

## 실전 컨테이너 구성

### 1. 웹 애플리케이션 스택

```yaml
# docker-compose.webapp.yml
version: '3.8'
services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./certbot/conf:/etc/letsencrypt
    depends_on:
      - webapp

  webapp:
    build: 
      context: ./webapp
      dockerfile: Dockerfile
    environment:
      - NODE_ENV=production
      - DB_HOST=db
    depends_on:
      - db
      - redis

  db:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    secrets:
      - db_password

  redis:
    image: redis:alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### 2. CI/CD 파이프라인 구성

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

test:
  stage: test
  script:
    - docker pull $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker run --rm $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA npm test

deploy:
  stage: deploy
  script:
    - docker stack deploy -c docker-compose.prod.yml myapp
```

### 3. 백업 및 복구 시스템

```bash
#!/bin/bash
# backup.sh

# 데이터베이스 백업
docker exec db pg_dump -U postgres myapp > backup_$(date +%Y%m%d).sql

# 볼륨 백업
docker run --rm \
    --volumes-from db \
    -v $(pwd)/backups:/backups \
    alpine \
    tar czf /backups/volumes_$(date +%Y%m%d).tar.gz /var/lib/postgresql/data
```

## 문제 해결 가이드

### 1. 일반적인 문제와 해결

1. **컨테이너 시작 실패**
```bash
# 로그 확인
docker logs --tail 50 container_name

# 설정 확인
docker inspect container_name
```

2. **네트워크 문제**
```bash
# 네트워크 연결 테스트
docker run --rm \
    --network container:problem_container \
    nicolaka/netshoot \
    curl -v target_host
```

3. **리소스 부족**
```bash
# 리소스 사용량 모니터링
docker stats

# 불필요한 리소스 정리
docker system prune -af
```

### 2. 성능 최적화

1. **이미지 최적화**

```dockerfile
# 다단계 빌드 예시
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
```

2. **컨테이너 최적화**

```bash
# 리소스 제한 설정
docker run -d \
    --name optimized_app \
    --cpus=".5" \
    --memory="512m" \
    --memory-swap="512m" \
    your-app:latest
```

## 결론
이 가이드에서는 Docker의 설치부터 운영까지 필요한 실전적인 내용을 다루었다. 컨테이너화된 애플리케이션의 개발, 배포, 운영에 필요한 핵심 개념과 명령어를 포함하고 있으며, 실제 프로덕션 환경에서 발생할 수 있는 문제들의 해결 방법도 다루었다.
