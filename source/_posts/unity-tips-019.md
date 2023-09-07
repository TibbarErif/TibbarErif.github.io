---
title: 【Unity小技巧】TextMesh - Pro 出现 T 字图标问题
date: 2021-12-28 10:29:56
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## TextMesh - Pro
`TextMesh - Pro` 是 Unity 收购的一款字体扩展插件，已内置到 Unity 引擎里可以直接使用。
昨天晚上在制作 UI 的时候，发现「下拉菜单」组件会显示一个 T 字的图标。

![DropDown组件出现神秘T文字图标](https://pic.imgdb.cn/item/61ca77472ab3f51d918b6b0e.jpg)

咋看之下好像是缺少了字体，并且运行游戏的时候这个 T 图标还在。
虽然 TextMesh 并不支持中文显示，但我创建的 DropDown 只有英文，所以肯定不是字体的问题。

## 模拟器环境
将游戏的测试环境改成模拟器，却发现 T 字图标没了。

![模拟器环境运行](https://pic.imgdb.cn/item/61ca77d42ab3f51d918bcac2.jpg)
## 解决方法
在模拟器下没问题，那就能排除是组件本身的问题了。
我突然想到有一天自己打开了 Gizmos 没有关掉。

![Gizmos](https://pic.imgdb.cn/item/61ca78442ab3f51d918c21ca.jpg)

将其关闭后，T 字图标果然消失了。
Gizmos 旁白有个下拉菜单，点开可以查看详情：

![Gizmos详情](https://pic.imgdb.cn/item/61ca78832ab3f51d918c49ea.jpg)

可以发现上面确实有 T 字图标。
## Gizmos 是什么？
借着这个机会，我趁机查了一下 Gizmos 的相关资料，看看它到底是个什么东西。
（竟然害我昨天晚上折腾许久……）

参考文章：[https://developer.unity.cn/ask/question/5f7961ecedbc2a0020ab01a4](https://developer.unity.cn/ask/question/5f7961ecedbc2a0020ab01a4)

简单的概括 Gizmos 就是一个画线辅助系统。
在游戏中可能存在许多「看不见」的东西，比如敌人的『攻击范围』。
为了方便调试，可以绘制一条虚拟的辅助线。

直接上代码看效果便一目了然：

```
using System;
using System.Collections;
using System.Collections.Generic;
using LitJson;
using UnityEngine;
using UnityEngine.UI;

public class Test : MonoBehaviour
{
    private void OnDrawGizmosSelected()
    {
        Gizmos.DrawWireSphere(transform.position, 5);
    }
}
```

编写上述代码，然后将脚本挂在场景中的物体上面，即可看到效果。

![虚拟线](https://pic.imgdb.cn/item/61ca7a6c2ab3f51d918db153.jpg)
只要是继承了 `MonoBehaviour` 的脚本，都可以用这个方法绘制虚拟线。

上面的代码调用了 `Gizmos.DrawWireSphere` 方法，即画一个半径为 5 的圆圈。
可是发现只有在编辑器场景中出现了圆圈，在游戏场景并没有看到圆圈，即使运行游戏也看不见。
这是由于我上面已经把 Gizmos 关掉了，只要重新打开即可：

![打开 Gizmos](https://pic.imgdb.cn/item/61ca7ad52ab3f51d918e028f.jpg)

如上所示，游戏场景中可以看到虚拟线了。

## 结尾
Gizmos 是 Unity 中的画线辅助系统，可以在调试的时候绘制虚拟线帮助开发者直观的看见「不可视之物」，例如敌人的攻击范围、子弹的运行轨迹等等。

Gizmos 官方文档：[https://docs.unity3d.com/ScriptReference/Gizmos.html](https://docs.unity3d.com/ScriptReference/Gizmos.html)