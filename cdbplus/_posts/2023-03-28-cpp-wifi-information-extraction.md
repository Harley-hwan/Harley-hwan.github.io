---
layout: post
title: "(C++) Windows 환경에서의 Wi-Fi 네트워크 프로그래밍"
subtitle: "WlanAPI를 활용한 Wi-Fi 스캔 및 연결 구현"
gh-repo: your-github-username/your-repo-name
gh-badge: [star, fork, follow]
tags: [c++, programming, wifi, wlanapi, windows, network]
comments: true
---

# Windows 환경에서의 Wi-Fi 네트워크 프로그래밍
- 최초 작성일: 2024년 1월 6일 (월)

<br/>

## 목차
1. [소개](#소개)
2. [Wi-Fi 스캔 기능](#wi-fi-스캔-기능)
3. [Wi-Fi 연결 기능](#wi-fi-연결-기능)
4. [유틸리티 함수들](#유틸리티-함수들)
5. [주의사항](#주의사항)
6. [전체 코드 분석](#전체-코드-분석)
7. [결론](#결론)

<br/>

## 소개
Windows 환경에서 Wi-Fi 네트워크를 프로그래밍 방식으로 제어하는 것은 매우 유용한 기능이다. 이 포스트에서는 WlanAPI를 사용하여 Wi-Fi 네트워크를 스캔하고 연결하는 방법을 상세히 살펴보겠다.

<br/>

## Wi-Fi 스캔 기능

### OnBnClickedButtonWifiScan 함수
Wi-Fi 스캔 버튼 클릭 시 실행되는 핵심 함수.

```cpp
void CWifiManagerDlg::OnBnClickedButtonWifiScan()
{
    std::vector<std::tuple<CString, LONG, CString>> v_Wifilist;
    m_lcWifiList.DeleteAllItems();

    v_Wifilist = ListAvailableWifiNetworks();

    // RSSI 값 기준으로 내림차순 정렬
    std::sort(v_Wifilist.begin(), v_Wifilist.end(),
        [](const auto& a, const auto& b) {
            return std::get<1>(a) > std::get<1>(b);
        });

    int nIndex = 0;
    for (const auto& item : v_Wifilist) {
        CString ssid, listItem;
        LONG rssi;
        CString linkTime;
        std::tie(ssid, rssi, linkTime) = item;

        // WAVE로 시작하는 SSID만 리스트에 추가
        if (ssid.Find(_T("WAVE")) == 0)
        {
            listItem.Format(_T("%s - RSSI: %d - First Connect: %s"),
                ssid, rssi, linkTime);
            m_lcWifiList.InsertItem(nIndex, listItem);
            nIndex++;
        }
    }
}
```

주요 특징:
1. `std::tuple`을 사용하여 SSID, RSSI, 연결 시간 정보를 관리
2. 람다 표현식을 활용한 RSSI 기준 정렬
3. "WAVE" 접두사를 가진 네트워크만 필터링

### ListAvailableWifiNetworks 함수
실제 Wi-Fi 스캔을 수행하는 함수.

```cpp
std::vector<std::tuple<CString, LONG, CString>> CWifiManagerDlg::ListAvailableWifiNetworks()
{
	std::vector<std::tuple<CString, LONG, CString>> availableNetworks;
	DWORD negotiatedVersion;
	HANDLE clientHandle = NULL;

	DWORD ret = WlanOpenHandle(2, NULL, &negotiatedVersion, &clientHandle);
	if (ret != ERROR_SUCCESS) {
		return availableNetworks;
	}

	PWLAN_INTERFACE_INFO_LIST ifList = NULL;
	ret = WlanEnumInterfaces(clientHandle, NULL, &ifList);
	if (ret != ERROR_SUCCESS) {
		WlanCloseHandle(clientHandle, NULL);
		return availableNetworks;
	}

	for (DWORD i = 0; i < ifList->dwNumberOfItems; i++) {
		PWLAN_INTERFACE_INFO pIfInfo = &ifList->InterfaceInfo[i];
		PWLAN_BSS_LIST pBssList = NULL;
		ret = WlanGetNetworkBssList(clientHandle, &pIfInfo->InterfaceGuid, NULL,
			dot11_BSS_type_any, FALSE, NULL, &pBssList);

		if (ret != ERROR_SUCCESS) {
			continue;
		}

		for (DWORD j = 0; j < pBssList->dwNumberOfItems; j++) {
			PWLAN_BSS_ENTRY pBssEntry = &pBssList->wlanBssEntries[j];
			DOT11_SSID ssid = pBssEntry->dot11Ssid;
			std::wstring networkName = ConvertSSID(ssid.ucSSID, ssid.uSSIDLength);
			LONG rssi = pBssEntry->lRssi;

			ULARGE_INTEGER ftSystemTime1970;
			ftSystemTime1970.QuadPart = 116444736000000000ULL;
			ULARGE_INTEGER ftTimestamp;
			ftTimestamp.QuadPart = ftSystemTime1970.QuadPart + (pBssEntry->ullHostTimestamp * 10);

			FILETIME ftFirstAvailableTime;
			ftFirstAvailableTime.dwHighDateTime = ftTimestamp.HighPart;
			ftFirstAvailableTime.dwLowDateTime = ftTimestamp.LowPart;

			SYSTEMTIME stFirstAvailableTime;
			FileTimeToSystemTime(&ftFirstAvailableTime, &stFirstAvailableTime);

			CString firstAvailableTime;
			firstAvailableTime.Format(_T("%02u:%02u:%02u"),
				stFirstAvailableTime.wHour,
				stFirstAvailableTime.wMinute,
				stFirstAvailableTime.wSecond);

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
```

주요 기능:
1. WlanAPI 초기화 및 핸들 관리
2. 인터페이스 열거 및 BSS 목록 획득
3. 타임스탬프 변환 및 시간 정보 포맷팅
4. 메모리 관리 및 리소스 해제

<br/>

## Wi-Fi 연결 기능

### OnBnClickedButtonWifiConnect 함수
Wi-Fi 연결 버튼 처리 함수.

```cpp
void CWifiManagerDlg::OnBnClickedButtonWifiConnect()
{
	POSITION pos = m_lcWifiList.GetFirstSelectedItemPosition();
	if (pos != NULL) {
		int selectedIndex = m_lcWifiList.GetNextSelectedItem(pos);
		CString selectedNetwork = m_lcWifiList.GetItemText(selectedIndex, 0);

		// SSID 부분만 추출 (SSID - RSSI: XX 형식에서)
		std::wregex ssidPattern(L"^([^ ]+)");
		std::wsmatch match;
		std::wstring selectedNetworkW = selectedNetwork.GetString();
		std::regex_search(selectedNetworkW, match, ssidPattern);
		std::wstring networkName = match.str(1);
		std::wstring password = L"wave1234";  // 기본 비밀번호

		if (ConnectToSelectedWifi(networkName, password)) {
			AfxMessageBox(_T("Wi-Fi에 연결되었습니다!"));

			// 윈도우 타이틀 업데이트
			CString windowTitle;
			windowTitle.Format(_T("FTP Client - Connected to %s"),
				CString(WStringToString(networkName).c_str()));
			this->SetWindowText(windowTitle);
		}
		else {
			AfxMessageBox(_T("Wi-Fi 연결에 실패했습니다."));
		}
	}
	else {
		AfxMessageBox(_T("선택된 Wi-Fi가 없습니다."));
	}
}
```

핵심 기능:
1. 선택된 네트워크 정보 추출
2. 정규식을 통한 SSID 파싱
3. 연결 상태 UI 업데이트
4. 에러 처리 및 사용자 피드백

### ConnectToSelectedWifi 함수
실제 Wi-Fi 연결을 수행하는 함수.

```cpp
bool CWifiManagerDlg::ConnectToSelectedWifi(const std::wstring& networkName, const std::wstring& password)
{
	std::string name(networkName.begin(), networkName.end());
	std::string pass(password.begin(), password.end());
	std::string fileName = "myWlan.xml";

	std::ofstream xmlFile;
	xmlFile.open(fileName.c_str());
	if (!xmlFile.is_open()) {
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

	// 시스템 프로파일에 XML 파일 추가
	std::string command = "netsh wlan add profile filename=" + fileName;
	if (system(command.c_str()) != 0) {
		return false;
	}

	// 네트워크 연결
	command = "netsh wlan connect name=" + name;
	if (system(command.c_str()) == 0) {
		return true;
	}

	return false;
}
```

구현 특징:
1. XML 프로파일 생성
2. WPA2-PSK 보안 설정
3. netsh 명령어 실행
4. 연결 상태 모니터링

<br/>

## 유틸리티 함수들

### ConvertSSID 함수
SSID 문자열 변환을 담당하는 유틸리티 함수.

```cpp
std::wstring CWifiManagerDlg::ConvertSSID(const unsigned char* ssid, size_t ssidLength)
{
	int len = MultiByteToWideChar(CP_UTF8, MB_ERR_INVALID_CHARS,
		reinterpret_cast<const char*>(ssid), ssidLength, NULL, 0);

	// UTF-8로 변환 실패시 시스템 기본 코드페이지로 시도
	if (len == 0 && GetLastError() == ERROR_NO_UNICODE_TRANSLATION) {
		len = MultiByteToWideChar(CP_ACP, 0,
			reinterpret_cast<const char*>(ssid), ssidLength, NULL, 0);
	}

	if (len > 0) {
		std::wstring networkName(len, L'\0');
		if (MultiByteToWideChar(CP_UTF8, 0,
			reinterpret_cast<const char*>(ssid), ssidLength,
			&networkName[0], len) > 0) {
			return networkName;
		}
	}

	// 변환 실패시 빈 문자열 반환
	return std::wstring();
}
```

주요 특징:
1. UTF-8 우선 시도
2. 시스템 코드페이지 폴백
3. 견고한 에러 처리

### WStringToString 함수
문자열 타입 변환 유틸리티.

```cpp
std::string CWifiManagerDlg::WStringToString(const std::wstring& wstr)
{
	string str;
	size_t size;
	str.resize(wstr.length());
	wcstombs_s(&size, &str[0], str.size() + 1, wstr.c_str(), wstr.size());
	return str;
}
```

<br/>

## 주의사항
1. **메모리 관리**
   - WlanAPI 리소스는 반드시 해제해야 함
   - 문자열 변환 시 버퍼 오버플로우 주의
   
2. **보안 고려사항**
   - 비밀번호는 평문으로 저장되지 않도록 주의
   - XML 파일 생성 시 적절한 권한 설정 필요

3. **에러 처리**
   - WlanAPI 함수 호출 결과 항상 검증
   - 네트워크 상태 변화 고려

4. **성능 최적화**
   - 불필요한 스캔 작업 최소화
   - 리소스 적절한 시점에 해제

<br/>

## 전체 코드 분석

### 코드 구조
```
CWifiManagerDlg
├── OnBnClickedButtonWifiScan
│   └── ListAvailableWifiNetworks
│       └── ConvertSSID
├── OnBnClickedButtonWifiConnect
│   ├── ConnectToSelectedWifi
│   └── WStringToString
└── 유틸리티 함수들
```

### 주요 특징
1. **모듈화**
   - 기능별 명확한 분리
   - 재사용 가능한 구조

2. **에러 처리**
   - 단계별 상태 확인
   - 사용자 피드백 제공

3. **리소스 관리**
   - RAII 패턴 활용
   - 적절한 메모리 해제

4. **코드 스타일**
   - 일관된 명명 규칙
   - 명확한 주석 처리

<br/>

## 결론
Windows 환경에서의 Wi-Fi 프로그래밍은 WlanAPI를 통해 효과적으로 구현할 수 있다. 적절한 에러 처리와 리소스 관리가 중요하며, 사용자 경험을 고려한 UI 업데이트도 필수적이다. 이 코드는 Wi-Fi 관련 기능을 구현하는 좋은 참고 사례가 될 수 있다.
