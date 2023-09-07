---
title: 【Unity小技巧】精灵 SpriteRender 点击事件
date: 2022-01-07 16:00:43
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
Unity 的 UI 可以直接添加 Button 来触发点击事件。
而精灵并不是 UI，无法用 Button 的方式添加事件。

让精灵也可以被点击，可以参考本文的方法。
## 射线检测
UI 之所以能够被检测到点击事件，是因为在 Canvas 上面有一个 Graphic Raycaster 组件在进行射线检测，然后发送给 EventSystem 处理。

但是 SpriteRender 无法用 Canvas 的射线进行检测，我们可以在 Camera（摄像机）上面添加射线检测组件 Physics 2D Raycaster

![Physics 2D Raycaster](https://pic.imgdb.cn/item/61d7f47f2ab3f51d91ff2d8c.jpg)
## 点击事件
接着给需要点击的精灵添加 Collider（碰撞体）：

![添加碰撞体组件](https://pic.imgdb.cn/item/61d7f4d52ab3f51d91ff74d9.jpg)

接着再添加 EventTrigger（事件触发器）组件：

![EventTrigger](https://pic.imgdb.cn/item/61d7f5092ab3f51d91ffa741.jpg)

EventTrigger 里可以设置不同的鼠标事件：

- Pointer Enter：鼠标滑入
- Pointer Exit：鼠标离开
- Pointer Down：鼠标按下
- Pointer Up：鼠标弹起

编写脚本：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MapBase : MonoBehaviour
{
    public string mapName;

    public void OnPointEnter()
    {
        Debug.Log("进入");
    }

    public void OnPointExit()
    {
        Debug.Log("离开");
    }

    public void OnPointDown()
    {
        Debug.Log("按下");
    }
}
```

分别绑定对应的事件即可。