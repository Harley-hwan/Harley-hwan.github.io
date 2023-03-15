---
layout: post
title: (c++) 프린터 스풀러로 라벨 프린터에 인쇄 명령 보내기
subtitle: c++, zebra, labelprinter, windows, printer, spooler, OpenPrinterW, StartDocPrinterA, StartPagePrinter, WritePrinter, EndPagePrinter, EndDocPrinter
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c, c++, zebra, labelprinter, windows, printer, spooler, OpenPrinterW, StartDocPrinterA, StartPagePrinter, WritePrinter, EndPagePrinter, EndDocPrinter]
comments: true
---

#  Windows 프린터 스풀러로 라벨 프린터에 인쇄 명령 보내기

- 최초 작성일: 2023년 3월 15일 (수)
- 참고: https://www.zebra.com/content/dam/zebra_new_ia/en-us/manuals/printers/common/programming/zpl-zbi2-pm-en.pdf

## 목차

[TOC]

<br/>

## 내용

Windows 프린터 스풀러를 사용하여 TSC P200 라벨 프린터에 라벨링 정보와 인쇄 명령을 전송하는 C++ 프로그램이며, 코드는 다음과 같은 단계로 작동한다.

1. 프린터 이름 설정
    - std::wstring printerName = L"TSC P200";에서 프린터 이름을 설정한다. 
    - 이 예에서는 TSC P200라는 이름을 사용하였으나, 실제 사용하는 프린터의 이름으로 변경해야 한다.
2. 인쇄 명령 설정
    - std::string command에 인쇄할 라벨의 정보와 명령을 설정한다. 
    - 여기서는 30x10 mm 크기의 라벨에 'Hello, World!'라는 텍스트를 인쇄하는 예제가 포함되어 있다.
3. 프린터 열기
    - OpenPrinterW() 함수를 사용하여 프린터를 연다. 이 함수는 프린터 이름을 입력으로 받아 프린터와 연결을 설정한다. 
    - 연결이 성공하면 프린터 핸들을 반환한다.
4. 인쇄 작업 정보 설정
    - DOC_INFO_1A 구조체를 사용하여 인쇄 작업에 대한 정보를 설정한다. 
    - 이 구조체는 작업 이름, 출력 파일 및 데이터 유형을 포함한다. 
    - 이 예에서는 데이터 유형을 'RAW'로 설정하여 명령을 직접 프린터에 전송한다.
5. 인쇄 작업 시작
    - StartDocPrinterA() 함수를 사용하여 인쇄 작업을 시작한다. 
    - 이 함수는 프린터 핸들 및 DOC_INFO_1A 구조체를 입력으로 받는다.
6 인쇄 페이지 시작
    - StartPagePrinter() 함수를 사용하여 인쇄 페이지를 시작한다. 
    이 함수는 프린터 핸들을 입력으로 받는다.
7. 프린터에 데이터 쓰기
    - WritePrinter() 함수를 사용하여 프린터에 명령을 전송한다. 
    - 이 함수는 프린터 핸들, 명령 데이터 및 데이터 크기를 입력으로 받는다.
8. 인쇄 페이지 종료
    - EndPagePrinter() 함수를 사용하여 인쇄 페이지를 종료한다. 
    - 이 함수는 프린터 핸들을 입력으로 받는다.
9. 인쇄 작업 종료
    - EndDocPrinter() 함수를 사용하여 인쇄 작업을 종료한다. 
    - 이 함수는 프린터 핸들을 입력으로 받는다.
10. 프린터 닫기
    - ClosePrinter() 함수를 사용하여 프린터를 닫고, 프린터와의 연결을 해제한다. 
    - 이 함수는 프린터 핸들을 입력으로 받는다.

<br/>

이 코드를 사용하면 Windows 프린터 스풀러를 통해 프린터에 명령을 전송할 수 있다. 프린터 이름을 실제 사용하는 프린터 이름으로 바꾸고, 인쇄할 라벨의 정보와 명령을 필요에 따라 수정하여 코드를 사용할 수 있다. 

이 코드를 컴파일하고 실행하면, 설정한 라벨 정보와 프린트 명령을 TSC P200 라벨 프린터로 전송하여 인쇄가 진행된다.

이 코드는 대부분의 에러 상황을 감지하고 적절한 오류 메시지를 출력하여 사용자에게 알린다. 이를 통해 프린터 열기, 설정, 인쇄 작업 및 페이지 시작/종료 등 과정에서 발생할 수 있는 문제를 쉽게 파악할 수 있다.

정리하면, 이 코드는 Windows의 프린터 스풀러를 사용하여 TSC P200 라벨 프린터와 통신하는 C++ 프로그램이다.

프린터 이름, 라벨 정보 및 인쇄 명령을 설정한 후, 프린터와 연결하고 인쇄 작업을 시작하여 라벨을 인쇄하고 작업을 종료한다.

이 과정에서 발생할 수 있는 오류를 처리하고 사용자에게 알리는 기능도 포함되어 있다.

<br/>

<br/>

### 버전 1

```c++
#include <iostream>
#include <string>
#include <Windows.h>

int main() {
    // 프린터 이름 설정. 실제 사용하는 프린터 이름 변경
    std::wstring printerName = L"TSC P200";

    // 인쇄할 라벨의 정보와 명령을 설정
    std::string command = "SIZE 30 mm, 10 mm\n"
        "GAP 3 mm, 0\n"
        "DIRECTION 1\n"
        "CLS\n"
        "TEXT 10, 10, \"3\", 0, 1, 1, \"Hello, World!\"\n"
        "PRINT 1\n";

    // 프린터 핸들 초기화
    HANDLE hPrinter;

    // 프린터와 연결 시도. 실패하면 에러 메시지를 출력하고 종료.
    if (!OpenPrinterW(const_cast<LPWSTR>(printerName.c_str()), &hPrinter, nullptr)) {
        std::cerr << "Error opening printer: " << GetLastError() << std::endl;
        return 1;
    }

    // 인쇄 작업 정보 설정.
    DOC_INFO_1A docInfo;
    docInfo.pDocName = const_cast<char*>("TSC P200 Printing");
    docInfo.pOutputFile = nullptr;
    docInfo.pDatatype = const_cast<char*>("RAW");

    // 인쇄 작업을 시작. 실패하면 에러 메시지를 출력하고 종료.
    DWORD jobId = StartDocPrinterA(hPrinter, 1, reinterpret_cast<LPBYTE>(&docInfo));
    if (jobId == 0) {
        std::cerr << "Error starting print job: " << GetLastError() << std::endl;
        ClosePrinter(hPrinter);
        return 1;
    }

    // 페이지 인쇄를 시작. 실패하면 에러 메시지를 출력하고 종료.
    if (!StartPagePrinter(hPrinter)) {
        std::cerr << "Error starting page: " << GetLastError() << std::endl;
        EndDocPrinter(hPrinter);
        ClosePrinter(hPrinter);
        return 1;
    }

    // 설정한 라벨 정보와 명령을 프린터로 전송.
    DWORD bytesWritten;
    if (!WritePrinter(hPrinter, const_cast<char*>(command.data()), command.size(), &bytesWritten)) {
        std::cerr << "Error writing to printer: " << GetLastError() << std::endl;
        EndPagePrinter(hPrinter);
        EndDocPrinter(hPrinter);
        ClosePrinter(hPrinter);
        return 1;
    }

    // 페이지 인쇄를 종료. 실패하면 에러 메시지를 출력하고 종료.
    if (!EndPagePrinter(hPrinter)) {
        std::cerr << "Error ending page: " << GetLastError() << std::endl;
        EndDocPrinter(hPrinter);
        ClosePrinter(hPrinter);
        return 1;
    }

    // 인쇄 작업을 종료. 실패하면 에러 메시지를 출력하고 종료.
    if (!EndDocPrinter(hPrinter)) {
        std::cerr << "Error ending print job: " << GetLastError() << std::endl;
        ClosePrinter(hPrinter);
        return 1;
    }

    // 프린터와의 연결 종료.
    ClosePrinter(hPrinter);
    return 0;
}
```

<br/>

<br/>

### 버전 2 (ZPL)

```c++
#include <iostream>
#include <string>
#include <Windows.h>

int main() {
    // 프린터 이름을 설정합니다. 실제 사용하는 프린터 이름으로 변경하세요.
    std::wstring printerName = L"TSC P200";

    // 인쇄할 라벨의 정보와 명령을 설정합니다.
    std::string command = "^XA\n"
        "^MMT\n"
        "^PW203\n"
        "^LL203\n"
        "^LS0\n"
        "^FO10,10^A0N,28,28^FDHello,World!^FS\n"
        "^PQ1,0,1,Y^XZ\n";

    // 프린터 핸들을 초기화합니다.
    HANDLE hPrinter;

    // 프린터와 연결을 시도합니다. 실패하면 에러 메시지를 출력하고 종료합니다.
    if (!OpenPrinterW(const_cast<LPWSTR>(printerName.c_str()), &hPrinter, nullptr)) {
        std::cerr << "Error opening printer: " << GetLastError() << std::endl;
        return 1;
    }

    // 인쇄 작업에 대한 정보를 설정합니다.
    DOC_INFO_1A docInfo;
    docInfo.pDocName = const_cast<char*>("TSC P200 Printing");
    docInfo.pOutputFile = nullptr;
    docInfo.pDatatype = const_cast<char*>("RAW");

    // 인쇄 작업을 시작합니다. 실패하면 에러 메시지를 출력하고 종료합니다.
    DWORD jobId = StartDocPrinterA(hPrinter, 1, reinterpret_cast<LPBYTE>(&docInfo));
    if (jobId == 0) {
        std::cerr << "Error starting print job: " << GetLastError() << std::endl;
        ClosePrinter(hPrinter);
        return 1;
    }

    // 페이지 인쇄를 시작합니다. 실패하면 에러 메시지를 출력하고 종료합니다.
    if (!StartPagePrinter(hPrinter)) {
        std::cerr << "Error starting page: " << GetLastError() << std::endl;
        EndDocPrinter(hPrinter);
        ClosePrinter(hPrinter);
        return 1;
    }

    // 설정한 라벨 정보와 명령을 프린터로 전송합니다.
    DWORD bytesWritten;
    if (!WritePrinter(hPrinter, const_cast<char*>(command.data()), command.size(), &bytesWritten)) {
        std::cerr << "Error writing to printer: " << GetLastError() << std::endl;
        EndPagePrinter(hPrinter);
        EndDocPrinter(hPrinter);
        ClosePrinter(hPrinter);
        return 1;
    }

    // 페이지 인쇄를 종료합니다. 실패하면 에러 메시지를 출력하고 종료합니다.
    if (!EndPagePrinter(hPrinter)) {
        std::cerr << "Error ending page: " << GetLastError() << std::endl;
        EndDocPrinter(hPrinter);
        ClosePrinter(hPrinter);
        return 1;
    }

    // 인쇄 작업을 종료합니다. 실패하면 에러 메시지를 출력하고 종료합니다.
    if (!EndDocPrinter(hPrinter)) {
        std::cerr << "Error ending print job: " << GetLastError() << std::endl;
        ClosePrinter(hPrinter);
        return 1;
    }

    // 프린터와의 연결을 종료합니다.
    ClosePrinter(hPrinter);
    return 0;
}
```
