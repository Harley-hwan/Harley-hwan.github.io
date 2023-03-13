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

### 소스 1

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

## 결과 1

![image](https://user-images.githubusercontent.com/68185569/224203955-c5e35379-41da-422c-8081-da33da12b77b.png)

<br/>

<br/>

### 소스 2

```c++
#include <arpa/inet.h>
#include <ifaddrs.h>
#include <netdb.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <linux/if_link.h>
#include <string.h>

int main(int argc, char *argv[]) {
    struct ifaddrs *ifaddr, *ifa;
    int family, s;
    char host[NI_MAXHOST];

    if (getifaddrs(&ifaddr) == -1) {
        perror("getifaddrs");
        exit(EXIT_FAILURE);
    }

    // Wi-Fi 인터페이스에 해당하는 IPv4 주소 찾기
    for (ifa = ifaddr; ifa != NULL; ifa = ifa->ifa_next) {
        if (ifa->ifa_addr == NULL || ifa->ifa_addr->sa_family != AF_INET) {
            continue;
        }
        if (strcmp(ifa->ifa_name, "wlan0") == 0) { // Wi-Fi 인터페이스 이름에 맞게 수정
            struct sockaddr_in* addr = (struct sockaddr_in*)ifa->ifa_addr;
            char ip[INET_ADDRSTRLEN];
            inet_ntop(AF_INET, &(addr->sin_addr), ip, INET_ADDRSTRLEN);
            printf("Wi-Fi IPv4 Address: %s\n", ip);
            break;
        }
    }

    if (ifa == NULL) {
        printf("Unable to find Wi-Fi interface.\n");
        return 1;
    }

    // 모든 인터페이스에 대해 정보 출력
    for (ifa = ifaddr; ifa != NULL; ifa = ifa->ifa_next) {
        if (ifa->ifa_addr == NULL) {
            continue;
        }
        family = ifa->ifa_addr->sa_family;

        // Display interface name and family (including symbolic
        // form of the latter for the common families).
        printf("%-8s %s (%d)\n",
               ifa->ifa_name,
               (family == AF_PACKET) ? "AF_PACKET" :
               (family == AF_INET) ? "AF_INET" :
               (family == AF_INET6) ? "AF_INET6" : "???",
               family);

        // For an AF_INET* interface address, display the address.
        if (family == AF_INET || family == AF_INET6) {
            s = getnameinfo(ifa->ifa_addr,
                            (family == AF_INET) ? sizeof(struct sockaddr_in) :
                            sizeof(struct sockaddr_in6),
                            host, NI_MAXHOST, NULL, 0, NI_NUMERICHOST);
            if (s != 0) {
                printf("getnameinfo() failed: %s\n", gai_strerror(s));
                exit(EXIT_FAILURE);
            }

            printf("\t\taddress: <%s>\n", host);
        } else if (family == AF_PACKET && ifa->ifa_data != NULL) {
            struct rtnl_link_stats* stats = (struct rtnl_link_stats*)ifa->ifa_data;

            printf("\t\ttx_packets = %10u; rx_packets = %10u\n"
                   "\t\ttx_bytes   = %10u; rx_bytes   = %10u\n",
                   stats->tx_packets, stats->rx_packets,
                   stats->tx_bytes, stats->rx_bytes);
        }
   

    }

    freeifaddrs(ifaddr);
    exit(EXIT_SUCCESS);
}
```

<br/>

이 코드는 getifaddrs() 함수를 이용하여 현재 시스템의 네트워크 인터페이스 정보를 조회하고, 이를 이용하여 Wi-Fi 인터페이스의 IPv4 주소를 출력하는 예제 코드이다.

<코드의 구성>

- ifaddrs 구조체를 선언하고, getifaddrs() 함수를 이용하여 네트워크 인터페이스 정보를 조회한다.
- 조회한 인터페이스 정보를 순회하면서 Wi-Fi 인터페이스 이름("wlan0")과 일치하는 인터페이스의 IPv4 주소를 출력한다.
- 모든 인터페이스에 대해 정보를 출력한다. 인터페이스 이름, 인터페이스 패밀리(AF_PACKET, AF_INET, AF_INET6 등), IPv4/IPv6 주소 등의 정보를 출력한다.
- 마지막으로, getifaddrs() 함수를 이용하여 조회한 인터페이스 정보를 해제한다.

<br/>

- 주의할 점으로는, 코드 중간에 Wi-Fi 인터페이스의 이름이 "wlan0"으로 하드코딩되어 있다. 만약 시스템에서 사용하는 Wi-Fi 인터페이스의 이름이 다르다면, 이를 수정해주어야 한다. 
- 또한, IPv4 주소 출력 부분도 Wi-Fi 인터페이스에 대해서만 적용되도록 되어 있다. 이를 다른 인터페이스에 대해서도 적용하고 싶다면, 코드를 수정해주어야 한다.

<br/>

## 결과 2

![image](https://user-images.githubusercontent.com/68185569/224610333-a240b558-e48c-475b-b006-f9438ef9a43f.png)
