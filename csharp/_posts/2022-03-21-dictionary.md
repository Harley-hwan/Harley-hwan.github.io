---
layout: post
title: (C#) 딕셔너리 (Dictionary)
subtitle: Data Structure
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c#, unity, datastructure, dictionary]
comments: true
---

# 딕셔너리 (Dictionary) : Data Structure

- 최초 작성일: 2021년 3월 21일(월)

## 목차

[TOC]

## 내용

```c#
using System;
using System.Collections.Generic;

namespace Dictionary
{
    class Monster
    {
        public int id;
        public Monster(int id) { this.id = id; }

    }
    class Program
    {
        static void Main(string[] args)
        {
            List<int> list = new List<int>();

            // Hashtable
            // 아주 큰 박스 [ . . . . . . . ] 1만개 (1~10000)
            // 박스 여러개를 쪼개 놓는다. [1~10] [11~20] [ ] [ ] [ ] [ ] [ ] 1천개
            // 777번 공?
            // 메모리 손해
            // [메모리를 내주고, 성능을 취한다.]

            // ID 식별자
            // Key -> Value
            // Dictionary

            Dictionary<int, Monster> dic = new Dictionary<int, Monster>();

            for (int i = 0; i < 10000; i++)
            {
                dic.Add(i, new Monster(i));
            }

            Monster mon;
            bool found = dic.TryGetValue(7777, out mon);

            dic.Remove(7777);
            dic.Clear();


            // Monster mon = dic[5000];

            //dic.Add(1, new Monster(1));

            //dic[5] = new Monster(5);
        }
    }
}
```
