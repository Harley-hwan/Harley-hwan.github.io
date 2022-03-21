---
layout: post
title: (C#) 13. 구구단
subtitle: multiplication table with C#
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c#, unity, codeflowcontrol, multiplicationtable]
comments: true
---

# 구구단

- 최초 작성일: 2021년 3월 21일(월)

## 목차

[TOC]

## 내용

```c#
using System;

namespace ex1
{
    class Program
    {
        static void Main(string[] args)
        {
            // 구구단
            for (int i = 2; i <= 9; i++)
            {
                for (int j = 1; j <= 9; j++)
                {
                    Console.WriteLine($"{i} * {j} = {i * j}");
                }
                Console.WriteLine();
            }
        }
    }
}

```
