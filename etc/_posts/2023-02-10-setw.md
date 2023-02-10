---
layout: post
title: 표 형식으로 출력하기 (setw(), cout.setf(~);
subtitle: c++, table, log, setw, cout.setf, setf
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c, c++, setw, cout.setf, setf, log, cout]
comments: true
---

#  c++ 표 형태로 출력하기 setw()
- 최초 작성일: 2023년 2월 10일 (금)

## 목차

[TOC]

<br/>

## 내용

```c++
	cout.setf(ios_base::left);

	string resCamera = resCameraPass + "(" + degreebuffer + ")";	// 카메라 검증 결과
	string resdiag = resdiagPass + "(" + strErrCode + ")";			// 진단 결과 (에러코드)

	/// <summary>
	/// Result Part
	/// </summary>
	cout << setw(3) << " "	<< setw(10) << "항목"	<< setw(12) << "내용"	<< setw(15) << "결과"	<< setw(15) << "비고"		<< endl;

	cout << setw(3) << "1"	<< setw(10) << "카메라"	<< setw(12) << "Roll"	<< setw(15) << resCamera<< setw(15) << "Threshold 2"	<< endl;
	cout << setw(3) << "2"	<< setw(10) << "진단"	<< setw(12) << "ErrCode"<< setw(15) << resdiag	<< setw(15) << " "		<< endl;
	cout << setw(3) << "3"	<< setw(10) << "버전"	<< setw(12) << "SW Ver"	<< setw(15) << SWVer	<< setw(15) << " "		<< endl;
	cout << setw(3) << " "	<< setw(10) << " "	<< setw(12) << "FW Ver"	<< setw(15) << FWVer	<< setw(15) << " "		<< endl;
	cout << setw(3) << " "	<< setw(10) << " "	<< setw(12) << "E6 Ver"	<< setw(15) << E6Ver	<< setw(15) << " "		<< endl;
	cout << setw(3) << "4"	<< setw(10) << "위상캘"	<< setw(12) << " "	<< setw(15) << resdiag	<< setw(15) << " "		<< endl;
	cout << setw(3) << "5"	<< setw(10) << "IMU"	<< setw(12) << " "	<< setw(15) << resdiag	<< setw(15) << " "		<< endl;

```

<br/>

## 결과

![image](https://user-images.githubusercontent.com/68185569/218002463-c66dc783-8de1-4220-a3b3-5d4a7f14aa33.png)
