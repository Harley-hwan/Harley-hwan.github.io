---
layout: post
title: 파일 오픈 시, 파일명 설정 방법
subtitle: c++, c, file, fstream, ofstream, ifstream, fileopen, filename
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c, c++, filename, file, fstream, ifstream, ofstream]
comments: true
---

# 파일 오픈 시, 파일명 설정 방법
- 최초 작성일: 2023년 2월 10일 (금)

## 목차

[TOC]

<br/>

## 내용

```c++
	CString fileName;
	fileName.Format(_T("Result_%s.txt"), Serialno.c_str());
	fout.open(fileName_res);
```

<br/>

```c
  	char filename[255];
	sprintf_s(filename, "Result_%s.txt", Serialno);
	fout.open(filename);

```
<br/>
