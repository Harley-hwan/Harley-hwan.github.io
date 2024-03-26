---
layout: post
title: SFTP Connect to Ubuntu with FileZilla
subtitle: ifconfig, sudo, vmware, vm, ubuntu, linux, ip addr, ens33
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [ifconfig, sudo, vmware, vm, ubuntu, linux, ip addr, ens33]
comments: true
---

# SFTP Connect to Ubuntu with Filezilla

- 최초 작성일: 2024년 3월 26일(화)

## 목차

[TOC]

<br/>

---
### ens33의 역할과 의미
- `ens33`은 Ubuntu 시스템 내에서의 네트워크 인터페이스 이름이다. VMware 또는 다른 가상 환경에서 생성된 가상 네트워크 어댑터를 나타낸다.
- 이 네트워크 인터페이스를 통해 가상 머신이 외부 네트워크와 통신한다. 즉, 인터넷 접속이나 다른 네트워크 기기와의 연결에 필수적이다.
- `ens33`가 'DOWN' 상태라면 네트워크 연결이 비활성화된 상태이므로, 인터넷이나 외부 네트워크에 접속할 수 없다.

### VMware Ubuntu 환경에서 SFTP 연결 설정하기
1. **네트워크 인터페이스 확인 및 활성화:**
   - Ubuntu 터미널을 열고 `ip addr` 명령어를 실행하여 현재 네트워크 인터페이스 상태를 확인한다.
      ```bash
          ip addr
      ```
    - **예시화면:**
      ```bash
          2: ens33: <BROADCAST,MULTICAST,DOWN> mtu 1500 qdisc mq state DOWN group default qlen 1000
              link/ether 00:0c:29:ff:ee:dd brd ff:ff:ff:ff:ff:ff
      ```
   - 여기서 `ens33`이 'DOWN' 상태로 나타난다면, 네트워크가 비활성화된 상태이다.
   - `ens33`을 활성화하기 위해 다음 명령어를 입력한다:
      ```bash
          sudo ip link set ens33 up
      ```
   - 이후 다시 `ip addr`를 실행하여 `ens33`이 'UP' 상태인지 확인한다.
      ```
          2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
              link/ether 00:0c:29:ff:ee:dd brd ff:ff:ff:ff:ff:ff
      ```

2. **IP 주소 확인:**
   - 네트워크 인터페이스가 'UP' 상태일 때, `ip addr` 명령어를 통해 할당받은 IP 주소를 확인한다. 이 IP 주소는 FileZilla를 통한 SFTP 연결에 사용된다.
   - 아래의 출력 예시를 보면 **192.168.74.128** 라는 ip를 할당받은 것을 볼 수 있다.
   - **ifconfig** 명령어로도 확인이 가능.
      ```
          2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
              link/ether 00:0c:29:45:ba:a6 brd ff:ff:ff:ff:ff:ff
              altname enp2s1
              inet 192.168.74.128/24 brd 192.168.74.255 scope global dynamic ens33
                 valid_lft 1693sec preferred_lft 1693sec
              inet6 fe80::20c:29ff:fe45:baa6/64 scope link 
                 valid_lft forever preferred_lft forever
      ```
      
3. **FileZilla 설정:**
   - FileZilla 클라이언트를 열고 '파일' > '사이트 관리자'로 이동한다.
   - '새 사이트'를 클릭하고, 적절한 이름을 지정한다.
   - 다음 설정을 입력한다:
     - **호스트:** Ubuntu 가상 머신에 할당된 IP 주소.
     - **포트:** `22` (SSH와 SFTP의 기본 포트).
     - **프로토콜:** 'SFTP - SSH 파일 전송 프로토콜'.
     - **로그온 유형:** '일반'.
     - **사용자:** Ubuntu 시스템의 사용자 이름.
     - **비밀번호:** 해당 사용자의 비밀번호.
   - '연결'을 클릭하여 SFTP 접속을 시도한다.

<br/>

이 과정을 통해 VMware Ubuntu 환경에 있는 가상 머신에 SFTP로 안전하게 파일을 전송하고 관리할 수 있다. 

`ens33`과 같은 네트워크 인터페이스를 활성화하고 올바르게 설정하는 것이 중요하며, 이는 연결의 기본적인 요구 사항이다.

