---
layout: post
title: C++에서 파이썬 스크립트 불러오기
subtitle: c, c++, python, Py_SetProgramName, Py_Initialize, PyRun_SimpleString, Py_Finalize, PyMem_RawFree
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c, c++, shellapi, windowsapi, system, command, exe, execute, HWND, windows.h, Shellapi.h, DT1-Remote]
comments: true
---

# c++ 프로젝트에서 파이썬 스크립트 불러오기
- 최초 작성일: 2023년 6월 8일 (목)

## 목차

[TOC]

<br/>

## 내용



<br/>

## 코드 설명


<br/>

<br/>

## 코드 1

```c++
#include "../Python39_64/include/Python.h"

int main(int argc, char* argv[])
{
    wchar_t* program = Py_DecodeLocale(argv[0], NULL);
    if (program == NULL) {
        fprintf(stderr, "Fatal error: cannot decode argv[0]\n");
        exit(1);
    }

    Py_SetProgramName(program);  /* optional but recommended */
    Py_Initialize();
    PyRun_SimpleString("from time import time,ctime\n"
        "print('Today is',ctime(time()))\n");
    Py_Finalize();

    PyMem_RawFree(program);
    return 0;
}
```
