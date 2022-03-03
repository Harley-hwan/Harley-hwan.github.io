---
layout: post
title: (MFC) 콘솔창 띄우기
subtitle: Console Screen for mfc
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c++, mfc, debug]
comments: true
---

# MFC 프로그래밍 시 콘솔창 띄우기

- 최초 작성일: 2021년 3월 3일(목)

## 목차

[TOC]

## 내용

```c++
#pragma comment(linker, "/entry:WinMainCRTStartup /subsystem:console")
```

<br/>
