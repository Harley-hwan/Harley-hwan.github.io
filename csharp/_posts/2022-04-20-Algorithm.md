---
layout: post
title: (C#) 44. 스택과 큐
subtitle: Stack, Queue
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c#, datastructure, stack, queue]
comments: true
---

# 스택과 큐 (Stack & Queue)

- 최초 작성일: 2021년 4월 20일(수)

## 목차

[TOC]

## 내용

### 스택 (Stack)

![image](https://user-images.githubusercontent.com/68185569/164134511-2d70976d-6bd5-4896-9ff7-7cbff5617a90.png)

스택(Stack)은 '쌓다'의 의미로, 데이터를 세로로 쌓아 올린 형태의 자료구조이다.

벽돌을 위로 쌓아올렸다가 다시 치우는 걸 상상해보자. 벽돌을 쌓아올린 후에 그것을 다시 치우려면 제일 위에 있는 것부터 차례대로 빼내어야한다.

다시 말해, 가장 늦게 삽인된 것이 가장 먼저 제거된다. 이것이 스택의 특징이며 LIFO (Last In First Out)라 부른다.

<br/>

### 큐 (Queue)

![image](https://user-images.githubusercontent.com/68185569/164134895-8156c2dc-95d2-4a28-8d5d-d68bccf57455.png)

큐(Queue)는 '줄을 서다'의 의미로, 데이터를 일렬로 나열시키는 형태의 자료구조이다.

사람들이 화장실을 이용하기 위해 줄을 서고 있는 걸 상상해보자. 가장 먼저 온 사람이 차례를 기다려 화장실 사용을 위해 대기열로부터 나갈 수 있다.

다시 말해, 가장 먼저 삽인된 것이 가장 먼저 제거된다. 이것이 큐의 특징이며 FIFO (First In First Out)라 부른다.

<br/>

### C# 코드

#### Program.cs

```c#
using System;
using System.Collections.Generic;

namespace Exercise
{
    // [1] [2] [3] [4]

    // 스택:  LIFO (후입선출 Last In First Out)
    // 큐  :  FIFO (선입선출 First In First Out)

    class Program
    {
        static void Main(string[] args)
        {
            Stack<int> stack = new Stack<int>();

            stack.Push(101);
            stack.Push(102);
            stack.Push(103);
            stack.Push(104);
            stack.Push(105);

            int data = stack.Pop();
            int data2 = stack.Peek();

            
            Queue<int> queue = new Queue<int>();
            queue.Enqueue(101);
            queue.Enqueue(102);
            queue.Enqueue(103);
            queue.Enqueue(104);
            queue.Enqueue(105);

            int data3 = queue.Dequeue();
            int data4 = queue.Peek();
        }
    }
}

```

<br/>

---

### Result Video

![image](https://user-images.githubusercontent.com/68185569/164135534-4aaeeb51-0126-43a1-bffc-6933bde511aa.png)
