---
layout: post
title: "이더넷 어댑터 정보 찾기"
subtitle: "c++, window, ethernet, internet, LAN, interface"
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c, c++, window, ethernet, internet, LAN, interface]
comments: true
---

# 현재 연결되어 있는 이더넷 어댑터의 정보 얻기
- 최초 작성일: 2024년 1월 10일 (수)

## 목차

[TOC]

<br/>

## 내용



```c++
std::vector<std::wstring> GetWifiInterfaceNames() {
	std::vector<std::wstring> wifiNames;
	HRESULT hr = CoInitializeEx(NULL, COINIT_APARTMENTTHREADED);

	if (SUCCEEDED(hr)) {
		INetworkListManager* pNetworkListManager;
		hr = CoCreateInstance(CLSID_NetworkListManager, NULL, CLSCTX_ALL, IID_INetworkListManager, (void**)&pNetworkListManager);

		if (SUCCEEDED(hr)) {
			IEnumNetworkConnections* pEnumNetworkConnections;
			hr = pNetworkListManager->GetNetworkConnections(&pEnumNetworkConnections);

			if (SUCCEEDED(hr)) {
				INetworkConnection* pNetworkConnection;
				ULONG fetched;

				while (pEnumNetworkConnections->Next(1, &pNetworkConnection, &fetched) == S_OK) {
					VARIANT_BOOL isConnected;
					pNetworkConnection->get_IsConnectedToInternet(&isConnected);

					if (isConnected) {
						INetwork* pNetwork;
						pNetworkConnection->GetNetwork(&pNetwork);

						BSTR bstrNetworkName;
						pNetwork->GetName(&bstrNetworkName);

						wifiNames.push_back(bstrNetworkName);
						SysFreeString(bstrNetworkName);
						pNetwork->Release();
					}

					pNetworkConnection->Release();
				}

				pEnumNetworkConnections->Release();
			}

			pNetworkListManager->Release();
		}

		CoUninitialize();
	}

	return wifiNames;
}
```
