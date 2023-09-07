---
title: 【Unity小技巧】解决 Dropdown 下拉菜单展开后子菜单被遮挡问题
date: 2021-12-30 15:32:46
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
使用 Dropdown 制作下拉菜单 UI 时，展开的子菜单会被其他 UI 遮挡。

## 解决方案
再创建一个 Canvas，并且点选 override sorting。
将 Sort Order 设置为比当前画布更大的值。
最后将下拉菜单组件挂在这个新的 Canvas 下面。

![Canvas](https://pic.imgdb.cn/item/61cd61012ab3f51d91a50ce5.jpg)

![不再遮挡子菜单了](https://pic.imgdb.cn/item/61cd61852ab3f51d91a5754d.jpg)

新的问题又来了，如何让子菜单向下展示而不是向上？
好像不能人为控制，只能让系统自己判断。