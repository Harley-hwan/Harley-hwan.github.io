---
layout: post
title: Socket error display
subtitle: socket, error, display, c++, c, mfc
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c, c++, socket, sock, errordisplay, error, mfc]
comments: true
---

#  Socket error display
- 최초 작성일: 2023년 2월 10일 (금)

## 목차

[TOC]

<br/>

## 내용

```c++
void displayerror(int nErrorCode)
{
    LPVOID lpMsgBuf;
    FormatMessage(
        FORMAT_MESSAGE_ALLOCATE_BUFFER |
            FORMAT_MESSAGE_FROM_SYSTEM |
            FORMAT_MESSAGE_IGNORE_INSERTS,
        NULL,
        nErrorCode,
        MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT), // Default language
        (LPTSTR)&lpMsgBuf,
        0,
        NULL);

    printf("%d %s\n", nErrorCode, (LPCTSTR)lpMsgBuf);
    LocalFree(lpMsgBuf);
}
```
