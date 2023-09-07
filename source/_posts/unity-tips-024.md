---
title: 【Unity小技巧】LitJSON 与浮点数精度问题
date: 2021-12-31 20:16:59
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
游戏中许多数据都要保存下来，一般都是保存成 json 格式的文件。
而 Unity 自身并没有提供什么强大的 JSON 插件，需要我们自己去下载。

C# 的 JSON 插件一般都是选择：Newtonsoft.Json 和 LitJSON。
我选择的是 LitJSON。
## float 问题
LitJSON 无法保存 float 类型的浮点数。
因此需要改成 double 或者 decimal。

最开始的时候我以为用 double 就行了。
但是今天在设计装备随机属性系统上，发现有精度的坑。
## 精度问题
假如有个 float 的数据：1.3 要保存成 JSON 文件。

```
public class FormatterKeyValue
{
    public string key;
    public double value;
}
```

声明一个用来格式化数据的类，然后用 LitJSON 把类转化成 json 字符串。
结果却发现输出的是诸如：1.2999……的结果。

也就是说因为类型转换导致精度出现了问题。
## 解决方法
这是因为 double 的精度还不够，可以改用精度更高的 decimal 类型来保存数据。
虽然 decimal 也有精度问题，但保存两位小数的浮点数已经不会再出现精度问题了。