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
   - Windows 환경 설치
   - Linux 환경 설치
   - Mac 환경 설치
2. [초기 설정](#초기-설정)
   - Docker 서비스 설정
   - 네트워크 설정
   - 볼륨 관리
   - 보안 초기 설정
3. [기본 명령어 실습](#기본-명령어-실습)
   - 컨테이너 생명주기 관리
   - 이미지 관리
   - 리소스 모니터링
4. [실전 컨테이너 구성](#실전-컨테이너-구성)
   - 웹 애플리케이션 스택 구성
   - CI/CD 파이프라인 구성
   - 백업 및 복구 시스템
5. [문제 해결 가이드](#문제-해결-가이드)
   - 일반적인 문제와 해결
   - 성능 최적화

   &nbsp;

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

   &nbsp;

2. **WSL2 설치 및 설정**

   ```powershell
   # PowerShell 관리자 모드에서 실행
   wsl --install
   
   # WSL2를 기본 버전으로 설정
   wsl --set-default-version 2
   
   # WSL 상태 확인
   wsl -l -v
   ```

   &nbsp;

3. **Docker Desktop 설치**
   - Docker Hub에서 최신 버전 다운로드
   - 설치 중 'Use WSL 2 instead of Hyper-V' 옵션 선택
   - 설치 완료 후 시스템 재시작

   &nbsp;
   
설치 확인:
   ```powershell
   # Docker 버전 확인
   docker --version
   
   # Docker Compose 버전 확인
   docker-compose --version
   
   # Docker 시스템 정보 확인
   docker info
   ```

   &nbsp;

<br>

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

   &nbsp;

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

   &nbsp;

3. **Docker 엔진 설치**
   최신 버전의 Docker 엔진과 관련 도구들을 설치한다.
   ```bash
   sudo apt update
   sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
   ```

   &nbsp;

4. **사용자 권한 설정**
   일반 사용자가 sudo 없이 Docker를 사용할 수 있도록 설정한다.

   ```bash
   # 현재 사용자를 docker 그룹에 추가
   sudo usermod -aG docker $USER
   
   # 변경사항 적용을 위한 재로그인
   newgrp docker
   ```

   &nbsp;

<br>

### Mac 환경 설치
Mac 환경에서는 Docker Desktop을 통해 Docker를 설치하고 관리한다.

1. **시스템 요구사항**
   - macOS 11 이상
   - Apple Silicon(M1/M2) 또는 Intel 프로세서
   - 최소 4GB RAM (8GB 이상 권장)
   - 최소 50GB 여유 디스크 공간

   &nbsp;

2. **설치 과정**
   ```bash
   # Homebrew를 통한 설치
   brew install --cask docker
   
   # 또는 Docker 웹사이트에서 Docker Desktop for Mac 다운로드 후 설치
   # https://www.docker.com/products/docker-desktop
   ```

   &nbsp;

3. **설치 확인 및 초기 설정**
   ```bash
   # Docker 버전 확인
   docker --version
   
   # 테스트 컨테이너 실행
   docker run hello-world
   ```

   &nbsp;

<br>

## 초기 설정

### Docker 서비스 설정
Docker 데몬의 기본 설정을 최적화하고 보안을 강화한다.

1. **데몬 설정 파일 구성**
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
       "max-concurrent-uploads": 10
   }
   ```
   
   &nbsp;

2. **서비스 활성화 및 시작**
   ```bash
   # 서비스 상태 확인
   sudo systemctl status docker
   
   # 부팅 시 자동 시작 설정
   sudo systemctl enable docker
   
   # 서비스 재시작
   sudo systemctl restart docker
   ```

   &nbsp;

<br>

### 네트워크 설정
Docker의 네트워크는 컨테이너 간 통신과 외부 연결을 관리한다.

1. **네트워크 드라이버 이해**
   - bridge: 단일 호스트 내 컨테이너 간 통신
   - host: 호스트의 네트워크 직접 사용
   - overlay: 다중 호스트 간 컨테이너 통신
   - none: 네트워크 기능 비활성화

   &nbsp;

2. **사용자 정의 네트워크 생성**
   ```bash
   # 애플리케이션별 격리된 네트워크 생성
   docker network create \
       --driver bridge \
       --subnet 172.18.0.0/16 \
       --gateway 172.18.0.1 \
       app_network
   
   # 암호화된 오버레이 네트워크 생성
   docker network create \
       --driver overlay \
       --opt encrypted \
       secure_network
   ```

   &nbsp;

<br>

## 기본 명령어 실습

### 컨테이너 생명주기 관리
컨테이너의 전체 라이프사이클을 관리하는 기본 명령어들이다.

1. **컨테이너 실행과 관리**

   ```bash
   # 기본 컨테이너 실행
   docker run nginx
   
   # 백그라운드 실행 (-d)와 포트 매핑 (-p)
   docker run -d --name webserver -p 80:80 nginx
   
   # 환경변수 설정과 볼륨 마운트
   docker run -d \
       --name webapp \
       -e "NODE_ENV=production" \
       -v $(pwd):/app \
       node:16
   ```

   &nbsp;

3. **컨테이너 상태 관리**
   ```bash
   # 실행 중인 컨테이너 목록
   docker ps
   
   # 모든 컨테이너 목록 (중지된 것 포함)
   docker ps -a
   
   # 컨테이너 상세 정보
   docker inspect webapp
   
   # 실시간 로그 확인
   docker logs -f webapp
   ```

   &nbsp;

4. **컨테이너 리소스 제어**
   ```bash
   # 메모리 제한
   docker run -d \
       --name limited_app \
       --memory="512m" \
       --memory-swap="512m" \
       nginx
   
   # CPU 제한
   docker run -d \
       --name cpu_limited \
       --cpus=".5" \
       nginx
   ```

   &nbsp;

<br>

### 이미지 관리
Docker 이미지의 생성, 저장, 배포를 위한 명령어들이다.

&nbsp;

1. **이미지 기본 관리**
   ```bash
   # 이미지 검색
   docker search nginx
   
   # 이미지 다운로드
   docker pull nginx:latest
   
   # 이미지 목록 확인
   docker images
   
   # 이미지 삭제
   docker rmi nginx:latest
   ```

   &nbsp;

2. **커스텀 이미지 생성**
   ```dockerfile
   # Dockerfile 예시
   FROM node:16-alpine
   
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   
   EXPOSE 3000
   CMD ["npm", "start"]
   ```

   ```bash
   # 이미지 빌드
   docker build -t myapp:1.0 .
   
   # 이미지 태그 설정
   docker tag myapp:1.0 registry.example.com/myapp:1.0
   ```

   &nbsp;

<br>

## 실전 컨테이너 구성

### 웹 애플리케이션 스택 구성
   ```yaml
   # docker-compose.yml
   version: '3.8'
   services:
     nginx:
       image: nginx:alpine
       ports:
         - "80:80"
       volumes:
         - ./nginx/conf.d:/etc/nginx/conf.d
       depends_on:
         - webapp
   
     webapp:
       build: ./webapp
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
   
     redis:
       image: redis:alpine
       volumes:
         - redis_data:/data
   
   volumes:
     postgres_data:
     redis_data:
   ```

   &nbsp;

### CI/CD 파이프라인 구성
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

   &nbsp;

### 백업 및 복구 시스템
   ```bash
   #!/bin/bash
   # backup.sh
   
   # 데이터베이스 백업
   docker exec db pg_dump -U postgres myapp > backup_$(date +%Y%m%d).sql
   
   # 볼륨 데이터 백업
   docker run --rm \
       --volumes-from db \
       -v $(pwd)/backups:/backups \
       alpine \
       tar czf /backups/volumes_$(date +%Y%m%d).tar.gz /var/lib/postgresql/data
   ```

   &nbsp;
   
<br>

## 문제 해결 가이드

### 일반적인 문제와 해결
1. **컨테이너 시작 실패**

   ```bash
   # 로그 확인
   docker logs --tail 50 container_name
   
   # 컨테이너 상태 확인
   docker inspect container_name
   ```

   &nbsp;

2. **성능 최적화**
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

   &nbsp;

<br>

## 결론
이 가이드는 Docker의 설치부터 운영까지 필요한 실전적인 내용을 다루었다. 기본적인 설치와 설정부터 시작하여 실전 환경 구성, 문제 해결까지 포괄적으로 다루어 실무에서 바로 활용할 수 있도록 구성하였다.
