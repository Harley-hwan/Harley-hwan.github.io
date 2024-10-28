---
layout: post
title: "Docker 실전 가이드: 설치와 환경 구성"
subtitle: "Docker 환경 구축과 실전 활용을 위한 단계별 가이드"
gh-repo: your-github-username/your-repo-name
gh-badge: [star, fork, follow]
tags: [docker, installation, configuration, devops, containerization]
comments: true
---

# Docker 실전 가이드: 설치와 환경 구성
- 최초 작성일: 2024년 10월 25일 (금)

<br>

## 목차
1. [Docker 설치 가이드](#docker-설치-가이드)
2. [초기 설정](#초기-설정)
3. [기본 명령어 실습](#기본-명령어-실습)
4. [실전 컨테이너 구성](#실전-컨테이너-구성)
5. [운영 환경 설정](#운영-환경-설정)

<br>

## Docker 설치 가이드

### Windows 환경 설치

1. **시스템 요구사항 확인**
   - Windows 10 Pro, Enterprise, Education (Build 16299 이상)
   - 또는 Windows 11
   - CPU 가상화 지원 (BIOS에서 활성화)
   - 최소 4GB RAM

2. **WSL2 설치**

```powershell
# PowerShell 관리자 모드에서 실행
wsl --install
```

3. **Docker Desktop 설치**
   - [Docker Hub](https://www.docker.com/products/docker-desktop) 에서 설치 파일 다운로드
   - 설치 중 'Use WSL 2 instead of Hyper-V' 옵션 선택
   - 설치 완료 후 시스템 재시작

<br>

### Linux(Ubuntu) 환경 설치

1. **시스템 업데이트 및 필수 패키지 설치**

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

```bash
# GPG 키 추가
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 저장소 추가
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

3. **Docker 엔진 설치**

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

4. **사용자 권한 설정**

```bash
# 현재 사용자를 docker 그룹에 추가
sudo usermod -aG docker $USER

# 변경사항 적용을 위한 재로그인
newgrp docker
```

## 초기 설정

### 1. Docker 서비스 상태 확인

```bash
# 서비스 상태 확인
sudo systemctl status docker

# 서비스 자동 시작 설정
sudo systemctl enable docker
```

<br>

### 2. Docker 네트워크 설정

1. **기본 네트워크 확인**

```bash
# 네트워크 목록 조회
docker network ls

# 기본 bridge 네트워크 상세 정보
docker network inspect bridge
```

2. **사용자 정의 네트워크 생성**

```bash
# 기본 bridge 네트워크 생성
docker network create --driver bridge my-network

# 특정 서브넷 지정하여 네트워크 생성
docker network create --driver bridge --subnet 172.18.0.0/16 my-custom-net
```

<br>

### 3. 저장소 설정

1. **Docker Hub 로그인**

```bash
# Docker Hub 인증
docker login

# 프라이빗 레지스트리 로그인
docker login private-registry.example.com
```

2. **레지스트리 미러링 설정**
   
```json
# /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://mirror.gcr.io",
    "https://registry.docker-cn.com"
  ]
}
```

<br>

## 기본 명령어 실습

### 1. 컨테이너 기본 조작

1. **컨테이너 실행과 관리**

```bash
# Nginx 컨테이너 실행
docker run -d --name web -p 80:80 nginx

# 컨테이너 상태 확인
docker ps

# 컨테이너 로그 확인
docker logs web

# 컨테이너 중지/시작
docker stop web
docker start web
```

2. **컨테이너 리소스 제한**

```bash
# 메모리 제한
docker run -d --name limited-web --memory=512m nginx

# CPU 제한
docker run -d --name cpu-limited --cpus=".5" nginx
```

<br>

### 2. 볼륨 관리

1. **볼륨 생성 및 사용**

```bash
# 볼륨 생성
docker volume create my-data

# 볼륨 마운트하여 컨테이너 실행
docker run -d \
  --name db \
  -v my-data:/var/lib/mysql \
  mysql:5.7
```

2. **바인드 마운트**

```bash
# 호스트 디렉토리 마운트
docker run -d \
  --name web \
  -v $(pwd)/html:/usr/share/nginx/html \
  nginx
```

<br>

## 실전 컨테이너 구성

### 1. 웹 서버 환경 구성

1. **Nginx 설정**

```bash
# 설정 파일 준비
mkdir -p nginx/conf.d
cat > nginx/conf.d/default.conf << EOF
server {
    listen 80;
    server_name localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html;
    }
}
EOF

# 컨테이너 실행
docker run -d \
  --name nginx \
  -p 80:80 \
  -v $(pwd)/nginx/conf.d:/etc/nginx/conf.d \
  nginx
```

<br>

### 2. 데이터베이스 환경 구성

1. **MySQL 설정**

```bash
# 환경 변수 파일 생성
cat > .env << EOF
MYSQL_ROOT_PASSWORD=mysecret
MYSQL_DATABASE=myapp
MYSQL_USER=myuser
MYSQL_PASSWORD=mypassword
EOF

# MySQL 컨테이너 실행
docker run -d \
  --name mysql \
  --env-file .env \
  -v mysql-data:/var/lib/mysql \
  mysql:5.7
```

### 3. 개발 환경 구성

1. **Node.js 개발 환경**

```bash
# 개발용 Dockerfile 작성
cat > Dockerfile.dev << EOF
FROM node:16

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

CMD ["npm", "run", "dev"]
EOF

# 개발 환경 실행
docker run -d \
  --name node-dev \
  -v $(pwd):/app \
  -p 3000:3000 \
  -e NODE_ENV=development \
  $(docker build -q -f Dockerfile.dev .)
```

<br>

2. **Python 개발 환경**

```bash
# 개발용 docker-compose.yml 작성
cat > docker-compose.dev.yml << EOF
version: '3'
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - .:/app
    ports:
      - "5000:5000"
    environment:
      - FLASK_ENV=development
      - FLASK_DEBUG=1
EOF

# 환경 실행
docker-compose -f docker-compose.dev.yml up -d
```

<br>

## 운영 환경 설정

### 1. 모니터링 구성

1. **기본 모니터링 설정**

```bash
# 컨테이너 리소스 사용량 모니터링
docker stats

# 특정 컨테이너 상세 정보
docker inspect container_id
```

2. **Prometheus + Grafana 설정**

```yaml
# docker-compose.monitoring.yml
version: '3'
services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana
    depends_on:
      - prometheus
    ports:
      - "3000:3000"
```

<br>

### 2. 로그 관리

1. **로그 드라이버 설정**

```json
# /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

2. **로그 수집 시스템 구성**

```yaml
# docker-compose.logging.yml
version: '3'
services:
  filebeat:
    image: elastic/filebeat:7.15.0
    volumes:
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
```

<br>

### 3. 자동화 구성

1. **컨테이너 자동 재시작**

```bash
# 항상 재시작
docker run -d --restart always nginx

# 오류 시에만 재시작
docker run -d --restart on-failure:5 nginx
```

2. **자동 업데이트 설정**

```yaml
# docker-compose.prod.yml
version: '3'
services:
  app:
    image: myapp:latest
    deploy:
      update_config:
        parallelism: 2
        delay: 10s
        order: start-first
```

<br>

### 4. 보안 설정

1. **기본 보안 강화**

```bash
# 권한 제한 실행
docker run -d \
  --cap-drop ALL \
  --cap-add NET_BIND_SERVICE \
  nginx

# 읽기 전용 루트 파일시스템
docker run -d \
  --read-only \
  --tmpfs /tmp \
  nginx
```

2. **네트워크 격리**

```bash
# 격리된 네트워크 생성
docker network create --driver bridge isolated_network

# 격리된 네트워크에서 컨테이너 실행
docker run -d --network isolated_network nginx
```

<br>

## 결론

이 가이드는 Docker의 실전적인 설치와 구성 방법을 다루었다. 환경 설정부터 운영까지 필요한 핵심 명령어와 설정을 포함하고 있으며, 실제 개발 및 운영 환경에서 바로 활용할 수 있다. 보안과 모니터링 같은 운영 필수 요소도 포함하여, 안정적인 Docker 환경을 구축할 수 있도록 구성하였다.
