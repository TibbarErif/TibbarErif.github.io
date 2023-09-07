---
title: 【Unity小技巧】构建舒适的开发环境
date: 2021-12-22 17:47:23
tags: 
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
Unity 在不同的平台上面会有一些不同的地方，比如文件夹的位置或者权限；除此之外，对于一些资源文件，在调试过程可能需要反复替换，例如图集，即使修改其中一张图片也得重新打包一次，这样就十分麻烦了，如果能在调试过程直接使用散图，那就不需要重新打包了。

构建一个舒适的开发环境十分重要，可以大大的减轻开发者的心智负担。

## 开发环境
一般来说，项目可分为：线上环境（product）、预发布（Pre-release）环境和调试环境（develop）。

- 线上环境：即正式发布后的环境，用户实际体验的程序
- 预发布环境：即在内部测试的环境，用于在上线前进行的测试，如果没有发现 BUG，就可以直接发布到线上版本了
- 调试环境：即程序员开发过程中使用的环境

通常的开发框架会提供一个全局变量以便开发人员判断所处的环境，在 Unity 中，它只是一个工具，并没有线上环境的说法，至于当前的环境是什么，需要我们自行定义。

## 预编译指令
预编译指令是 Unity 内置的一些常量，可以用来判断某些特殊的环境。

Unity 的预编译指令：[https://docs.unity3d.com/cn/2019.4/Manual/PlatformDependentCompilation.html](https://docs.unity3d.com/cn/2019.4/Manual/PlatformDependentCompilation.html)
（所有的指令都可以在这个手册查询）

预编译指令的使用方法：

```
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Test : MonoBehaviour
{
    private void Start()
    {
#if UNITY_EDITOR
        Debug.Log("Unity Editor");
#endif
    }
}
```

预编译指令与通常的脚本代码不一样，写起来十分之丑，没有对齐，语法也与代码不一样。
上述代码：

```
#if UNITY_EDITOR
        Debug.Log("Unity Editor");
#endif
```
`UNITY_EDITOR` 是 Unity 预定义好的一个常量，它用来判断当前的环境是否是在 Unity 的编辑器中，编辑器环境就是你用 Unity 打开游戏项目时的环境。

除了判断是否在编辑器环境，还可以判断当前的系统：

```
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Test : MonoBehaviour
{
    private void Start()
    {
#if UNITY_EDITOR_WIN
        Debug.Log("Window");
#elif UNITY_EDITOR_OSX
        Debug.Log("Mac");
#endif
    }
}
```

当你的电脑系统是 Window 时，就会输出 Window；如果你用的是苹果电脑（Mac），那么就会输出 Mac。

## 如何使用？
通过上述的示例代码，我们可以通过预编译指令区分不同的开发环境和运行环境。
为了让我们的开发环境变得更加舒适，可以参考下面的方法。

### 加载资源
游戏开发中经常要用到【图集】，所谓图集就是将许多张图片合并成一张，这样可以提高处理效率，减少 DrawCall。对游戏的优化来说十分重要，但是对开发来说就十分头疼了。

假如我们要修改其中一张图片，整个图集就得重新打包，这是十分麻烦的，为了解决这个问题，我们可以区分【编辑器环境】与【游戏运行环境】，然后根据不同的环以不同的方式加载资源，下面是伪代码示例：

```
public Sprite[] LoadSprites(string path)
{
#if UNITY_EDITOR
        // ...加载散图
#else
        // ...加载图集
#endif
}
```

如此一来，我们调试的时候就可以直接替换掉修改的图片，而不需要重新打包成图集了。

### 显示不同的 UI
假设我们的游戏是跨平台的，支持 PC 端和移动端。
在 PC 端可以用键盘控制角色的移动，但是在移动端就不行了，而是要创建一个虚拟摇杆。

伪代码示例如下：

```
#if UNITY_EDITOR_WIN
    // ... 键盘控制
#elif UNITY_IOS
    // ... 虚拟摇杆
#endif
```

通过判断当前的环境来实现对应的控制方法。
这样我们就不需要给不同平台打包不同的版本，只需要直接打包一份即可实现自适应。

### 环境兼容问题
Unity 的一些文件夹，在不同的平台权限是不一样的。
例如：`Application.streamingPath` 在安卓下是读取不到的，对于不同平台的一些差异，也可以通过预编译指令来特殊处理。

### 方便调试
作为开发者，我们肯定不会像玩家那样正常打怪升级。
而是需要内置一个【作弊器】来直接调试游戏的参数，比如打开什么开关或者直接升到 99 级。
作弊器不能让玩家直接打开，仅能放在编辑器环境下面调用。

对于这些既不想在游戏发布时删掉，又不想让玩家直接看到的功能，也可以用预编译指令来处理。

## 结尾
除了上面介绍的外，预编译指令可以实现许多有趣的东西，这就要看自己的脑洞有多大了！
总之，通过预编译指令搭建一个舒舒服服的开发环境吧~