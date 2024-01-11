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

	이 함수는 사용자가 "Wifiscan" 버튼을 클릭하면 호출됩니다. 주요 작업은 다음과 같다.
	- WiFi 네트워크 목록을 가져온다.
	- RSSI(Received Signal Strength Indicator) 값에 따라 목록을 정렬한다.
	- 각 네트워크의 SSID, RSSI 값, 첫 연결 시간을 표시하는 문자열을 만들고 리스트 박스에 추가한다.
	- __'UpdateData(FALSE)'__ 를 호출하여 UI를 업데이트한다.

2. __ListAvailableWifiNetworks()__ 함수:

	이 함수는 사용 가능한 WiFi 네트워크 목록을 반환합니다. 주요 작업은 다음과 같다.
	- WLAN 클라이언트 핸들을 초기화한다.
	- 사용 가능한 인터페이스 목록을 검색한다.
	- 각 인터페이스에 대해 BSS(Basic Service Set) 목록을 가져온다.
	- BSS 목록의 각 항목에 대해 SSID를 변환하고, RSSI 값을 가져오며, 첫 연결 시간을 계산한다.
	- 변환된 SSID, RSSI 값, 첫 연결 시간을 튜플에 넣고 벡터에 추가한다.
	- 사용이 끝난 메모리를 해제하고 WLAN 클라이언트 핸들을 닫는다.

3. __ConvertSSID()__ 함수:

	이 함수는 주어진 SSID 데이터를 __'std::wstring'__ 유형으로 변환합니다. 주요 작업은 다음과 같다.
	- SSID 데이터를 UTF-8 인코딩으로 해석하려고 시도한다.
	- 유효한 UTF-8 문자열이 아닌 경우, 시스템 기본 코드 페이지를 사용하여 문자열을 해석한다.
	- 변환된 __'std::wstring'__ 을 반환한다.


	최근 수정한 부분은 __ConvertSSID()__ 함수에서 문자열을 변환하는 부분이다.
	SSID가 유효한 UTF-8 문자열이 아닌 경우 시스템 기본 코드 페이지를 사용하여 문자열을 변환하도록 수정하였다.

<br/>

---

<br/>

### 소스 1

```c++
void CMainDlg::OnBnClickedBtnWifiscan()
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
std::vector<std::tuple<CString, LONG, CString>> CMainDlg::ListAvailableWifiNetworks()
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
std::wstring CMainDlg::ConvertSSID(const UCHAR* ssid, ULONG ssidLength) {
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
void CMainDlg::OnBnClickedBtnWifiscan()
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

std::vector<std::tuple<CString, LONG, CString>> CMainDlg::ListAvailableWifiNetworks()
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

std::wstring CMainDlg::ConvertSSID(const UCHAR* ssid, ULONG ssidLength) {
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

<br/>

## 추가 수정 버전

노트북에서 와이파이 동글 장착 시, 동시에 와이파이를 2개 연결하는 것이 가능하다. 이러한 환경에서 위의 코드를 사용하면 와이파이 사용자 프로필이 중복되어 나오며, 해당 와이파이를 선택하여 연결을 시도해도 연결에 실패한다.

해당 환경에서도 정상 작동하도록 아래와 같이 코드를 수정하였다.

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

	std::set<CString> ssidSet; // 중복된 SSID를 추적하는 데 사용될 set

	for (DWORD i = 0; i < ifList->dwNumberOfItems; i++) {
		PWLAN_INTERFACE_INFO pIfInfo = &ifList->InterfaceInfo[i];

		PWLAN_BSS_LIST pBssList = NULL;
		ret = WlanGetNetworkBssList(clientHandle, &pIfInfo->InterfaceGuid, NULL, dot11_BSS_type_any, FALSE, NULL, &pBssList);
		if (ret != ERROR_SUCCESS) {
			std::cerr << "WlanGetNetworkBssList failed with error: " << ret << std::endl;
			continue;
		}

		for (DWORD j = 0; j < pBssList->dwNumberOfItems; j++) {
			PWLAN_BSS_ENTRY pBssEntry = &pBssList->wlanBssEntries[j];
			DOT11_SSID ssid = pBssEntry->dot11Ssid;

			std::wstring networkName = ConvertSSID(ssid.ucSSID, ssid.uSSIDLength);
			CStringW networkNameW = CStringW(networkName.c_str());
			CString networkNameT = CString(networkNameW);

			// 중복된 네트워크 확인
			if (ssidSet.find(networkNameT) == ssidSet.end()) {
				ssidSet.insert(networkNameT);

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

				availableNetworks.push_back(std::make_tuple(networkNameT, rssi, firstAvailableTime));
			}
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
bool CRadarCalibrationDlg::ConnectToSelectedWifi(const std::wstring& networkName, const std::wstring& password) {
	// SSID와 비밀번호를 std::string으로 변환
	std::string name(networkName.begin(), networkName.end());
	std::string pass(password.begin(), password.end());

	// XML 파일 생성
	std::string fileName = "myWlan.xml";
	std::ofstream xmlFile;
	xmlFile.open(fileName.c_str());

	if (!xmlFile.is_open()) {
		std::cerr << "Failed to create XML file." << std::endl;
		return false;
	}

	// XML 파일 작성
	xmlFile << "<?xml version=\"1.0\"?>\n";
	xmlFile << "<WLANProfile xmlns=\"http://www.microsoft.com/networking/WLAN/profile/v1\">\n";
	xmlFile << "<name>" << name << "</name>\n";
	xmlFile << "<SSIDConfig>\n<SSID>\n<hex>";
	for (int i = 0; i < name.length(); i++)
		xmlFile << std::hex << (int)name.at(i);
	xmlFile << "</hex>\n<name>" << name << "</name>\n</SSID>\n</SSIDConfig>\n";
	xmlFile << "<connectionType>ESS</connectionType>\n";
	xmlFile << "<connectionMode>auto</connectionMode>\n<MSM>\n<security>\n";
	xmlFile << "<authEncryption>\n<authentication>WPA2PSK</authentication>\n";
	xmlFile << "<encryption>AES</encryption>\n<useOneX>false</useOneX>\n";
	xmlFile << "</authEncryption>\n<sharedKey>\n<keyType>passPhrase</keyType>\n";
	xmlFile << "<protected>false</protected>\n<keyMaterial>" << pass << "</keyMaterial>\n";
	xmlFile << "</sharedKey>\n</security>\n</MSM>\n";
	xmlFile << "<MacRandomization xmlns=\"http://www.microsoft.com/networking/WLAN/profile/v3\">\n";
	xmlFile << "<enableRandomization>false</enableRandomization>\n</MacRandomization>\n";
	xmlFile << "</WLANProfile>";

	xmlFile.close();

	std::string interfaceName = "Wi-Fi"; // 기본값 "Wi-Fi"

	// Add the XML file to the system profile
	std::string command = "netsh wlan add profile filename=\"" + fileName + "\" interface=\"" + interfaceName + "\"";
	int result = system(command.c_str());
	if (result != 0) {
		std::cerr << "Failed to add WLAN profile, command returned: " << result << std::endl;
		return false;
	}

	// Connect to the network
	command = "netsh wlan connect name=\"" + name + "\" interface=\"" + interfaceName + "\"";
	result = system(command.c_str());
	if (result == 0) {
		return true;
	}
	else {
		std::cerr << "Failed to connect to the Wi-Fi network, command returned: " << result << std::endl;
		return false;
	}
}
```

<br/>

1. __ConnectToSelectedWifi()__ 함수

- 선택한 Wi-Fi 네트워크에 연결한다.
- 네트워크 이름과 비밀번호를 사용하여 Wi-Fi 프로필을 생성하고, 이 프로필을 사용해서 실제로 Wi-Fi에 연결을 시도한다.

2. __ListAvailableWifiNetworks()__ 함수

- 사용 가능한 Wi-Fi 네트워크 목록을 찾는다.
- 시스템에서 감지한 모든 Wi-Fi 네트워크를 리스트업하며, 중복된 네트워크는 제외한다.
- 각 네트워크의 이름, 신호 강도(RSSI), 첫 연결 시간 등의 정보를 제공한다.


<br/>

<br/>

## 추가 수정 2

std::string interfaceName = "Wi-Fi";

위와 같이 interfaceName 변수를 기본값인 "Wi-Fi"로 설정하여 메인 와이파이 어댑터로 설정을 했었다.

그것을 서브 와이파이 어댑터에 연결하도록 변경하고 싶다.

<br/>

예를 들어, 노트북의 메인 와이파이 어댑터 이름이 "Wi-Fi" 이고, 와이파이 동글을 장착했을 때 새로 생기는 이름이 "Wi-Fi 2" 혹은 "Wi-Fi 3" 과 같다면 해당 어댑터에 연결하고자 함이다.

<br/>

```c++
std::vector<std::wstring> GetWifiInterfaceNames() 
{
	system("netsh wlan show interfaces > interfaces.txt");

	std::vector<std::wstring> wifiNames;
	std::ifstream file("interfaces.txt");
	if (!file.is_open()) {
		std::wcerr << L"Error opening file" << std::endl;
		return wifiNames;
	}

	std::string line;
	bool isNewInterface = true;
	while (std::getline(file, line)) {
		if (isNewInterface && !line.empty()) {
			std::size_t pos = line.find(':');
			if (pos != std::string::npos) {
				std::string name = Trim(line.substr(pos + 1));
				wifiNames.push_back(std::wstring(name.begin(), name.end()));
			}
			isNewInterface = false;
		}
		if (line.empty()) {
			isNewInterface = true;
		}
	}
	file.close();
	return wifiNames;
}
```

<br/>

```c++
bool ConnectToSelectedWifi(const std::wstring& networkName, const std::wstring& password)
{
	std::string name(networkName.begin(), networkName.end());
	std::string pass(password.begin(), password.end());

	std::string fileName = "myWlan.xml";
	std::ofstream xmlFile;
	xmlFile.open(fileName.c_str());

	if (!xmlFile.is_open()) {
		std::cerr << "Failed to create XML file." << std::endl;
		return false;
	}

	// XML 파일 작성
	xmlFile << "<?xml version=\"1.0\"?>\n";
	xmlFile << "<WLANProfile xmlns=\"http://www.microsoft.com/networking/WLAN/profile/v1\">\n";
	xmlFile << "<name>" << name << "</name>\n";
	xmlFile << "<SSIDConfig>\n<SSID>\n<hex>";
	for (int i = 0; i < name.length(); i++)
		xmlFile << std::hex << (int)name.at(i);
	xmlFile << "</hex>\n<name>" << name << "</name>\n</SSID>\n</SSIDConfig>\n";
	xmlFile << "<connectionType>ESS</connectionType>\n";
	xmlFile << "<connectionMode>auto</connectionMode>\n<MSM>\n<security>\n";
	xmlFile << "<authEncryption>\n<authentication>WPA2PSK</authentication>\n";
	xmlFile << "<encryption>AES</encryption>\n<useOneX>false</useOneX>\n";
	xmlFile << "</authEncryption>\n<sharedKey>\n<keyType>passPhrase</keyType>\n";
	xmlFile << "<protected>false</protected>\n<keyMaterial>" << pass << "</keyMaterial>\n";
	xmlFile << "</sharedKey>\n</security>\n</MSM>\n";
	xmlFile << "<MacRandomization xmlns=\"http://www.microsoft.com/networking/WLAN/profile/v3\">\n";
	xmlFile << "<enableRandomization>false</enableRandomization>\n</MacRandomization>\n";
	xmlFile << "</WLANProfile>";

	xmlFile.close();

	std::vector<std::wstring> availableInterfaces = GetWifiInterfaceNames();

	//std::wcout.clear(); // 스트림의 에러 플래그를 초기화
	//std::wcout.sync_with_stdio(true);

	//// 사용 가능한 Wi-Fi 인터페이스 목록 출력
	//std::wcout << "Available Wi-Fi Interfaces:" << std::endl;
	//for (const auto& iface : availableInterfaces) {
	//	std::wcout << iface << std::endl;
	//}

	std::string interfaceName = "Wi-Fi";

	// "Wi-Fi" 외에 다른 인터페이스가 있는지 확인
	for (const auto& iface : availableInterfaces) {
		std::wstring ifaceName = iface;
		if (ifaceName != L"Wi-Fi" && ifaceName.find(L"Wi-Fi") != std::wstring::npos) {
			interfaceName = std::string(ifaceName.begin(), ifaceName.end());
			std::cout << "Selected interfaceName: " << interfaceName << std::endl;
		}
	}

	std::string command = "netsh wlan add profile filename=\"" + fileName + "\" interface=\"" + interfaceName + "\"";
	int result = system(command.c_str());
	if (result != 0) {
		std::cerr << "Failed to add WLAN profile, command returned: " << result << std::endl;
		return false;
	}

	// Connect to the network
	command = "netsh wlan connect name=\"" + name + "\" interface=\"" + interfaceName + "\"";
	result = system(command.c_str());
	if (result == 0) {
		return true;
	}
	else {
		std::cerr << "Failed to connect to the Wi-Fi network, command returned: " << result << std::endl;
		return false;
	}
}
```
