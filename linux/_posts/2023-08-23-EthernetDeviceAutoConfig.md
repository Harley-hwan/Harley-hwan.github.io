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

<br/>

<br/>

## 추가 설명 (자세히)

<br/>

### `/Info.xml` 파일에서 "RFCH" 문자열 추출

```bash
IP1=`cat /Info.xml|grep -i "RFCH"|gawk -F"<" '{print $2}'|gawk -F">" '{print $2}'`
```

1. **`cat /Info.xml`**:
    - `cat`: 주어진 파일의 내용을 화면에 출력하는 명령어.
    - `/Info.xml`: `cat`에 전달된 파일 경로.
    - 결과: `/Info.xml`의 내용이 출력됨.

2. **`grep -i "RFCH"`**:
    - `grep`: 텍스트에서 특정 패턴을 검색하는 명령어.
    - `-i`: 대소문자 구분 없이 검색하는 옵션.
    - 결과: `/Info.xml`에서 "RFCH" 문자열(대소문자 구분 없이)을 포함하는 모든 라인이 출력됨.

3. **`gawk -F"<" '{print $2}'`**:
    - `gawk`: 텍스트 처리용 프로그래밍 언어 AWK의 GNU 버전.
    - `-F"<"`: 입력 텍스트를 "<"로 구분.
    - `{print $2}`: 나눈 텍스트 중 두 번째 부분을 출력.
    - 결과: `<` 문자를 기준으로 구분된 텍스트의 두 번째 부분이 출력됨.

4. **`gawk -F">" '{print $2}'`**:
    - `gawk`: 동일한 명령어, 이번에는 ">"로 텍스트를 구분.
    - 결과: `>` 문자로 구분된 텍스트의 두 번째 부분이 출력됨.

<br/>

<br/>

---

```bash
echo "IP1" $IP1
ETH_IP="192.168.7."$IP1
echo "ETH_IP" $ETH_IP
```

1. **`echo "IP1" $IP1`**: 
   - `echo` 명령어를 사용하여 "IP1" 라는 문자열 뒤에 `$IP1` 변수의 값을 출력함.
   - 예: `IP1` 값이 `10`일 경우, 출력 결과는 `IP1 10`이다.

2. **`ETH_IP="192.168.7."$IP1`**:
   - `ETH_IP`라는 변수를 선언하고 "192.168.7." 라는 문자열에 `$IP1` 변수의 값을 추가하여 해당 변수에 할당함.
   - 예: `IP1` 값이 `10`일 경우, `ETH_IP` 값은 `192.168.7.10`이다.

3. **`echo "ETH_IP" $ETH_IP`**:
   - `ETH_IP` 변수의 이름과 그 값을 출력함.
   - 예: `ETH_IP` 값이 `192.168.7.10`일 경우, 출력 결과는 `ETH_IP 192.168.7.10`이다.

<br/>

<br/>

**총결**: 이 코드 `/Info.xml` 파일에서 "RFCH" 문자열을 포함하는 태그의 내용을 출력함.

<br/>

---

<br/>

### `ETH_INIT()`

이 함수는 시스템의 네트워크 인터페이스 설정을 초기화하거나 업데이트하는 기능임.

```bash
function ETH_INIT()
{
  ...
}
```

- **`sudo rm -rf /etc/network/interfaces`**: 네트워크 인터페이스 설정 파일을 삭제하는 명령. (주석 처리되어 있음)
- **`sudo echo " "`**: 설정 파일에 빈 줄 추가.
- **`sudo echo -n "allow-hotplug "`**: 장치의 동적 추가/제거를 허용하는 설정.
- **`sudo echo $ETH_NAME`**: 장치 이름을 설정 파일에 추가.
- **`sudo echo -n "auto "`**: 부팅 시 장치를 자동으로 활성화.
- **`sudo echo "iface" $ETH_NAME "inet static"`**: 정적 IP 설정 사용.
- **`sudo ifconfig $ETH_NAME $ETH_IP netmask 255.255.255.0`**: 네트워크 인터페이스에 IP 주소와 넷마스크 설정.

<br/>

### `ETH_CHANGE_IP()`

이 함수는 네트워크 인터페이스의 IP 주소를 변경하는 기능임.

```bash
function ETH_CHANGE_IP()
{
  ...
}
```

- **`sudo sed 's/192.168.7.1/192.168.7.2/g' /etc/network/interfaces`**: 설정 파일 내 IP 주소 변경.
- **`sudo ifconfig $ETH_NAME $ETH_IP netmask 255.255.255.0`**: 네트워크 인터페이스에 새 IP 주소와 넷마스크 설정.
- **`echo "ETH_CHANGE_IP"`**: IP 주소 변경 상태 메시지 출력.

<br/>

### `INTERFACES_INIT()`

이 함수는 네트워크 설정을 초기화하고 특히 무선 네트워크 인터페이스를 설정하는 기능임.

```bash
function INTERFACES_INIT()
{
  ...
}
```

- **`sudo rm -rf /etc/network/interfaces`**: 현재의 네트워크 설정 파일 삭제.
- **`sudo echo "source-directory /etc/network/interfaces.d"`**: 추가적인 네트워크 인터페이스 설정이 저장된 디렉토리를 참조.
- **`sudo echo -n "auto lo"` & `sudo echo "iface lo inet loopback"`**: 로컬 호스트 인터페이스 설정. 시스템의 내부 통신을 위한 인터페이스.
- **`sudo echo "auto wlan0"`**: 시스템 부팅 시 `wlan0` 인터페이스를 자동으로 활성화.
- **`sudo echo "iface wlan0 inet static"`**: `wlan0` 인터페이스에 대한 정적 IP 설정을 지정.
- **`sudo echo "pre-up iptables-restore < /usr/share/network-conf/ap/iptables.up.rules"`**: 인터페이스가 활성화되기 전에 iptables 규칙을 복원. 네트워크 보안 설정을 위함.
- **`sudo echo "address 192.168.8.1"`**: `wlan0`에 할당될 정적 IP 주소 지정.
- **`sudo echo "netmask 255.255.255.0"`**: 넷마스크 설정. 이 주소는 네트워크의 범위 결정.
- **`sudo echo "wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf"`**: WPA (Wi-Fi Protected Access) 설정 파일의 경로 지정. 무선 네트워크의 보안 설정을 위함.

<br/>

<br/>


### **네트워크 인터페이스 이름 검색 (ETH_NAME)**
```bash
ETH_NAME=`ifconfig | grep -i "enx" | gawk -F" " '{print$1}'`
echo "ETH_NAME" $ETH_NAME
```
- `ifconfig` 명령은 모든 네트워크 인터페이스의 정보를 출력.
- `grep -i "enx"`로 "enx"를 포함하는 인터페이스 이름만을 필터링. `-i` 옵션은 대소문자를 구분하지 않는다.
- `gawk -F" " '{print$1}'`를 사용하여 각 줄의 첫 번째 단어만 추출. "enx"로 시작하는 인터페이스 이름.
- 최종 결과는 `ETH_NAME` 변수에 저장.

### **네트워크 인터페이스의 IP 주소 검색 (ETH_ADR)**
```bash
ETH_ADR=`ifconfig $ETH_NAME|grep "inet addr:"|gawk -F":" '{print $2}'|gawk -F" " '{print $1}'`
echo "ETH ADR" $ETH_ADR
```
- `ifconfig $ETH_NAME` 명령은 특정 인터페이스(`$ETH_NAME`)의 정보를 출력.
- `grep "inet addr:"`를 사용하여 IP 주소 정보를 포함하는 줄만 필터링.
- `gawk`를 이용하여 IP 주소를 추출.

### **네트워크 인터페이스 설정 파일에서 인터페이스 이름 검색 (GET_NAME)**
```bash
GET_NAME=`cat /etc/network/interfaces|grep $ETH_NAME|grep "inet static"|gawk -F" " '{print $2}'`
echo "GET_NAME" $GET_NAME
```
- `/etc/network/interfaces` 파일에서 `grep $ETH_NAME`와 `grep "inet static"`을 통해 해당 인터페이스의 정적 IP 설정 정보를 찾는다.
- `gawk`로 인터페이스 이름을 추출.

### **네트워크 인터페이스 설정 파일에서 IP 주소 검색 (GET_ADR)**
```bash
GET_ADR=`cat /etc/network/interfaces|grep "192.168.7"|gawk -F" " '{print $2}'`
echo "GET_ADR" $GET_ADR
```
- `/etc/network/interfaces` 파일에서 `grep "192.168.7"`을 이용하여 해당 IP 주소를 포함하는 줄을 찾는다.
- `gawk`로 IP 주소를 추출합니다.

### **ip 명령을 사용하여 인터페이스 이름 검색 (ENX_NAME)**
```bash
ENX_NAME=`sudo ip link show|grep "enx"|gawk -F": " '{print $2}'`
echo "ENX_NAME" $ENX_NAME
```
- `sudo ip link show`로 모든 네트워크 인터페이스의 링크 정보를 출력.
- `grep "enx"`와 `gawk`를 이용하여 "enx"로 시작하는 인터페이스 이름을 추출.

### **인터페이스의 상태 (ENX_STATUS)**
```bash
ENX_STATUS=`sudo ip link show|grep "enx"|gawk -F" " '{print $3}'|grep -n "UP" | cut -d: -f1`
echo "ENX_STATUS" $ENX_STATUS
```
- `sudo ip link show`와 `grep "enx"`로 "enx"로 시작하는 인터페이스의 정보를 추출.
- `gawk`로 해당 인터페이스의 상태(UP/DOWN 등)를 추출. 
- `grep -n "UP"`으로 "UP" 상태인 줄의 번호를 찾아 `cut`을 이용하여 그 번호만 추출.

<br/>

<br/>

### **ENX 인터페이스 상태 확인 및 활성화**
```bash
if [ $ENX_STATUS == $ENX_UP ]; then
        echo "ENX is UP status..."
else
        sudo ifconfig $ENX_NAME up
        sleep 1
        ENX_STATUS=`sudo ip link show|grep "enx"|gawk -F" " '{print $9}'`
        echo "ENX up start:" $ENX_STATUS
fi
```

- 이 부분은 `$ENX_STATUS` 값이 `$ENX_UP`과 동일한지 확인.
- 만약 동일하다면, "ENX is UP status..."라는 메시지를 출력한다. 즉, ENX 인터페이스가 활성 상태인 것을 의미함.
- 동일하지 않다면, `sudo ifconfig $ENX_NAME up` 명령으로 해당 인터페이스를 활성화 시키려고 시도한다. 그 후, 잠시 1초 동안 기다린다가 인터페이스의 현재 상태를 다시 확인하여 출력함.

### **ETH_NAME 값 확인 및 ETH 초기화**

```bash
if [ -z $ETH_NAME ]; then
 echo "ETH_NAME is nothing"
 exit 1
else
 ...
fi
```

- `-z` 테스트는 문자열의 길이가 0인지 확인. 이 부분에서는 `$ETH_NAME` 변수가 설정되어 있는지 확인.
- 만약 설정되어 있지 않다면, "ETH_NAME is nothing"을 출력하고 스크립트를 종료.
- 설정되어 있다면, 아래의 내부 로직을 실행함.

### **네트워크 인터페이스 이름과 설정 값 확인**

```bash
echo "run ETH_INIT_1"
if [ -z $GET_NAME ]; then
  echo "run ETH_INIT_2"
  ETH_INIT
  exit 1
else
 ...
fi
```

- `$GET_NAME` 변수의 값이 설정되어 있는지 확인.
- 설정되어 있지 않다면, "run ETH_INIT_2"를 출력하고 `ETH_INIT` 함수를 실행한 후 스크립트를 종료.
- 설정되어 있다면, 아래의 내부 로직을 실행함.

### **IP 주소 일치 여부 확인 및 초기화**

```bash
echo "run ETH_INIT_3"
if [ $ETH_ADR == $ETH_IP ]; then
  echo "ETH_ADR is same"
else
  echo "run ETH_INIT_4"
  INTERFACES_INIT
  ETH_INIT
  exit 1
fi
```

- `$ETH_ADR`와 `$ETH_IP`가 동일한지 확인.
- 동일하다면, "ETH_ADR is same"를 출력.
- 동일하지 않다면, "run ETH_INIT_4"를 출력하고, `INTERFACES_INIT` 및 `ETH_INIT` 함수를 실행한 후 스크립트를 종료함.
