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

<br/>

아래는 현재 연결된 와이파이의 사용자 프로필만 출력하는 C++ 코드이다.

현재 연결된 와이파이의 사용자 프로필 이름을 가져오고, 이를 기준으로 전체 사용자 프로필 중 연결된 와이파이의 사용자 프로필만 출력하는 방식을 사용했다.

```c++
#include <iostream>
#include <Windows.h>
#include <wlanapi.h>
#include <objbase.h>
#include <wtypes.h>
#include <string>
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
            (PDWORD)&pConnectInfo, NULL);
        if (ret != ERROR_SUCCESS) {
            continue;
        }

        std::wstring connectedProfileName = ConvertWCharToString(pConnectInfo->strProfileName);
        std::cout << "Connected WiFi profile: " << connectedProfileName << std::endl;

        WlanFreeMemory(pConnectInfo);

        PWLAN_PROFILE_INFO_LIST pProfileList = NULL;
        ret = WlanGetProfileList(clientHandle, &pIfInfo->InterfaceGuid, NULL, &pProfileList);
        if (ret == ERROR_SUCCESS) {
            for (DWORD j = 0; j < pProfileList->dwNumberOfItems; j++) {
                PWLAN_PROFILE_INFO pProfileInfo = &pProfileList->ProfileInfo[j];
                std::wstring profileName = ConvertWCharToString(pProfileInfo->strProfileName);
                if (profileName == connectedProfileName) {
                    std::cout << "Matched connected WiFi profile: " << profileName << std::endl;
                }
            }

            WlanFreeMemory(pProfileList);
        }
    }

    WlanFreeMemory(ifList);
    WlanCloseHandle(clientHandle, NULL);
    return 0;
}

```

<br/>

코드를 실행해보면 아래와 같은 에러가 뜬다.

![image](https://user-images.githubusercontent.com/68185569/219581944-80a37a51-81f2-4eaa-b5a1-33e24d6949a2.png)

<br/>

이 에러는 std::wstring 타입의 값에 대해 << 연산자를 지원하지 않는 것으로 보인다.
이를 해결하기 위해서는 std::wstring 타입을 출력 가능한 형태로 변환하는 방법을 사용하면 된다.

예를 들어, std::wcout 스트림을 사용하여 std::wstring 값을 출력하거나, std::wstringstream을 사용하여 std::wstring 값을 문자열로 변환하는 방법이 있다.
아래는 std::wstringstream을 사용하여 std::wstring 값을 출력하는 코드 예시이다.

```c++
std::wstring str = L"example";
std::wstringstream ss;
ss << str;
std::wcout << ss.str() << std::endl;
```

<br/>

위 코드는 std::wstring 값을 std::wstringstream 객체에 넣어 문자열로 변환한 후, std::wcout 스트림을 사용하여 출력하는 방식으로 동작한다.

이를 참고하여 코드를 수정해보자. 사실 std::cout 을 std::wcout 으로만 바꿔주면 된다.

<br/>

### 풀 소스

```c++
#include <iostream>
#include <Windows.h>
#include <wlanapi.h>
#include <objbase.h>
#include <wtypes.h>
#include <string>
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
            continue;
        }

        std::wstring connectedProfileName = ConvertWCharToString(pConnectInfo->strProfileName);
        std::wcout << "Connected WiFi profile: " << connectedProfileName << std::endl;

        WlanFreeMemory(pConnectInfo);

        PWLAN_PROFILE_INFO_LIST pProfileList = NULL;
        ret = WlanGetProfileList(clientHandle, &pIfInfo->InterfaceGuid, NULL, &pProfileList);
        if (ret == ERROR_SUCCESS) {
            for (DWORD j = 0; j < pProfileList->dwNumberOfItems; j++) {
                PWLAN_PROFILE_INFO pProfileInfo = &pProfileList->ProfileInfo[j];
                std::wstring profileName = ConvertWCharToString(pProfileInfo->strProfileName);
                if (profileName == connectedProfileName) {
                    std::wcout << "Matched connected WiFi profile: " << profileName << std::endl;
                }
            }

            WlanFreeMemory(pProfileList);
        }
    }

    WlanFreeMemory(ifList);
    WlanCloseHandle(clientHandle, NULL);
    return 0;
}

```

<br/>

### 결과2

![image](https://user-images.githubusercontent.com/68185569/219582653-68ab9172-7ad5-47b6-b541-11ce38c1959d.png)



