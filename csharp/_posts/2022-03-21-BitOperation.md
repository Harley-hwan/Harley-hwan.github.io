---
layout: post
title: (C#) 비트 연산
subtitle: Bit Operation with C#
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c#, unity, bitoperation]
comments: true
---

# 데이터 연산

- 최초 작성일: 2021년 3월 21일(월)

## 목차

[TOC]

## 내용

```c#
using System;

namespace BitOperation
{
    class Program
    {
        static void Main(string[] args)
        {
            int num = 1;
            int id = 123;
            int key = 401;

            int a = id ^ key;
            int b = a ^ key;
            // <<   >>  &(and)   !(not)   ^(xor)   ~(not)
            num = num << 3;

            Console.WriteLine(a);
            Console.WriteLine(b);
        }
    }
}
```
