---
layout: post
title: 스크립트를 통한 명령어 출력의 특정 필드 추출
subtitle: gawk, sudo, bash, shell, script, command line, awk
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [linux, gawk, sudo, bash, shell, script, command line, awk]
comments: true

---

# 스크립트를 통한 명령어 출력의 특정 필드 추출

- 최초 작성일: 2023년 8월 23일(수)

## 목차

[TOC]

<br/>

## 코드

```bash
sudo /home/pi/test/launch_jig -v | gawk 'NR == 1 {print $2}'
```

<br/>

## 설명

### **1. `sudo` 명령어**
- `sudo`: Super User DO의 약자로, 다른 사용자의 권한(대개는 root)으로 명령을 실행할 수 있게 해주는 명령어이다. 여기서는 스크립트나 프로그램을 root 권한으로 실행하기 위해 사용된다.

### **2. `/home/pi/test/launch_jig -v`**
- `/home/pi/test/launch_jig`: 사용자 pi의 홈 디렉터리 내의 `test` 폴더에 있는 `launch_jig`라는 이름의 스크립트나 실행 파일을 가리킨다..
- `-v`: `launch_jig` 프로그램에 전달되는 옵션 또는 인자로, 여기서 이 옵션은 "version" 정보를 출력하는 데 사용된다.

### **3. `|` (파이프)**
- `|`: 파이프는 Unix/Linux에서 한 명령어의 출력을 다른 명령어의 입력으로 전달하는 데 사용되는 연산자이다. 여기서는 `launch_jig -v`의 출력을 `gawk` 명령어의 입력으로 전달한다.

### **4. `gawk 'NR == 1 {print $2}'`**
- `gawk`: GNU awk의 줄임말로, 텍스트 처리와 데이터 추출 및 보고를 위한 프로그래밍 언어이다.
- `'NR == 1 {print $2}'`: 이 gawk 프로그램은 입력받은 텍스트의 첫 번째 줄(NR == 1)의 두 번째 필드($2)를 출력하도록 지시한다.

---

결론적으로, 이 명령어는 `launch_jig` 프로그램을 root 권한으로 실행하여 그 출력 중 첫 번째 줄의 두 번째 단어(또는 필드)를 추출하게 된다.

참고로, __"./launch_jig -v"__ 의 출력은 __"Version: 12.34.56"__ 와 같아서 앞에 Version: 을 제외한 값만 가져오기 위함이다.
