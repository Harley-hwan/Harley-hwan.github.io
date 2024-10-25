---
layout: post
title: "Docker 실전 가이드: 설치부터 운영까지"
subtitle: "Docker를 이용한 컨테이너 실전 활용 방법"
gh-repo: your-github-username/your-repo-name
gh-badge: [star, fork, follow]
tags: [docker, containerization, devops, docker-compose, deployment]
comments: true
---

# Docker 실전 가이드: 설치부터 운영까지
- 최초 작성일: 2024년 10월 25일 (금)
  
<br/>

## 목차
1. [Docker 설치](#docker-설치)
2. [기본 명령어](#기본-명령어)
3. [Dockerfile 작성](#dockerfile-작성)
4. [Docker Compose 활용](#docker-compose-활용)
5. [실전 예제](#실전-예제)
6. [문제 해결](#문제-해결)
   
</br>

## Docker 설치

### Windows에 설치
1. Docker Desktop 다운로드
   ```plaintext
   https://www.docker.com/products/docker-desktop
   ```
2. WSL2 설치 (Windows 10/11 필요)
3. Docker Desktop 설치 및 실행

### Linux(Ubuntu)에 설치
```bash
# 이전 버전 제거
sudo apt-get remove docker docker-engine docker.io

# 필요한 패키지 설치
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

# Docker 공식 GPG 키 추가
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Docker 저장소 추가
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

# Docker 설치
sudo apt-get update
sudo apt-get install docker-ce
```

</br>

## 기본 명령어

### 이미지 관련 명령어
```bash
# 이미지 검색
docker search nginx

# 이미지 다운로드
docker pull nginx

# 이미지 목록 확인
docker images

# 이미지 삭제
docker rmi nginx
```

### 컨테이너 관련 명령어
```bash
# 컨테이너 실행
docker run -d -p 80:80 nginx

# 실행 중인 컨테이너 목록
docker ps

# 모든 컨테이너 목록
docker ps -a

# 컨테이너 중지
docker stop container_id

# 컨테이너 삭제
docker rm container_id

# 컨테이너 로그 보기
docker logs container_id
```

</br>

## Dockerfile 작성

### Node.js 애플리케이션 예제
```dockerfile
# 베이스 이미지 선택
FROM node:14

# 작업 디렉토리 설정
WORKDIR /app

# package.json 복사
COPY package*.json ./

# 의존성 설치
RUN npm install

# 소스 코드 복사
COPY . .

# 포트 설정
EXPOSE 3000

# 실행 명령
CMD ["npm", "start"]
```

### Python 웹 애플리케이션 예제
```dockerfile
FROM python:3.9

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

</br>

## Docker Compose 활용

### 기본 docker-compose.yml
```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - .:/app
    environment:
      - NODE_ENV=development
  
  db:
    image: mongodb
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db

volumes:
  mongodb_data:
```

### Docker Compose 명령어
```bash
# 서비스 시작
docker-compose up -d

# 서비스 중지
docker-compose down

# 서비스 로그 보기
docker-compose logs

# 서비스 재시작
docker-compose restart
```

</br>

## 실전 예제

### React + Node.js + MongoDB 스택 구성
```yaml
version: '3'
services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    depends_on:
      - backend

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "5000:5000"
    environment:
      - MONGODB_URI=mongodb://db:27017/myapp
    depends_on:
      - db

  db:
    image: mongo:latest
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db

volumes:
  mongodb_data:
```

### 프로덕션 배포 설정
```dockerfile
# 멀티 스테이지 빌드 예제
FROM node:14 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

</br>

## 문제 해결

### 일반적인 문제와 해결방법

1. **컨테이너 접속이 안될 때**
```bash
# 네트워크 확인
docker network ls
docker network inspect network_name

# 포트 매핑 확인
docker port container_id
```

2. **컨테이너가 자동 종료될 때**
```bash
# 로그 확인
docker logs container_id

# 대화형 모드로 실행
docker run -it image_name /bin/bash
```

3. **디스크 공간 부족**
```bash
# 미사용 리소스 정리
docker system prune -a

# 볼륨 정리
docker volume prune
```

4. **메모리 문제**
```bash
# 메모리 제한 설정
docker run -m 512m image_name

# 메모리 사용량 모니터링
docker stats
```

### 성능 최적화 팁

1. **이미지 크기 최적화**
   - 다단계 빌드 사용
   - .dockerignore 파일 활용
   - 경량 베이스 이미지 사용

2. **캐시 활용**
   - 빌드 캐시 활용
   - 레이어 최소화
   - 의존성 설치 레이어 분리

</br>
