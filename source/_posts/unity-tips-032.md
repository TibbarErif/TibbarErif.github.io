---
title: 【Unity小技巧】字符串转化为类的实例化对象
date: 2022-02-17 17:50:25
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
本文介绍一种将字符串转化为实例化对象的方法，假设我们需要将一个字符串动态转化为某个类，再执行类的某个方法，就可以使用这种方法。
## 转换方法
网上大多都是复制粘贴一大堆代码，然后用反射机制实现，但本文介绍的是一种超级简单的方法，仅需要几行代码即可，原理是利用 C# 自带的 Activator：

```
// 假设有一个简单的类
public class Person 
{
    public string prefix;
    public string name;

    public string GetFullName() {
        return  prefix + name;
    }
}

// 引入System
using System;

// 待实例化的字符串
string component = "Person";

// 获得类的类型变量
Type type = Type.GetType(component);

// 实例化该类型的对象
var person = Activator.CreateInstance(type) as Person;

// 现在就可以调用属性和方法了
person.prefix = "aa";
preson.name = "bb";

string fullname = person.GetFullName();
```

OK，完成！