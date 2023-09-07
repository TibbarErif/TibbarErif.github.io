---
title: 【Unity小技巧】解决 Unity 的 Dropdown 组件无法获取问题
date: 2021-12-29 21:14:54
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
刚写完上篇文章，没一会又踩了新坑 o(╥﹏╥)o

## 事件说明
正在制作菜单界面，需要有一个下拉菜单来筛选道具分类。
如下：
![Dropdown](https://pic.imgdb.cn/item/61cc5faf2ab3f51d9104ba60.jpg)

## 无法获取
结果发现在脚本中设置了变量：

```
public Dropdown classifyDropdown;
```

场景中的下拉菜单居然拖不进去？
试着又写了 `GetComponent<Dropdown>` 来获取该组件，却返回 `null`。

喵喵喵？

## 问题原因
其实……是我弄错组件了。
虽然名字差不多，但我用的是 TextMesh Pro 那个组件……

![问题原因](https://pic.imgdb.cn/item/61cc606f2ab3f51d91056567.jpg)

实际上要用到的是 `UI - Legacy - Dropdown` 才对！
