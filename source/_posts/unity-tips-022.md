---
title: 【Unity小技巧】UI 上面奇怪的线是什么？
date: 2021-12-30 15:23:32
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
在使用 Unity 的 Button 组件、Dropdown 组件等等，场景上会出现奇怪的“线”。
这些线到底是什么呢？

![奇怪](https://pic.imgdb.cn/item/61cd5eef2ab3f51d91a2699b.jpg)
## 线是什么？
今天无意中看到 UI 组件里面有一个特殊的字段：`navigation`（导航）。

![navigation](https://pic.imgdb.cn/item/61cd5f332ab3f51d91a2cc9f.jpg)

这些可操作的 UI 组件上都具有这种导航字段。
正如字面意思，这其实是一个导航系统。

例如在点击选中一个 Button，当前的 Button 就会被激活，此时如果你按下键盘的上下左右键，就会移动到下一个 Button 上面。

![导航UI](https://pic.imgdb.cn/item/61cd5fc92ab3f51d91a3994f.gif)

该导航有 5 种：
- None（关闭）：关闭导航。
- Automatic（自动导航）：自动识别最近的一个控件并导航到下一个控件。
- Horizontal（水平导航）：水平方向导航到下一个控件。
- Vertical（垂直导航）：垂直方向导航到下一个控件。
- Explicit（指定导航）：特别指定在按下特定方向键时从此按钮导航到哪一个控件。

## 隐藏线
如果不希望看到这些线，可以点击下方的 `visualize` 按钮，线就会消失了。