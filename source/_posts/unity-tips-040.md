---
title: 【Unity小技巧】快速选中场景中带有某个组件的对象
date: 2023-01-12 16:54:35
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
# 前言
因为要改动游戏的基础字体，这是个非常繁琐的工程，不仅费时而且费力，需要手动一个个选中场景中的 Text 然后修改字体，十分麻烦，我突然想到会不会在 Unity 中可以一键筛选出全部带有 Text 的对象，然后一键修改呢？抱着试一下的心态搜了一下，果然有！
# 方法
在 Hierarchy 的上方有一个搜索框，如下图所示：

![Hierarchy 搜索框](https://s2.loli.net/2023/01/12/GeJMrNyY9UuFI5f.jpg)

在搜索框输入：`t:text`，然后就可以看到，所有带有 `Text` 的组件全部显示出来了：

![搜索组件](https://s2.loli.net/2023/01/12/ZLhrxRw83VCz2c4.jpg)

接着，点击第一个，再按住 `Shift` 键，拖到最底部，再点击最下面的那个 Text 对象就可以全选了：

![全选 Text 对象](https://s2.loli.net/2023/01/12/14iOAqkfRrcwb7Y.jpg)

最后，在右侧的 Text 组件中修改 Font（字体）即可实现一键批量修改，工作量大大减少。