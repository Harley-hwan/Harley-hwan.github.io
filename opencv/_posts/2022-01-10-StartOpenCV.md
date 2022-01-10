---
layout: post
title: opencv 시작하기 (c++)
subtitle: visual studio를 활용하여 c++로 opencv 시작해보자.
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [opencv, c++, vs, visual studio]
comments: true
---

# OpenCV (Open Source Computer Vision)

- 최초 작성일: 2022년 1월 10일(월)

![image](https://user-images.githubusercontent.com/68185569/148713016-ce9aba13-75d8-49e0-bad4-f9ec805ef808.png)


## 목차

[TOC]

## 개념
OpenCV(Open Source Computer Vision)은 실시간 컴퓨터 비전을 목적으로 한 프로그래밍 라이브러리이다. 원래는 인텔이 개발하였다. 

실시간 이미지 프로세싱에 중점을 둔 라이브러리이다. 인텔 CPU에서 사용되는 경우 속도의 향상을 볼 수 있는 IPP(Intel Performance Primitives)를 지원한다. 

이 라이브러리는 윈도우, 리눅스 등에서 사용 가능한 크로스 플랫폼이며 오픈소스 BSD 허가서 하에서 무료로 사용할 수 있다. 

OpenCV는 TensorFlow, Torch / PyTorch 및 Caffe의 딥러닝 프레임워크를 지원한다.

<br/>

## 설치
아래의 링크를 통해 opencv 최신 버전을 다운로드할 수 있다.

https://sourceforge.net/projects/opencvlibrary/

![image](https://user-images.githubusercontent.com/68185569/148712866-80a36699-d49f-47f3-9933-502667aa6a80.png)


![image](https://user-images.githubusercontent.com/68185569/148713163-935351dd-35be-482a-bf3e-08590f514cda.png)

다운로드하고, 위와 같은 파일을 실행하면 아래와 같은 창이 뜨고, 설치를 원하는 경로를 지정한 후 'Exatract' 버튼을 누른다. 그냥 압축풀기하는 거랑 같은 거다.

<br/>

![image](https://user-images.githubusercontent.com/68185569/148713134-db5f767f-08ae-4a03-afc5-04ef028ad0b5.png)

그러면 opencv 폴더가 생성이 된다.

그러면 이제 간단한 실습을 위해 Visual Studio를 실행하여 빈 프로젝트(Empty Project)를 하나 생성해보자.
나는 프로젝트명을 opencvEX로 했다. (원하는 걸로 하면 됨.)

![image](https://user-images.githubusercontent.com/68185569/148717767-51a42273-3c93-4b78-bfce-3c94fe6825c3.png)

![image](https://user-images.githubusercontent.com/68185569/148717865-3f52a895-7a67-49f3-a928-5fcccdc586c6.png)

빈 프로젝트를 하나 생성했으면, 이제 아까 다운로드했던 opencv를 Visual Studio와 연동시켜줘야한다.

![image](https://user-images.githubusercontent.com/68185569/148718070-6ac84346-020d-478f-996e-663904c20b79.png)

위의 사진처럼 Project > Property를 누르면 아래 창이 뜨고, 아래 사진에서 보이는 것처럼 본인이 opencv를 설치했던 경로를 추가해준다. (opencv_world455d.lib 의 경우에는 각자 설치한 버전마다 상이할 수 있음)

![image](https://user-images.githubusercontent.com/68185569/148718173-d9eadcc5-5461-43c0-8e00-0af75f19acfc.png)
![image](https://user-images.githubusercontent.com/68185569/148718165-f713ea5f-f76b-4535-bee1-02631b901ca6.png)
![image](https://user-images.githubusercontent.com/68185569/148718183-76abf632-4082-4868-8f52-4384b66b4606.png)

<br/>

설정을 완료했으면 아래의 코드를 입력해보고, 빨간 줄이 뜨며 에러가 뜨면 설정이 잘못된 것이고 에러가 사라지면 잘 된것이다.

![image](https://user-images.githubusercontent.com/68185569/148719308-24bf3323-72f8-46b8-90a6-f5eb9b7cb75c.png)

자, 이제 그러면 간단한 코드를 통해 이미지를 띄워보는 예제를 실행해보자.

시작하기에 앞서 이미지 파일을 아무거나 하나 준비하자. 그 다음 아래의 사진처럼 솔루션 탐색기(Solution Explorer)에서 현재 실행중인 프로젝트명(opencvEX)에서 마우스 우측 버튼을 누르면 oepn Folder in File Explorer를 클릭하면 현재 프로젝트의 파일 위치를 열 수 있다. 그러면 거기다 이미지 파일을 저장해두자.

![image](https://user-images.githubusercontent.com/68185569/148719296-24b3c0b3-7c02-4e5f-a5f8-4f865896bdad.png)

<br/>

이제 직접 아래의 코드를 입력하고 F5 키를 눌러 실행해보자.

```c++
#include <opencv2/imgcodecs.hpp>
#include <opencv2/videoio.hpp>
#include <opencv2/highgui.hpp>

#include <iostream>
#include <stdio.h>

using namespace cv;
using namespace std;


int main(int ac, char** av) {

	Mat img = imread("lion.png"); // 임의로 이미지 파일하나를 다운로드하여 	

	imshow("img", img);
	waitKey(0);		// 이미지 로드 후 띄우고 바로 작업 종료 하지 않고, 대기한다. 사용자의 요청을 기다린다.

	return 0;
}
```

 
