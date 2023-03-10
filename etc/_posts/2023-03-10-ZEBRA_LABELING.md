---
layout: post
title: (c++) 라벨 프린터에 TCP/IP로 프린트 명령 보내기 
subtitle: c++, zebra, labelprinter, tcp/ip, command, labeling
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c, c++, zebra, labelprinter, tcp/ip, command, labeling]
comments: true
---

#  ZEBRA 라벨 프린터에 TCP/IP로 프린트 명령 보내기
- 최초 작성일: 2023년 3월 10일 (금)

## 목차

[TOC]

<br/>

## 내용

Zebra 라벨 프린터는 ZPL(Zebra Programming Language)이라는 자체 프로그래밍 언어를 사용한다.

ZPL 명령을 통해 라벨 프린터를 제어할 수 있다.

<br/>

MFC C++ 프로그램에서는 라벨 프린터와 통신하기 위해 다음과 같은 방법을 사용할 수 있다.

### 1. 시리얼 통신 또는 TCP/IP 통신을 이용하여 라벨 프린터와 통신하기

- 시리얼 통신: 시리얼 포트를 열어 라벨 프린터와 통신합니다.
- TCP/IP 통신: 라벨 프린터의 IP 주소를 이용하여 네트워크를 통해 통신합니다.

### 2. ZPL 명령어를 생성하여 라벨 프린터로 전송하기

- ZPL 명령어는 일반적으로 ASCII 문자열로 생성되며, 라벨 프린터에게 전송됩니다.
- 생성된 ZPL 명령어를 라벨 프린터로 전송하는 방법으로는 시리얼 통신 또는 TCP/IP 통신 등이 있다.

<br/>

라벨 프린터와 통신하는 방법에 따라 코드가 달라질 수 있다. 

시리얼 통신을 이용하는 경우에는 MFC의 CSerialPort 클래스를 사용할 수 있으며, TCP/IP 통신을 이용하는 경우에는 MFC의 CSocket 클래스를 사용할 수 있다.

ZPL 명령어를 생성하는 방법은 다양하지만, 일반적으로는 문자열 연산을 이용하여 생성하게 된다.

<br/>

라벨 프린터와의 통신 및 ZPL 명령 생성에 대한 자세한 내용은 Zebra의 공식 문서를 참고한다.

<br/>

<br/>

MFC C++ 프로그램에서 ZEBRA 라벨 프린터에 TCP/IP 통신으로 라벨 프린터 명령을 주는 예시 코드

<br/>

```c++
#include <afxsock.h>

void SendLabelCommandToZebraPrinter(CString ipAddress, int port, CString labelCommand)
{
    CSocket socket;
    if (socket.Create() == 0)
    {
        AfxMessageBox(_T("Socket creation failed"));
        return;
    }

    if (socket.Connect(ipAddress, port) == 0)
    {
        AfxMessageBox(_T("Connection failed"));
        return;
    }

    if (socket.Send(labelCommand, labelCommand.GetLength()) == -1)
    {
        AfxMessageBox(_T("Sending command failed"));
        return;
    }

    socket.Close();
}

```

<br/>

위 코드에서 ipAddress 변수는 라벨 프린터의 IP 주소를, port 변수는 라벨 프린터의 포트 번호를 나타내며, labelCommand 변수는 보낼 라벨 프린터 명령을 나타낸다.

SendLabelCommandToZebraPrinter 함수를 호출하여 라벨 프린터에 명령을 보낼 수 있다.

예를 들어 다음과 같이 호출할 수 있습니다.

<br/>

```c++
CString ipAddress = _T("192.168.1.100");
int port = 9100;
CString labelCommand = _T("^XA^FO50,50^ADN,36,20^FDHello World^FS^XZ");

SendLabelCommandToZebraPrinter(ipAddress, port, labelCommand);
```

<br/>

위 코드에서 labelCommand 변수에는 라벨 프린터 명령어가 포함되어 있다. 

이 코드는 Hello World라는 문자열을 포함하는 라벨을 생성하고 이를 라벨 프린터에 전송하는 예시이다. 

라벨 프린터 명령어의 자세한 내용은 ZPL 프로그래밍 가이드를 참조.
