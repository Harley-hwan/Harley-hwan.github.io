---
layout: post
title: [MFC] 색상 슬라이드 프로그램 작성
subtitle: 슬라이더(Slider Control)의 이용
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c++, mfc, slider, rgb, windowprogramming]
comments: true
---

# [MFC] 색상 슬라이드 프로그램 작성

- 최초 작성일: 2022년 2월 8일 (화)


## 목차

[TOC]

## 실습

### 프로젝트 생성

다이얼로그 기반으로 mfc App 프로젝트를 새로 생성한다.

프로젝트명은 RGBSlide 라고 했다.

<br/>

### 다이얼로그 편집/디자인

![image](https://user-images.githubusercontent.com/68185569/152954786-dfd9ea8c-d6d4-4b26-b854-31f2c76e6137.png)

<br/>



<br/>

### 코드 작성

'MClock2.Dlg.h' 헤더 파일에 변수 선언을 위해 [Solution Explorer]-[MClock2]-[Header Files]에서 'MClock2Dlg.h' 을 더블클릭하고 다음과 같이 코드를 추가한다.

![image](https://user-images.githubusercontent.com/68185569/152735010-8171480e-1d46-467a-80bf-25f198c17e0c.png)

```c++
// CMClock2Dlg dialog
class CMClock2Dlg : public CDialogEx
{
// Construction
public:
	CMClock2Dlg(CWnd* pParent = nullptr);	// standard constructor

	CRect screen;
	int vsize, hsize;

	UINT htimer;

// Dialog Data
```

<br/>

다음으로 OnInitDialog() 함수의 변수를 초기화 시켜준다.

[Class View]-[MClock2]-[CMClock2Dlg]-[OnInitDialog()]를 더블클릭한 후 다음과 같이 코드를 추가한다.

![image](https://user-images.githubusercontent.com/68185569/152735134-267c3704-aede-41bc-81e6-9a5183134635.png)

```c++
BOOL CMClock2Dlg::OnInitDialog()
{
	CDialogEx::OnInitDialog();

	// Add "About..." menu item to system menu.
	CRect rect;
	screen.top = 0;
	screen.left = 0;
	screen.bottom = ::GetSystemMetrics(SM_CYSCREEN);
	screen.right = ::GetSystemMetrics(SM_CXSCREEN);		// 화면 크기

	htimer = SetTimer(1, 1000, NULL);

	GetWindowRect(rect);
	vsize = rect.Width();		// 프로그램의 가로 크기
	hsize = rect.Height();		// 프로그램의 세로 크기

	// IDM_ABOUTBOX must be in the system command range.
	ASSERT((IDM_ABOUTBOX & 0xFFF0) == IDM_ABOUTBOX);
	ASSERT(IDM_ABOUTBOX < 0xF000);
```

<br/>

이제는 날짜와 시간을 출력하기 위해 'WM_TIMER' 메시지를 추가한다.

[Ctrl+Shift+X] 키 혹은 [Menu]-[Project]-[Class Wizard] 를 클릭해서 클래스 마법사를 실행시켜 다음과 같이 WM_TIMER 메시지를 더블클릭해 'OnTimer' 함수를 추가하고, OnTimer()에 다음과 같이 코드를 추가한다.


```c++
void CMClock2Dlg::OnTimer(UINT_PTR nIDEvent)
{
	// TODO: Add your message handler code here and/or call default
	CTime gct = CTime::GetCurrentTime();

	CString strYear;
	CString strMonth;
	CString strDay;
	CString strTime;
	CString strYoil;

	UINT DayOfWeek[] =
	{
	   LOCALE_SDAYNAME7,   // Sunday
	   LOCALE_SDAYNAME1,
	   LOCALE_SDAYNAME2,
	   LOCALE_SDAYNAME3,
	   LOCALE_SDAYNAME4,
	   LOCALE_SDAYNAME5,
	   LOCALE_SDAYNAME6   // Saturday
	};

	strYear.Format(_T("%d 년 "), gct.GetYear());
	GetDlgItem(IDC_STATIC_YEAR)->SetWindowText((LPCTSTR)strYear);

	strMonth.Format(_T("%d 월 "), gct.GetMonth());
	GetDlgItem(IDC_STATIC_MONTH)->SetWindowText((LPCTSTR)strMonth);

	strDay.Format(_T("%d 일 "), gct.GetDay());
	GetDlgItem(IDC_STATIC_DAY)->SetWindowText((LPCTSTR)strDay);

	if (gct.GetHour() > 12) 
	{
		strTime.Format(_T("오후"));
		GetDlgItem(IDC_STATIC_AMPM)->SetWindowTextW((LPCTSTR)strTime);

		strTime.Format(_T("%d 시 %d 분 %d 초 "), gct.GetHour()-12, gct.GetMinute(), gct.GetSecond());
		GetDlgItem(IDC_STATIC_TIME)->SetWindowText((LPCTSTR)strTime);
	}
	else
	{
		strTime.Format(_T("오전"));
		GetDlgItem(IDC_STATIC_AMPM)->SetWindowTextW((LPCTSTR)strTime);

		strTime.Format(_T("%d 시 %d 분 %d 초 "), gct.GetHour(), gct.GetMinute(), gct.GetSecond());
		GetDlgItem(IDC_STATIC_TIME)->SetWindowText((LPCTSTR)strTime);
	}

	TCHAR strWeekday[256];

	::GetLocaleInfo(LOCALE_USER_DEFAULT, DayOfWeek[gct.GetDayOfWeek() - 1], strWeekday, sizeof(strWeekday));

	strYoil.Format(_T("%s "), strWeekday);
	GetDlgItem(IDC_STATIC_YOIL)->SetWindowText((LPCTSTR)strYoil);

	Invalidate();

	CDialogEx::OnTimer(nIDEvent);
}

```
<br/>
