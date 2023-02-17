---
layout: post
title: 현재 연결된 WiFi 사용자 프로필 리스트 검출
subtitle: c++, windows, wlan, WlanOpenHandle, WlanGetProfileList, WlanEnumInterfaces, ConvertWCharToString
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c, c++, windows, wlan, WlanOpenHandle, WlanGetProfileList, WlanEnumInterfaces, ConvertWCharToString]
comments: true
---

# 현재 연결된 WiFi
- 최초 작성일: 2023년 2월 17일 (금)

## 목차

[TOC]

<br/>

## 내용

시스템 커맨드인 "netsh wlan show profiles" 로 나온 결과 중,

현재 연결 중인 와이파이와 일치하는 사용자 프로필을 뽑고자 한다.

<br/>

## 실습1

<br/>

```c++
#include <iostream>
#include <Windows.h>
#include <wlanapi.h>
#include <objbase.h>
#include <wtypes.h>
#include <string>
#include <vector>

#pragma comment(lib, "Wlanapi.lib")
#pragma comment(lib, "ole32.lib")

std::wstring ConvertWCharToString(const WCHAR* wstr) {
    std::wstring str(wstr);
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
        PWLAN_CONNECTION_ATTRIBUTES pConnectInfo = NULL;

        // Get the current connection attributes.
        ret = WlanQueryInterface(clientHandle, &pIfInfo->InterfaceGuid, wlan_intf_opcode_current_connection, NULL,
            (PDWORD)&pConnectInfo, (PVOID)&pConnectInfo, NULL);
        if (ret != ERROR_SUCCESS) {
            std::cerr << "WlanQueryInterface failed with error: " << ret << std::endl;
            continue;
        }

        std::wcout << "Currently connected to: " << ConvertWCharToString(pConnectInfo->strProfileName) << std::endl;
        std::wcout << "Other profiles available: " << std::endl;

        PWLAN_PROFILE_INFO_LIST profileList = NULL;
        ret = WlanGetProfileList(clientHandle, &pIfInfo->InterfaceGuid, NULL, &profileList);
        if (ret != ERROR_SUCCESS) {
            std::cerr << "WlanGetProfileList failed with error: " << ret << std::endl;
            continue;
        }

        for (DWORD j = 0; j < profileList->dwNumberOfItems; j++) {
            PWLAN_PROFILE_INFO profileInfo = &profileList->ProfileInfo[j];
            std::wstring profileName = ConvertWCharToString(profileInfo->strProfileName);

            if (profileName != ConvertWCharToString(pConnectInfo->strProfileName)) {
                std::wcout << "- " << profileName << std::endl;
            }
        }

        WlanFreeMemory(pConnectInfo);
        WlanFreeMemory(profileList);
    }

    WlanFreeMemory(ifList);
    WlanCloseHandle(clientHandle, NULL);
    return 0;
}

```

<br/>

위의 코드를 실행하게 되면 아래와 같이 오류가 발생한다.

![image](https://user-images.githubusercontent.com/68185569/219573358-c18ecd40-2c2e-43b7-b6f5-1b4839c8bf85.png)

<br/>

<br/>

WlanQueryInterface() 함수는 다음과 같은 인자를 받는다.

```c++
DWORD WlanQueryInterface(
HANDLE hClientHandle,
const GUID *pInterfaceGuid,
WLAN_INTF_OPCODE OpCode,
PVOID pReserved,
PDWORD pdwDataSize,
PVOID *ppData,
PWLAN_OPCODE_VALUE_TYPE pWlanOpcodeValueType
);
```

<br/>

```c++
// Get the current connection attributes.
ret = WlanQueryInterface(clientHandle, &pIfInfo->InterfaceGuid, wlan_intf_opcode_current_connection, NULL,
    (PDWORD)&pConnectInfo, (PVOID)&pConnectInfo, NULL);
```

<br/>

하지만 현재 코드에서는 인자를 6개를 넘겨줬기 때문에 발생한 오류이다. 

함수 호출부분에서 넘겨주는 인자를 아래와 같이 수정해주면 된다.

<br/>

```c++
// Get the current connection attributes.
ret = WlanQueryInterface(
    clientHandle,
    &pIfInfo->InterfaceGuid,
    wlan_intf_opcode_current_connection,
    NULL,
    (PDWORD)&pConnectInfo,
    (PVOID*)&pConnectInfo,
    NULL);
```

<br/>

<br/>

## 풀소스

<br/>

실행시 현재 연결된 와이파이와 해당 와이파이를 사용하는 다른 사용자 프로필 이름이 출력된다.

<br/>

```c++
#include <iostream>
#include <Windows.h>
#include <wlanapi.h>
#include <objbase.h>
#include <wtypes.h>
#include <string>
#include <vector>

#pragma comment(lib, "Wlanapi.lib")
#pragma comment(lib, "ole32.lib")

std::wstring ConvertWCharToString(const WCHAR* wstr) {
    std::wstring str(wstr);
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

        std::wcout << "Currently connected to: " << ConvertWCharToString(pConnectInfo->strProfileName) << std::endl;
        std::wcout << "Other profiles available: " << std::endl;

        PWLAN_PROFILE_INFO_LIST profileList = NULL;
        ret = WlanGetProfileList(clientHandle, &pIfInfo->InterfaceGuid, NULL, &profileList);
        if (ret != ERROR_SUCCESS) {
            std::cerr << "WlanGetProfileList failed with error: " << ret << std::endl;
            continue;
        }

        for (DWORD j = 0; j < profileList->dwNumberOfItems; j++) {
            PWLAN_PROFILE_INFO profileInfo = &profileList->ProfileInfo[j];
            std::wstring profileName = ConvertWCharToString(profileInfo->strProfileName);

            if (profileName != ConvertWCharToString(pConnectInfo->strProfileName)) {
                std::wcout << "- " << profileName << std::endl;
            }
        }

        WlanFreeMemory(pConnectInfo);
        WlanFreeMemory(profileList);
    }

    WlanFreeMemory(ifList);
    WlanCloseHandle(clientHandle, NULL);
    return 0;
}

```
<br/>

### 결과1

![image](https://user-images.githubusercontent.com/68185569/219573236-74c8eccc-7a33-4673-a126-c28e20bdaaa5.png)

<br/>

<br/>

## 실습2

하지만 여기서, 필자 같은 경우는 와이파이 동글을 사용하여 동시에 2개의 와이파이가 연결하므로, 해당 와이파이들의 모든 사용자 프로필을 얻고자 한다.


