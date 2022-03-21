---
layout: post
title: (C#) 31. Interface (인터페이스)
subtitle: Syntax (문법)
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c#, unity, syntax, interface]
comments: true
---

# Interface (인터페이스) : Syntax (문법)

- 최초 작성일: 2021년 3월 21일(월)

## 목차

[TOC]

## 내용

```c#
using System;

namespace Interface
{
    class Program
    {
        abstract class Monster
        {
            // abstact : 상속받은 클래스들이 오버라이딩을 하는 것을 강요
            // 추상적으로 선언한 것이기 때문에 실제내용은 존재하지 않는 것. 정의를 할 수 없음.
            public abstract void Shout();     

            // public virtual void Shout() { };
        }

        interface IFlyable
        {
            void Fly();
        }

        class Orc : Monster
        {
            public override void Shout()
            {
                Console.WriteLine("록타르 오가르!");
            }
        }

        class FlyableOrc : Orc, IFlyable // 다중 상속 불가능
        {
            public void Fly()
            {
                
            }
        }

        class Skeleton : Monster        
        {
            public override void Shout()    // 정의하지 않으면 에러뜸. (함수안에 내용이 없으면)
            {
                Console.WriteLine("꾸에에엑!");
            }

        }

        static void DoFly(IFlyable flyable)
        {
            flyable.Fly();
        }

        static void Main(string[] args)
        {
            FlyableOrc orc = new FlyableOrc();
            DoFly(orc);
        }
    }
}

```
