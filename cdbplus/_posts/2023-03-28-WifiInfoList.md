---
layout: post
title: Wi-Fi 정보(SSID, RSSI, First Connect) 뽑기
subtitle: c++, window, wifi, wi-fi, wifilist, interface, 
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c, c c++, linux, command, ifconfig, ifaddrs, ip, systemcommand, getifaddrs, inet_ntop, strstr]
comments: true
---

# 현재 연결가능한 Wi-Fi List의 정보 (SSID, RSSI, first connect) 뽑기
- 최초 작성일: 2023년 3월 28일 (화)

## 목차

[TOC]

<br/>

## 내용



<br/>

### 소스 1

```c++
std::vector<std::tuple<std::wstring, LONG, CString>> ListAvailableWifiNetworks()
{
	std::vector<std::tuple<std::wstring, LONG, CString>> availableNetworks;

	DWORD negotiatedVersion;
	HANDLE clientHandle = NULL;

	// Initialize the handle to the WLAN client.
	DWORD ret = WlanOpenHandle(2, NULL, &negotiatedVersion, &clientHandle);
	if (ret != ERROR_SUCCESS) {
		std::cerr << "WlanOpenHandle failed with error: " << ret << std::endl;
		return availableNetworks;
	}

	PWLAN_INTERFACE_INFO_LIST ifList = NULL;
	ret = WlanEnumInterfaces(clientHandle, NULL, &ifList);
	if (ret != ERROR_SUCCESS) {
		std::cerr << "WlanEnumInterfaces failed with error: " << ret << std::endl;
		return availableNetworks;
	}

	for (DWORD i = 0; i < ifList->dwNumberOfItems; i++) {
		PWLAN_INTERFACE_INFO pIfInfo = &ifList->InterfaceInfo[i];

		PWLAN_BSS_LIST pBssList = NULL;
		ret = WlanGetNetworkBssList(clientHandle, &pIfInfo->InterfaceGuid, NULL, dot11_BSS_type_any, FALSE, NULL, &pBssList);
		if (ret != ERROR_SUCCESS) {
			std::cerr << "WlanGetNetworkBssList failed with error: " << ret << std::endl;
			return availableNetworks;
		}

		for (DWORD j = 0; j < pBssList->dwNumberOfItems; j++) {
			PWLAN_BSS_ENTRY pBssEntry = &pBssList->wlanBssEntries[j];
			DOT11_SSID ssid = pBssEntry->dot11Ssid;

			std::wstring networkName(reinterpret_cast<const wchar_t*>(ssid.ucSSID), ssid.uSSIDLength);

			LONG rssi = pBssEntry->lRssi; // RSSI 정보

			ULARGE_INTEGER ftSystemTime1970;
			ftSystemTime1970.QuadPart = 116444736000000000ULL; // 1970년 1월 1일 00:00:00 UTC와의 차이

			ULARGE_INTEGER ftTimestamp;
			ftTimestamp.QuadPart = ftSystemTime1970.QuadPart + (pBssEntry->ullHostTimestamp * 10); // 100ns 단위로 변환

			FILETIME ftFirstAvailableTime;
			ftFirstAvailableTime.dwHighDateTime = ftTimestamp.HighPart;
			ftFirstAvailableTime.dwLowDateTime = ftTimestamp.LowPart;

			SYSTEMTIME stFirstAvailableTime;
			FileTimeToSystemTime(&ftFirstAvailableTime, &stFirstAvailableTime);

			CString firstAvailableTime;
			firstAvailableTime.Format(_T("%02u:%02u:%02u"), stFirstAvailableTime.wHour, stFirstAvailableTime.wMinute, stFirstAvailableTime.wSecond);

			availableNetworks.push_back(std::make_tuple(networkName, rssi, firstAvailableTime));
		}
		WlanFreeMemory(pBssList);
	}

	WlanFreeMemory(ifList);
	WlanCloseHandle(clientHandle, NULL);

	return availableNetworks;
}

```



