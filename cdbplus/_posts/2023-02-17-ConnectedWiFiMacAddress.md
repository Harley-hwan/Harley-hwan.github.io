---
layout: post
title: (c++) 현재 연결된 와이파이의 MAC 주소 검출
subtitle: wifi, mac, macaddress, c++, wlanopenhandle, wlanapi, ole32, windows, winanapi
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c++, wifi, mac, macaddress, wlanopenhandle, wlanapi, ole32, windows, winanapi]
comments: true
---

# 현재 연결된 와이파이의 MAC 주소 검출

- 최초 작성일: 2023년 2월 17일(금)

## 목차

[TOC]


## 내용

```c++
#include <iostream>
#include <Windows.h>
#include <wlanapi.h>
#include <objbase.h>
#include <wtypes.h>
#pragma comment(lib, "Wlanapi.lib")
#pragma comment(lib, "ole32.lib")

int main() {
    DWORD negotiatedVersion;
    HANDLE clientHandle = NULL;

    // Initialize the handle to the WLAN client.
    DWORD ret = WlanOpenHandle(2, NULL, &negotiatedVersion, &clientHandle);
    if (ret != ERROR_SUCCESS) {
        std::cerr << "WlanOpenHandle failed with error: " << ret << std::endl;
        return 1;
    }

    PWLAN_INTERFACE_INFO_LIST ifList = NULL;
    ret = WlanEnumInterfaces(clientHandle, NULL, &ifList);
    if (ret != ERROR_SUCCESS) {
        std::cerr << "WlanEnumInterfaces failed with error: " << ret << std::endl;
        return 1;
    }

    for (DWORD i = 0; i < ifList->dwNumberOfItems; i++) {
        PWLAN_INTERFACE_INFO pIfInfo = &ifList->InterfaceInfo[i];
        PWLAN_CONNECTION_ATTRIBUTES pConnectInfo = NULL;

        // Get the current connection attributes.
        ret = WlanQueryInterface(
            clientHandle,
            &pIfInfo->InterfaceGuid,
            wlan_intf_opcode_current_connection,
            NULL,
            (PDWORD)&pConnectInfo,
            (PVOID*)&pConnectInfo,
            NULL);

        if (ret != ERROR_SUCCESS) {
            std::cerr << "WlanQueryInterface failed with error: " << ret << std::endl;
            continue;
        }

        std::cout << "MAC address: ";
        for (DWORD j = 0; j < sizeof(pConnectInfo->wlanAssociationAttributes.dot11Bssid); j++) {
            if (j > 0) std::cout << ":";
            printf("%02X", pConnectInfo->wlanAssociationAttributes.dot11Bssid[j]);
        }
        std::cout << std::endl;

        WlanFreeMemory(pConnectInfo);
    }

    WlanFreeMemory(ifList);
    WlanCloseHandle(clientHandle, NULL);
    return 0;
}

```

<br/>

## 결과

![image](https://user-images.githubusercontent.com/68185569/219561425-804218a4-137d-47aa-a0be-6c993f9e0ba7.png)
