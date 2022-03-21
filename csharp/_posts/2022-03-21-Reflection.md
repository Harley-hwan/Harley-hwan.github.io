---
layout: post
title: (C#) Reflection (리플렉션)
subtitle: Syntax (문법)
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c#, unity, syntax, reflection]
comments: true
---

# Reflection (리플렉션) : Syntax (문법)

- 최초 작성일: 2021년 3월 21일(월)

## 목차

[TOC]

## 내용

```c#
using System;
using System.Reflection;

namespace reflection
{
    class Program
    {
        class Important : System.Attribute
        {
            string message;

            public Important(string message) { this.message = message; }
        }
        class Monster
        {
            // hp입니다. 중요한 정보입니다.        // 일반적으로, 주석은 컴파일도 안됨.
            [Important("Very Important")]        // 컴퓨터가 인식할 수 있는 주석의 개념.
            public int hp;
            protected int attack;
            private float speed;

            void Attack() { }
        }
        static void Main(string[] args)
        {
            // Reflection : X-Ray
            Monster monster = new Monster();

            Type type = monster.GetType();

            var fields = type.GetFields(System.Reflection.BindingFlags.Public
                | System.Reflection.BindingFlags.NonPublic
                | System.Reflection.BindingFlags.Static
                | System.Reflection.BindingFlags.Instance);

            foreach (FieldInfo field in fields)
            {
                string access = "protected";
                if (field.IsPublic)
                    access = "public";
                else if (field.IsPrivate)
                    access = "private";

                var attributes = field.GetCustomAttributes();

                Console.WriteLine($"{access} {field.FieldType.Name} {field.Name}");
            }
        }
    }
}
```

</br>

## Result

![image](https://user-images.githubusercontent.com/68185569/159218555-f0008336-e463-449e-b327-b71bc608ee0d.png)
