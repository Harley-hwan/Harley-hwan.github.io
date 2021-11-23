---
layout: post
title: 오버로딩(Overloading)이란? (Java)
subtitle: Overloading 개념
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [java, overloading]
comments: true
---

# 오버로딩 (Overloading)

- 최초 작성일: 2021년 11월 23일(화)



![img](https://nesoy.github.io/assets/logo/Java.jpg)

## 목차

[TOC]

## 개념

- 같은 이름의 메소드를 여러 개 가지면서 파라미터의 유형 혹은 개수를 다르게 해서 사용할 수 있는 기술이다.
- 메소드 오버로딩, 생성자 오버로딩이 있다.
- 아래와 같이 메소드명이 method로 모두 동일하지만 파라미터로 받아오는 타입이 다르므로 다른 메소드로 여겨진다는 것이다.

<br/>

```c++
package calc;

public class Test {
	// 메소드
	public void method () {
		System.out.println("나는 method()");
	}
	
	// Overloading
	public void method(int number) {
		System.out.println("나는 숫자를 받을 수 있는 method()");
	}
    
	// Overloading
	public void method(String str)) {
		System.out.println("나는 문자열을 받을 수 있는 method()");
	}
}
```

 