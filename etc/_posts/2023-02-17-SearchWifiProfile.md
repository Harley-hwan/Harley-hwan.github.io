---
layout: post
title:  (c++) 와이파이 사용자 프로필 리스트 뽑기
subtitle: c++, windows, wifi, profiles, WlanOpenHandle, WlanGetProfileList, ConvertWCharToString
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c, c++, windows, wifi, profiles, WlanOpenHandle, WlanGetProfileList, ConvertWCharToString]
comments: true
---

# 와이파이 사용자 프로필 리스트 뽑기
- 최초 작성일: 2023년 2월 17일 (금)

## 목차

[TOC]

<br/>

## 내용

시스템 커맨드 "netsh wlan show profiles" 를 이용해서 나오는 결과랑 동일하게 나옴

<br/>

```c++
#include <iostream>
#include <Windows.h>
#include <wlanapi.h>
#include <objbase.h>
#include <wtypes.h>
#include <string>
#pragma comment(lib, "Wlanapi.lib")
#pragma comment(lib, "ole32.lib")

std::string ConvertWCharToString(const wchar_t* wstr) {
    std::string str;
    int len = WideCharToMultiByte(CP_ACP, 0, wstr, -1, NULL, 0, NULL, NULL);
    if (len > 0) {
        str.resize(len);
        WideCharToMultiByte(CP_ACP, 0, wstr, -1, &str[0], len, NULL, NULL);
    }
    return str;
}

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

        PWLAN_PROFILE_INFO_LIST pProfileList = NULL;
        ret = WlanGetProfileList(clientHandle, &pIfInfo->InterfaceGuid, NULL, &pProfileList);
        if (ret != ERROR_SUCCESS) {
            std::cerr << "WlanGetProfileList failed with error: " << ret << std::endl;
            continue;
        }

        for (DWORD j = 0; j < pProfileList->dwNumberOfItems; j++) {
            std::string profileName = ConvertWCharToString(pProfileList->ProfileInfo[j].strProfileName);
            std::cout << "SSID: " << profileName << std::endl;
        }

        WlanFreeMemory(pProfileList);
    }

    WlanFreeMemory(ifList);
    WlanCloseHandle(clientHandle, NULL);
    return 0;
}
```

<br/>

### 결과

![image](https://user-images.githubusercontent.com/68185569/219568884-e22e7bb7-67f7-4d51-973d-848d8d165b58.png)
