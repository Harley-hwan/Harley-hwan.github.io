---
layout: post
title: (C#) 49. 트리(Tree) 알고리즘
subtitle: Tree
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c#, datastructure, graph, dijkstra, shortestpath, algorithm, maze]
comments: true
---

# 트리 (Tree)

- 최초 작성일: 2023년 4월 28일(금)

## 목차

[TOC]

## 내용



## 개념

계층적 구조를 갖는 데이터를 표현하기 위한 자료구조

- 노드(Node): 데이터를 표현
- 간선(Edge): 노드의 계층 구조를 표현하기 위해 사용

<br/>

### 트리의 예시

![image](https://user-images.githubusercontent.com/68185569/235040840-3b5e7ef4-37b2-4714-b79f-014385aaa948.png)

<br/>

## 실습 1

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Tree
{
    class TreeNode<T>
    {
        public T Data { get; set; }
        public List<TreeNode<T>> Children { get; set; } = new List<TreeNode<T>>();
    }
    class Program
    {
        static TreeNode<string> MakeTree()
        {
            TreeNode<string> root = new TreeNode<string>() { Data = "R1 개발실" };
            {
                {
                    TreeNode<string> node = new TreeNode<string>() { Data = "디자인팀" };
                    node.Children.Add(new TreeNode<string>() { Data = "전투" });
                    node.Children.Add(new TreeNode<string>() { Data = "경제" });
                    node.Children.Add(new TreeNode<string>() { Data = "스토리" });
                    root.Children.Add(node);
                }
                {
                    TreeNode<string> node = new TreeNode<string>() { Data = "프로그래밍팀" };
                    node.Children.Add(new TreeNode<string>() { Data = "서버" });
                    node.Children.Add(new TreeNode<string>() { Data = "클라" });
                    node.Children.Add(new TreeNode<string>() { Data = "엔진" });
                    root.Children.Add(node);
                }
                {
                    TreeNode<string> node = new TreeNode<string>() { Data = "아트팀" };
                    node.Children.Add(new TreeNode<string>() { Data = "배경" });
                    node.Children.Add(new TreeNode<string>() { Data = "캐릭터" });
                    root.Children.Add(node);
                }
            }
            return root;
        }
        static void Main(string[] args)
        {
            TreeNode<string> root = MakeTree();
        }            
    }
}

```

<br/>

## 결과 1

![image](https://user-images.githubusercontent.com/68185569/235040654-7c28fe41-7e56-4e14-8264-445ca76fcd5a.png)

<br/>

<br/>

## 실습 2

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Tree
{
    class TreeNode<T>
    {
        public T Data { get; set; }
        public List<TreeNode<T>> Children { get; set; } = new List<TreeNode<T>>();
    }
    class Program
    {
        static TreeNode<string> MakeTree()
        {
            TreeNode<string> root = new TreeNode<string>() { Data = "R1 개발실" };
            {
                {
                    TreeNode<string> node = new TreeNode<string>() { Data = "디자인팀" };
                    node.Children.Add(new TreeNode<string>() { Data = "전투" });
                    node.Children.Add(new TreeNode<string>() { Data = "경제" });
                    node.Children.Add(new TreeNode<string>() { Data = "스토리" });
                    root.Children.Add(node);
                }
                {
                    TreeNode<string> node = new TreeNode<string>() { Data = "프로그래밍팀" };
                    node.Children.Add(new TreeNode<string>() { Data = "서버" });
                    node.Children.Add(new TreeNode<string>() { Data = "클라" });
                    node.Children.Add(new TreeNode<string>() { Data = "엔진" });
                    root.Children.Add(node);
                }
                {
                    TreeNode<string> node = new TreeNode<string>() { Data = "아트팀" };
                    node.Children.Add(new TreeNode<string>() { Data = "배경" });
                    node.Children.Add(new TreeNode<string>() { Data = "캐릭터" });
                    root.Children.Add(node);
                }
            }
            return root;
        }

        static void PrintTree(TreeNode<string> root)
        {
            Console.WriteLine(root.Data);

            foreach (TreeNode<string> child in root.Children)
                PrintTree(child);
        }

        static void Main(string[] args)
        {
            TreeNode<string> root = MakeTree();
            PrintTree(root);
        }            
    }
}

```

<br/>

## 결과 2

![image](https://user-images.githubusercontent.com/68185569/235041589-6ce88f2f-74cd-4469-a0b4-f0db7d93e2ea.png)
