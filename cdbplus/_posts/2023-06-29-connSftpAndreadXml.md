---
layout: post
title: (c++) sftp Connect & read xml
subtitle: c, c++, vs, sftp, ftp, CkSFtp, xml, pugi, pugixml
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c, c++, vs, sftp, ftp, CkSFtp, xml, pugi, pugixml]
comments: true
---

# sftp 접속 및 xml 다운로드 후, 원하는 값 꺼내오기
- 최초 작성일: 2023년 6월 29일 (목)

## 목차

[TOC]

<br/>

## 코드

### 사용한 라이브러리

sftp 접속: CksFtp
xml 접속: pugixml

<br/>

```c++
std::pair<std::string, std::string> connectSftp() {
	CkSFtp sftp;

	const char* hostname = "192.168.8.1";
	int port = 22;

	bool success = sftp.Connect(hostname, port);
	if (success != true) {
		std::cerr << sftp.lastErrorText() << "\n";
		return { "", "" };
	}

	success = sftp.AuthenticatePw("root", "fa");
	if (success != true) {
		std::cerr << sftp.lastErrorText() << "\n";
		return { "", "" };
	}

	success = sftp.InitializeSftp();
	if (success != true) {
		std::cerr << sftp.lastErrorText() << "\n";
		return { "", "" };
	}

	const char* handle;
	handle = sftp.openFile("/home/pi/test/Info.xml", "readOnly", "openExisting");
	if (sftp.get_LastMethodSuccess() != true) {
		std::cerr << sftp.lastErrorText() << "\n";
		return { "", "" };
	}

	CString dirName = _T("./xml");
	if (!CreateDirectory(dirName, NULL) && GetLastError() != ERROR_ALREADY_EXISTS) {
		// 디렉토리 생성 실패 처리
	}

	success = sftp.DownloadFile(handle, "./xml/Info.xml");
	if (success != true) {
		std::cerr << sftp.lastErrorText() << "\n";
		return { "", "" };
	}

	success = sftp.CloseHandle(handle);
	if (success != true) {
		std::cerr << sftp.lastErrorText() << "\n";
		return { "", "" };
	}

	pugi::xml_document doc;
	pugi::xml_parse_result result = doc.load_file("./xml/Info.xml");

	if (!result) {
		std::cerr << "Parsing failed with description: " << result.description() << "\n";
		return { "", "" };
	}

	pugi::xml_node nodeValW = doc.child("Root").child("SetValW");
	pugi::xml_node nodeValH = doc.child("Root").child("SetValH");

	if (!nodeValW) {
		std::cerr << "SetValW node not found." << std::endl;
		return { "", "" };
	}
	if (!nodeValH) {
		std::cerr << "SetValH node not found." << std::endl;
		return { "", "" };
	}

	std::string setValW = nodeValW.text().get();
	std::string setValH = nodeValH.text().get();

	try {
		if (std::stoi(setValW) >= 100 || std::stoi(setValW) <= -100) {
			std::cerr << "Invalid value for SetValW: " << setValW << std::endl;
			return { "", "" };
	}
	catch (std::exception& e) {
		std::cerr << "Exception caught trying to convert SetValW to integer: " << e.what() << std::endl;
		return { "", "" };
	}
  try {
  		if (std::stoi(setValH) >= 100 || std::stoi(setValH) <= -100) {
  			std::cerr << "Invalid value for SetValH: " << setValH << std::endl;
  			return { "", "" };
  		}
  }
  catch (std::exception& e) {
  	std::cerr << "Exception caught trying to convert SetValH to integer: " << e.what() << std::endl;
  	return { "", "" };
  }
	std::cout << "setValW: " << setValW << std::endl;
	std::cout << "setValH: " << setValH << std::endl;

	return { setValW, setValH };
}
```
