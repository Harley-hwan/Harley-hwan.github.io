---
layout: post
title: 네트워크 인터페이스 정보 가져오기 (ifaddrs)
subtitle: c++, linux, command, ifconfig, ifaddrs, ip, systemcommand, getifaddrs, inet_ntop, strstr
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c, c c++, linux, command, ifconfig, ifaddrs, ip, systemcommand, getifaddrs, inet_ntop, strstr]
comments: true
---

# 현재 Wi-Fi 연결된 IP 목록을 리스트 뽑기 (네트워크 인터페이스 정보)
- 최초 작성일: 2023년 3월 10일 (금)

## 목차

[TOC]

<br/>

## 내용

리눅스에서는 네트워크 인터페이스와 관련된 정보를 가져오기 위해 ifconfig나 ip 명령어를 사용할 수 있다.

이 명령어들은 시스템 콘솔에서 실행되며, 해당 명령어의 출력 결과를 파싱하여 필요한 정보를 추출하는 것이 가능하다.

그러나 C++에서는 시스템 명령어를 직접 실행하고 출력 결과를 파싱하는 것은 번거로운 일이 될 수 있다.

대신, 네트워크 인터페이스와 관련된 정보를 가져오기 위한 네트워크 인터페이스 함수를 사용할 수 있다.

예를 들어, getifaddrs 함수는 현재 시스템에서 사용 가능한 모든 네트워크 인터페이스의 정보를 가져올 수 있다.

이 함수는 ifaddrs 구조체를 사용하여 인터페이스 정보를 저장한다.

다음은 getifaddrs 함수를 사용하여 현재 Wi-Fi 연결된 IP 목록을 리스트로 뽑는 함수의 예시 코드이다.

<br/>

```c++
#include <iostream>
#include <ifaddrs.h>
#include <netdb.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <vector>
#include <cstring> // 추가된 헤더

std::vector<std::string> get_wifi_ips() {
    std::vector<std::string> ips;

    struct ifaddrs *ifaddr, *ifa;
    if (getifaddrs(&ifaddr) == -1) {
        std::cerr << "Failed to get network interface information.\n";
        return ips;
    }

    for (ifa = ifaddr; ifa != nullptr; ifa = ifa->ifa_next) {
        if (ifa->ifa_addr == nullptr) {
            continue;
        }

        if (ifa->ifa_addr->sa_family == AF_INET && strstr(ifa->ifa_name, "wlan") != nullptr) {
            // Wi-Fi interface found
            struct sockaddr_in *addr = (struct sockaddr_in *) ifa->ifa_addr;
            char ip_str[INET_ADDRSTRLEN];
            inet_ntop(AF_INET, &addr->sin_addr, ip_str, INET_ADDRSTRLEN);
            ips.push_back(ip_str);
        }
    }

    freeifaddrs(ifaddr);
    return ips;
}

int main() {
    std::vector<std::string> ips = get_wifi_ips();

    std::cout << "Wi-Fi IPs:\n";
    for (const auto& ip : ips) {
        std::cout << "- " << ip << "\n";
    }

    return 0;
}


```

<br/>

위 코드에서 getifaddrs 함수를 사용하여 모든 네트워크 인터페이스 정보를 가져오고, 각 인터페이스의 주소 체계를 확인하여 Wi-Fi 인터페이스를 찾는다.

그리고 inet_ntop 함수를 사용하여 주소를 문자열 형태로 변환한 후, 이를 std::vector에 추가한다.

마지막으로, 함수는 이 std::vector를 반환하며, 이를 출력하면 현재 Wi-Fi에 연결된 IP 주소 목록이 출력된다.

<br/>

## 결과

![image](https://user-images.githubusercontent.com/68185569/224203955-c5e35379-41da-422c-8081-da33da12b77b.png)

