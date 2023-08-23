---
layout: post
title: Ethernet Device Auto-Detection and IP Configuration
subtitle: linux, embedded, bash, shell, sh, ethernet, bash, script
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [linux, embedded, bash, shell, sh, ethernet, bash, script]
comments: true
--- 

# 이더넷 디바이스 자동 감지 및 IP 설정

- 최초 작성일: 2023년 8월 23일(수)

## 목차

[TOC]

<br/>

## 코드

```bash
#!/bin/bash
# golfzon pmk [2022.04.19]
# auto detect ethernet device and get name

ETH_IP="192.168.7.3"
ENX_STATUS=0

if [ -z $1 ]; then
 IP=3
else
 IP=$1
fi

IP1=`cat /GRadar_Info.xml|grep -i "RFCH"|gawk -F"<" '{print $2}'|gawk -F">" '{print $2}'`
echo "IP1" $IP1
ETH_IP="192.168.7."$IP1
echo "ETH_IP" $ETH_IP

function ETH_INIT()
{
  sudo echo " "  >> /etc/network/interfaces
  sudo echo -n "allow-hotplug " >> /etc/network/interfaces
  sudo echo $ETH_NAME >> /etc/network/interfaces
  sudo echo -n "auto " >> /etc/network/interfaces
  sudo echo $ETH_NAME >> /etc/network/interfaces
  sudo echo "iface" $ETH_NAME "inet static" >> /etc/network/interfaces
  sudo echo "address $ETH_IP" >> /etc/network/interfaces
  sudo echo "netmask 255.255.255.0" >> /etc/network/interfaces
  sudo ifconfig $ETH_NAME $ETH_IP netmask 255.255.255.0
}

function ETH_CHANGE_IP()
{
 sudo sed 's/192.168.7.1/192.168.7.2/g' /etc/network/interfaces
 sudo ifconfig $ETH_NAME $ETH_IP netmask 255.255.255.0
 echo "ETH_CHANGE_IP"
}

function INTERFACES_INIT()
{
sudo rm -rf /etc/network/interfaces
sleep 1
sudo echo "source-directory /etc/network/interfaces.d" >> /etc/network/interfaces
sudo echo -n "auto lo" >> /etc/network/interfaces
sudo echo "iface lo inet loopback" >> /etc/network/interfaces
sudo echo "allow-hotplug wlan0" >> /etc/network/inerfaces
sudo echo "auto wlan0" >> /etc/network/interfaces
sudo echo "    iface wlan0 inet static" >> /etc/network/interfaces
sudo echo "    pre-up iptables-restore < /usr/share/network-conf/ap/iptables.up.rules" >> /etc/network/interfaces
sudo echo "    address 192.168.8.1"  >> /etc/network/interfaces
sudo echo "    netmask 255.255.255.0" >> /etc/network/interfaces
sudo echo "    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf" >> /etc/network/interfaces
}

ETH_NAME=`ifconfig | grep -i "enx" | gawk -F" " '{print$1}'`
echo "ETH_NAME" $ETH_NAME

ETH_ADR=`ifconfig $ETH_NAME|grep "inet addr:"|gawk -F":" '{print $2}'|gawk -F" " '{print $1}'`
echo "ETH ADR" $ETH_ADR

GET_NAME=`cat /etc/network/interfaces|grep $ETH_NAME|grep "inet static"|gawk -F" " '{print $2}'`
echo "GET_NAME" $GET_NAME

GET_ADR=`cat /etc/network/interfaces|grep "192.168.7"|gawk -F" " '{print $2}'`
echo "GET_ADR" $GET_ADR

ENX_NAME=`sudo ip link show|grep "enx"|gawk -F": " '{print $2}'`
echo "ENX_NAME" $ENX_NAME
ENX_STATUS=`sudo ip link show|grep "enx"|gawk -F" " '{print $3}'|grep -n "UP" | cut -d: -f1`
echo "ENX_STATUS" $ENX_STATUS
ENX_DOWN=DOWN
ENX_UP=1
ENX_UNKNOW=UNKNOWN
echo "ENX_UNKNOW" $ENX_UNKNOW

if [ $ENX_STATUS == $ENX_UP ]; then
        echo "ENX is UP status..."
else
        sudo ifconfig $ENX_NAME up
        sleep 1
        ENX_STATUS=`sudo ip link show|grep "enx"|gawk -F" " '{print $9}'`
        echo "ENX up start:" $ENX_STATUS
fi

if [ -z $ETH_NAME ]; then
 echo "ETH_NAME is nothing"
 exit 1
else
 echo "run ETH_INIT_1"
 if [ -z $GET_NAME ]; then
  echo "run ETH_INIT_2"
  ETH_INIT
  exit 1
 else
   echo "run ETH_INIT_3"
   if [ $ETH_ADR == $ETH_IP ]; then
    echo "ETH_ADR  is same"
   else
    echo "run ETH_INIT_4"
    INTERFACES_INIT
    ETH_INIT
    exit 1
   fi
 fi
fi

```

## 설명

### 변수 설명
- `ETH_IP`: 설정할 이더넷 IP 주소. 기본값은 "192.168.7.3"이다.
- `ENX_STATUS`: enx 장치의 상태 정보.
- `IP`: 첫 번째 인수로 주어지는 IP. 주어지지 않으면 기본값 3을 사용.
- `IP1`: /GRadar_Info.xml 파일에서 "RFCH" 값을 기반으로 파싱한 IP의 끝부분 값.
- `ETH_NAME`: enx로 시작하는 이더넷 장치의 이름.
- `ETH_ADR`: 해당 이더넷 장치의 IP 주소.
- `GET_NAME`와 `GET_ADR`: /etc/network/interfaces 파일에서 파싱된 정보.
- `ENX_NAME`: "enx"로 시작하는 장치의 이름.
- `ENX_DOWN`, `ENX_UP`, `EN

X_UNKNOW`: enx 장치의 상태를 나타내는 문자열/숫자.

### 함수 설명
1. `ETH_INIT`: 이더넷 설정 초기화 및 주소 설정.
2. `ETH_CHANGE_IP`: 이더넷 IP 주소 변경 (현재 코드에서는 사용되지 않음).
3. `INTERFACES_INIT`: /etc/network/interfaces 파일 초기화 및 WLAN0 설정.

### 실행 흐름
1. 주어진 IP 인수를 기반으로 `ETH_IP` 설정.
2. /etc/network/interfaces 파일에서 설정 및 이더넷 장치 정보 파싱.
3. enx 장치의 상태 확인 및 필요시 활성화.
4. 이더넷 장치 이름 및 IP 주소 확인.
5. 조건에 따라 네트워크 설정 초기화 및 이더넷 IP 설정.

이 스크립트는 이더넷 장치의 이름과 상태를 감지하고, IP 주소를 설정하는 역할을 한다. /etc/network/interfaces 파일에 설정을 추가하거나 변경하는 기능도 포함되어 있다.
