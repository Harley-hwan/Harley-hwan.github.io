---
layout: post
title: (C#) 다차원 배열 (multi-array)
subtitle: Data Structure
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c#, unity, datastructure, multiarray]
comments: true
---

# 다차원 배열 (Multi-array) : Data Structure

- 최초 작성일: 2021년 3월 21일(월)

## 목차

[TOC]

## 내용

```c#
using System;

namespace multiArray
{
    class Program
    {
        class Map
        {
            int[,] tiles =
            {
                { 1, 1, 1, 1, 1 },
                { 1, 0, 0, 0, 1 },
                { 1, 0, 0, 0, 1 },
                { 1, 0, 0, 0, 1 },
                { 1, 1, 1, 1, 1 }
            };

            public void Render()
            {
                for (int y = 0; y < tiles.GetLength(1); y++) 
                {
                    for (int x = 0; x < tiles.GetLength(0); x++) 
                    {
                        if (tiles[y, x] == 1)
                            Console.ForegroundColor = ConsoleColor.Red;
                        else
                            Console.ForegroundColor = ConsoleColor.Green;

                        Console.Write('\u25cf');
                    }
                    Console.WriteLine();
                }
            }
        }

        static void Main(string[] args)
        {
            Map map = new Map();
            map.Render();



           // int[,] map = new int[2, 3];

            //[ . . ]
            //[ . . . . . . ]
            //[ . . . ]
            //int[][] a = new int[3][];
            //a[0] = new int[3];
            //a[1] = new int[6];
            //a[2] = new int[2];

            //a[0][0] = 1;

            /*
            // 다차원 배열
            int[] scores = new int[5] { 10, 30, 40, 20, 50 };

            //int[,] arr = new int[2, 3] { { 1, 2, 3 }, { 1, 2, 3 } };
            int[,] arr = new int[,] { { 1, 2, 3 }, { 1, 2, 3 } };

            arr[0, 0] = 1;
            arr[1, 0] = 1;
            

            // 2층 [ . . . ]
            // 1층 [ . . . ]
            */
        }
    }
}
```

<br/>

## Result

![image](https://user-images.githubusercontent.com/68185569/159217210-7a97dfa6-733f-4cf1-bd6b-b8e841abbe67.png)
