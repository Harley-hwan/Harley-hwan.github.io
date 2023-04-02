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
- 최초 작성일: 2023년 4월 2일 (일)

## 목차

[TOC]

<br/>

## 내용

이 소스 코드는 주요 기능으로 WiFi 네트워크를 스캔하고 결과를 리스트에 표시하는 MFC(Microsoft Foundation Class) 소스 코드이다.

<br/>

### 주요 기능 & 함수

1. __OnBnClickedBtnWifiscan()__ 함수:

	이 함수는 사용자가 "Wifiscan" 버튼을 클릭하면 호출됩니다. 주요 작업은 다음과 같습니다.
	- WiFi 네트워크 목록을 가져옵니다.
	- RSSI(Received Signal Strength Indicator) 값에 따라 목록을 정렬합니다.
	- 각 네트워크의 SSID, RSSI 값, 첫 연결 시간을 표시하는 문자열을 만들고 리스트 박스에 추가합니다.
	- __'UpdateData(FALSE)'__ 를 호출하여 UI를 업데이트합니다.

2. __ListAvailableWifiNetworks()__ 함수:

	이 함수는 사용 가능한 WiFi 네트워크 목록을 반환합니다. 주요 작업은 다음과 같습니다.
	- WLAN 클라이언트 핸들을 초기화합니다.
	- 사용 가능한 인터페이스 목록을 검색합니다.
	- 각 인터페이스에 대해 BSS(Basic Service Set) 목록을 가져옵니다.
	- BSS 목록의 각 항목에 대해 SSID를 변환하고, RSSI 값을 가져오며, 첫 연결 시간을 계산합니다.
	- 변환된 SSID, RSSI 값, 첫 연결 시간을 튜플에 넣고 벡터에 추가합니다.
	- 사용이 끝난 메모리를 해제하고 WLAN 클라이언트 핸들을 닫습니다.

3. __ConvertSSID()__ 함수:

	이 함수는 주어진 SSID 데이터를 __'std::wstring'__ 유형으로 변환합니다. 주요 작업은 다음과 같습니다.
	- SSID 데이터를 UTF-8 인코딩으로 해석하려고 시도합니다.
	- 유효한 UTF-8 문자열이 아닌 경우, 시스템 기본 코드 페이지를 사용하여 문자열을 해석합니다.
	- 변환된 __'std::wstring'__ 을 반환합니다.


	최근 수정한 부분은 __ConvertSSID()__ 함수에서 문자열을 변환하는 부분이다.
	SSID가 유효한 UTF-8 문자열이 아닌 경우 시스템 기본 코드 페이지를 사용하여 문자열을 변환하도록 수정하였다.

<br/>

---

<br/>

### 소스 1

```c++
void CRadarCalibrationDlg::OnBnClickedBtnWifiscan()
{
	std::vector<std::tuple<CString, LONG, CString>> v_Wifilist;

	m_list_wifi.ResetContent();
	ASSERT(m_list_wifi.GetCount() == 0);

	v_Wifilist = ListAvailableWifiNetworks();

	// RSSI 값을 기준으로 내림차순 정렬
	std::sort(v_Wifilist.begin(), v_Wifilist.end(), [](const auto& a, const auto& b) {
		return std::get<1>(a) > std::get<1>(b);
		});

	for (const auto& item : v_Wifilist) {
		CString ssid;
		LONG rssi;
		CString linkTime;
		std::tie(ssid, rssi, linkTime) = item;

		// WAVE로 시작하는 SSID만 리스트 뷰에 추가
		//if (ssid.CompareNoCase(_T("WAVE")) == 0) {
		CString listItem;
		listItem.Format(_T("%s - RSSI: %d - First Connect: %s"), ssid, rssi, linkTime);
		m_list_wifi.AddString(listItem);
		//}
	}

	UpdateData(FALSE);
}

```

<br/>

```c++
std::vector<std::tuple<CString, LONG, CString>> CRadarCalibrationDlg::ListAvailableWifiNetworks()
{
	std::vector<std::tuple<CString, LONG, CString>> availableNetworks;

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

			std::wstring networkName = ConvertSSID(ssid.ucSSID, ssid.uSSIDLength);


			LONG rssi = pBssEntry->lRssi; // RSSI 정보

			ULARGE_INTEGER ftSystemTime1970;
			ftSystemTime1970.QuadPart = 116444736000000000ULL; // 1970년 1월 1일 00:00:00 UTC와의 차이

			ULARGE_INTEGER ftTimestamp;
			ftTimestamp.QuadPart = ftSystemTime1970.QuadPart + (pBssEntry->ullHostTimestamp * 10); // 100ns 단위로 변환Length);

			FILETIME ftFirstAvailableTime;
			ftFirstAvailableTime.dwHighDateTime = ftTimestamp.HighPart;
			ftFirstAvailableTime.dwLowDateTime = ftTimestamp.LowPart;

			SYSTEMTIME stFirstAvailableTime;
			FileTimeToSystemTime(&ftFirstAvailableTime, &stFirstAvailableTime);

			CString firstAvailableTime;
			firstAvailableTime.Format(_T("%02u:%02u:%02u"), stFirstAvailableTime.wHour, stFirstAvailableTime.wMinute, stFirstAvailableTime.wSecond);

			//availableNetworks.push_back(std::make_tuple(CString(networkName.c_str()), rssi, firstAvailableTime));
			// Replace the following line
// availableNetworks.push_back(std::make_tuple(CString(ATL::CW2T(networkName.c_str())), rssi, firstAvailableTime));
// with
			CStringA networkNameA = CStringA(ATL::CW2A(networkName.c_str(), CP_UTF8));
			availableNetworks.push_back(std::make_tuple(CString(networkNameA), rssi, firstAvailableTime));

		}
		WlanFreeMemory(pBssList);
	}

	WlanFreeMemory(ifList);
	WlanCloseHandle(clientHandle, NULL);

	return availableNetworks;
}

```

<br/>

```c++
std::wstring CRadarCalibrationDlg::ConvertSSID(const UCHAR* ssid, ULONG ssidLength) {
	int len = MultiByteToWideChar(CP_UTF8, MB_ERR_INVALID_CHARS, reinterpret_cast<const char*>(ssid), ssidLength, NULL, 0);

	// If the string is not a valid UTF-8, try the system's default code page
	if (len == 0 && GetLastError() == ERROR_NO_UNICODE_TRANSLATION) {
		len = MultiByteToWideChar(CP_ACP, 0, reinterpret_cast<const char*>(ssid), ssidLength, NULL, 0);
	}

	std::wstring networkName(len, L'\0');
	MultiByteToWideChar(CP_UTF8, 0, reinterpret_cast<const char*>(ssid), ssidLength, &networkName[0], len);
	return networkName;
}
```

<br/>

---

<br/>

#### 원래 코드

```c++
CStringA networkNameA = CStringA(ATL::CW2A(networkName.c_str(), CP_UTF8));
availableNetworks.push_back(std::make_tuple(CString(networkNameA), rssi, firstAvailableTime));
```

<br/>

이 코드에서는 ATL::CW2A를 사용하여 유니코드 std::wstring인 networkName을 UTF-8로 인코딩된 CStringA 객체인 networkNameA로 변환한다. 

그런 다음, CString 생성자를 사용하여 CStringA에서 CString 객체로 변환하고, 이를 availableNetworks 벡터에 푸시한다.

그러나 아래와 같이 한글만 깨져서 출력되는 현상이 있어서 수정하였다.

![image](https://user-images.githubusercontent.com/68185569/229327726-c201d931-f95e-4d8a-9970-ea057b1f218d.png)

<br/>

<br/>

#### 수정 코드

```c++
CStringW networkNameW = CStringW(networkName.c_str());
CString networkNameT = CString(networkNameW);
availableNetworks.push_back(std::make_tuple(networkNameT, rssi, firstAvailableTime));
```

<br/>

이 코드에서는 networkName을 CStringW 객체인 networkNameW로 변환한다. 

CStringW는 유니코드 문자를 처리하는 데 사용되는 CString의 유니코드 버전이다. 

그런 다음, CString 생성자를 사용하여 CStringW에서 CString 객체로 변환하고, 이를 availableNetworks 벡터에 푸시한다. 

그러면 아래와 같이 한글이 정상적으로 출력된다.

![image](https://user-images.githubusercontent.com/68185569/229327819-f5123a9b-684c-477a-90b9-f8ce8a6b61db.png)


<br/>

---

<br/>

## 최종 소스

```c++
void CRadarCalibrationDlg::OnBnClickedBtnWifiscan()
{
	std::vector<std::tuple<CString, LONG, CString>> v_Wifilist;

	m_list_wifi.ResetContent();
	ASSERT(m_list_wifi.GetCount() == 0);

	v_Wifilist = ListAvailableWifiNetworks();

	// RSSI 값을 기준으로 내림차순 정렬
	std::sort(v_Wifilist.begin(), v_Wifilist.end(), [](const auto& a, const auto& b) {
		return std::get<1>(a) > std::get<1>(b);
		});

	for (const auto& item : v_Wifilist) {
		CString ssid;
		LONG rssi;
		CString linkTime;
		std::tie(ssid, rssi, linkTime) = item;

		// WAVE로 시작하는 SSID만 리스트 뷰에 추가
		//if (ssid.CompareNoCase(_T("WAVE")) == 0) {
		CString listItem;
		listItem.Format(_T("%s - RSSI: %d - First Connect: %s"), ssid, rssi, linkTime);
		m_list_wifi.AddString(listItem);
		//}
	}

	UpdateData(FALSE);
}

std::vector<std::tuple<CString, LONG, CString>> CRadarCalibrationDlg::ListAvailableWifiNetworks()
{
	std::vector<std::tuple<CString, LONG, CString>> availableNetworks;

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

			std::wstring networkName = ConvertSSID(ssid.ucSSID, ssid.uSSIDLength);


			LONG rssi = pBssEntry->lRssi; // RSSI 정보

			ULARGE_INTEGER ftSystemTime1970;
			ftSystemTime1970.QuadPart = 116444736000000000ULL; // 1970년 1월 1일 00:00:00 UTC와의 차이

			ULARGE_INTEGER ftTimestamp;
			ftTimestamp.QuadPart = ftSystemTime1970.QuadPart + (pBssEntry->ullHostTimestamp * 10); // 100ns 단위로 변환Length);

			FILETIME ftFirstAvailableTime;
			ftFirstAvailableTime.dwHighDateTime = ftTimestamp.HighPart;
			ftFirstAvailableTime.dwLowDateTime = ftTimestamp.LowPart;

			SYSTEMTIME stFirstAvailableTime;
			FileTimeToSystemTime(&ftFirstAvailableTime, &stFirstAvailableTime);

			CString firstAvailableTime;
			firstAvailableTime.Format(_T("%02u:%02u:%02u"), stFirstAvailableTime.wHour, stFirstAvailableTime.wMinute, stFirstAvailableTime.wSecond);

			CStringW networkNameW = CStringW(networkName.c_str());
			CString networkNameT = CString(networkNameW);
			availableNetworks.push_back(std::make_tuple(networkNameT, rssi, firstAvailableTime));
		}
		WlanFreeMemory(pBssList);
	}

	WlanFreeMemory(ifList);
	WlanCloseHandle(clientHandle, NULL);

	return availableNetworks;
}

std::wstring CRadarCalibrationDlg::ConvertSSID(const UCHAR* ssid, ULONG ssidLength) {
	int len = MultiByteToWideChar(CP_UTF8, MB_ERR_INVALID_CHARS, reinterpret_cast<const char*>(ssid), ssidLength, NULL, 0);

	// If the string is not a valid UTF-8, try the system's default code page
	if (len == 0 && GetLastError() == ERROR_NO_UNICODE_TRANSLATION) {
		len = MultiByteToWideChar(CP_ACP, 0, reinterpret_cast<const char*>(ssid), ssidLength, NULL, 0);
	}

	std::wstring networkName(len, L'\0');
	MultiByteToWideChar(CP_UTF8, 0, reinterpret_cast<const char*>(ssid), ssidLength, &networkName[0], len);
	return networkName;
}
```
