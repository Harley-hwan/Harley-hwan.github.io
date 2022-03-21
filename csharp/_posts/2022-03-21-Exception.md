---
layout: post
title: (C#) Exception (예외 처리)
subtitle: Syntax (문법)
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c#, unity, syntax, exception]
comments: true
---

# Exception (예외 처리) : Syntax (문법)

- 최초 작성일: 2021년 3월 21일(월)

## 목차

[TOC]

## 내용

```c#
using System;

namespace exception
{
    class Program
    {
        class TestException : Exception
        {

        }

        static void TestFunc()
        {
            throw new TestException();
        }

        static void Main(string[] args)
        {
            try
            {
                // 1. 0으로 나눌 때
                // 2. 잘못된 메모리를 참조 (null)
                // 3. 오버플로우
                
                // throw new TestException();
                TestFunc();
            }
            catch (DivideByZeroException e) 
            {

            }
            catch (Exception e)
            {

            }
            finally
            {
                // DB, 파일 정리 등등
            }
        }
    }
}


```
