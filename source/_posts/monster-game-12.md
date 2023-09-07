---
title: 《名为怪物的游戏》框架开发篇（场景系统）
date: 2021-08-10 22:12:37
tags:
 - unity
 - 开发技术

categories:
 - 新游开发
 - 名为怪物的游戏
---

## 前言
游戏框架可以理解为一个具有自动处理功能并且有着丰富可调用方法的类库。

提前写好各种通用方法，存入“框架”的方法库，然后开发就可以直接调用。
其实就是利用了“复用性”原则，把一些可以反复使用的方法抽取出来，作为全局调用类。

> 面向“复用”编程

方法库只是框架的基本功能，最主要的还是框架对于逻辑的自动处理。
比如切换场景时的淡入淡出效果，传送到某个地图，自动判断是否要触发剧情事件等等。

开发者只要写好触发条件，框架就能自动处理一些繁琐的过程。

因为游戏的定制性很强，所以网上没有什么好的框架，但是有许多插件包可以使用。
如果是别人开源好用的插件包，也可以直接引入框架中使用，减少自己的开发时间。

> 例如加密系统，需要对存档进行加密防止被本地修改，就要用到第三方的加密插件，当然也可以自己来写加密方法，但是自己写的无法跟专业人士相比，已经有十分成熟的开源插件，就没必要自己重复造轮子。原创系统也不会因为使用了别人的插件包就不叫做“原创”了，现代主流的框架基本都是依赖第三方包，如 PHP 的 composer，JavaScript 的 npm 等等，正是因为站在巨人的肩膀上，我们才能往更高的地方前进。

## 参考资料

游戏设计模式：[https://gpp.tkchu.me/introduction.html](https://gpp.tkchu.me/introduction.html)

## 游戏框架
游戏框架相当于工厂的流水线，只要设计好流水线，工厂的生产效率就能大幅提升。
框架的作用就是帮助开发者减少重复开发的时间，提高类的复用性。

框架的代码不应该与游戏本身的逻辑存在耦合。
耦合即有关联的意思，也就是说，框架应该是可以独立出来用在任何一个新游戏上面的。
而不是说只为了这个游戏开发框架。

除此之外，框架还将复杂的操作封装起来，只暴露出一个简单易调用的方法。
因此使用框架的时候，只需要关注开发游戏本身，而不需要关注内部是如何实现的。

那么接下来就一边学习一边尝试开发出一套属于自己的游戏框架。

## 模块设计
一套 RPG 游戏的框架包括：

- 场景系统
- 角色系统
- 装备系统
- 战斗系统
- 敌人系统
- 事件系统
- 任务系统
- ……

诸如此类，以模块进行区分，逐一实现各个功能。

## 规范开发分支
代码管理工具用的是 Git，因为是开发新的系统框架，所以单独切换出一条分支进行开发。

![分支切换](https://pic.imgdb.cn/item/610f534f5132923bf8d36cfb.jpg)

新版本取名为 `v1.1.0`，中版本号更新一位。
等系统更新完成再合并到 `master` 主分支。

然后推送到仓库服务器：

```
git push --set-upstream origin v1.1.0
```

这样就单独创建了一条新的分支，框架的开发在这条分支上进行。
代码仓库使用的是：[http://coding.net](http://coding.net)
也可以使用 Github，不过不稳定，国内有时候要翻墙才能访问。

除此之外，国产的还有：[https://gitee.com/](https://gitee.com/)
其他的我就没使用过了，国产的也不是很稳定，偶尔也会出现推送不上去或拉不下来的情况，但是访问速度比国外的快得多。

![代码仓库](https://pic.imgdb.cn/item/610f54435132923bf8d4a4f6.jpg)

为了提高效率，编写了一个 `shell` 脚本用来自动提交代码：

```
#!/bin/bash
git add .
git commit -m "shell_$1"
git push
echo "shell 快捷提交___$1 上传完成...\n"
```

这样每次提交代码只需要：

```
./autoloadSave.sh 测试
```

直接省略了输入 git 命令，懒人必备！

## 场景系统概述
游戏中分为主场景（负责全局的逻辑处理）以及地图。

![场景系统](https://pic.imgdb.cn/item/610f40de5132923bf8bb687a.jpg)

### 主场景
主场景就是 Unity 的 Scene（场景）。
游戏中只有一个主场景，其他的地图全部使用 Prefab（预制体）来实现。
这其实就是框架中常说的“单一入口”。

Unity 中的 Scene：

![Scene](https://pic.imgdb.cn/item/610f43a35132923bf8bf3213.jpg)

Unity 中的 Prefab：

![Prefab](https://pic.imgdb.cn/item/610f43c15132923bf8bf583e.jpg)

### 地图的切换原理
Prefab 放在 Resources 文件夹进行动态加载，主场景就是玩家当前所在的场景。
玩家切换地图时，并不是场景的切换，而是销毁当前地图的节点，然后通过预制体创建新地图的节点。

通常的游戏开发是一个地图就创建一个 Scene。
然后调用切换场景的方法：

```
SceneManager.LoadScene("MiniGame");
```

这样是真正的场景切换。
如果是体量比较小的游戏，没有几个场景就可以这么做。
但是我们的新游戏是长篇的 RPG，粗略的估计地图也有好几百个。

创建一个新的场景，就需要重新配置 Canvas（摄像机）和场景控制器节点。
这么做非常麻烦，因此把地图作为 Prefab（预制体）动态加载的方法就可以复用主场景中的 Canvas 和控制器节点。

> 单一场景有一个缺点就是很容易出现内存泄露的情况，所谓内存泄露就是没用的变量或者资源没有及时销毁从而白白占用系统内存空间。如果是 Unity 自带的 SceneManager 来切换场景，变量及资源会在切换场景时自动回收，但是单一场景，所有的内存回收都要自己操作，如果不小心忘记了，就会造成内存泄露，结果就是游戏玩得越久，越变越卡。

### 主场景的实现
主场景主要负责场景相关逻辑的处理。
玩家开始游戏后，只会载入一次主场景，然后由主场景判断当前需要加载的地图。

比如玩家打开游戏，载入主场景，此时显示的是标题界面。
如果玩家选择开始新游戏，就会载入游戏的初始地图；
如果玩家选择的是读取存档，就会加载存档数据的地图。

除了控制地图的加载，初始资源的加载也是在主场景进行的。
例如：本地化的语言文本，游戏的常驻资源等等。

编写 `FR_MainScene` 方法：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace FR_Scene
{
    public abstract class FR_MainScene : MonoBehaviour
    {
        protected void Awake()
        {
            InitMainSceneAction();
        }

        protected void Start()
        {
            OnLoadAction();
        }

        protected abstract void InitMainSceneAction();
        protected abstract void OnLoadAction();
    }
}
```

`InitMainSceneAction` 方法是场景的初始化方法，用于加载配置文件以及其他资源。
`OnLoadAction` 方法在载入主场景时的处理方法，例如打开游戏时载入标题画面。

这两个方法需要在子类进行实现。

> 由于是编写游戏框架，所有框架类都需要加上命名空间，这样做的好处是以后可以单独抽取出来，下次如果还要开发新游戏就可以直接把这套框架导入到新游戏的工程中。框架与当前游戏的逻辑进行解耦，只抽取出公共的方法，具体的实现通过继承接口或者抽象类的方式来完成。

这里使用了 `abstract` 来声明方法应当在子类进行实现。实际上在子类直接调用 `Start` 和 `Awake` 也没什么差别，抽象方法的作用是“约束”，也就是要子类强制按照规定好的标准写代码，下文也会遵循这样的设计。

> 框架的作用就是制定一些列的标准，使开发者能够按照统一的要求进行开发。

主场景的代码现在这样就够了，接下来编写地图系统。

## 地图系统
地图系统就比较复杂了。

一个游戏地图包括：地图中的美术资源、角色、NPC、可调查事件。
逻辑的处理又包括：查找地图中的节点，自动触发事件的处理。
还有操作方面的处理：监听玩家按键（如打开菜单、控制玩家移动）。

### 美术资源
地图的美术资源需要分层，比如桌子是在地板的上面。
地图的分层可以理解为 Photoshop 的图层。

![图层](https://pic.imgdb.cn/item/610f4ea75132923bf8cd67ed.jpg)

例如，玩家走到桌子前面会被遮挡。

![遮挡效果](https://pic.imgdb.cn/item/610f4f005132923bf8cddadb.jpg)

这其实就是桌子的图层比玩家所在的图层高，所以桌子显示在上层，玩家显示在下层，所以被遮挡了。
Unity 提供了 Sort 字段来控制图层的层级关系：

![sort](https://pic.imgdb.cn/item/610f4f6c5132923bf8ce6a2c.jpg)

该值越高，图层就显示在上方。
利用该值来控制地图的层级关系。

> 这里将使用 100 为单位，比如底层是 0，第一层就是 100，第二层是 200，这样做的好处是可以设置 101、102 这样的值来进行更加细致的分层。

为了方便直观的看出图层，每一个地图都有几个空的根节点 `Div_1`、`Div_2` 以此类推。

![空的根节点](https://pic.imgdb.cn/item/610f501e5132923bf8cf4a39.jpg)

`sort` 值为`100~199` 的就放在 `Div_1`，值为 `200~299` 的就放在 `Div_2`，玩家控制的角色放在第三层就可以了，值为 399（其他的 NPC 节点也在第三层，主角是最高的 399，这样就可以显示在 NPC 的上方）。

### 地图中的角色
先创建一个角色节点，给角色加上刚体，然后用物理学的方式来控制玩家移动，再加上 Unity 的动画系统，就实现了角色行走+移动动画。但这些并不是框架类要实现的，因为这已经是与游戏本身相关的逻辑了。

> 通常的 RPG 游戏有四方向行走，但是我们的新作只能左右移动，这不是“通用”的，所以不能在框架里写角色的移动逻辑。

框架中的角色类只要最基本的方法，操作控制需要抽取出来，让子类自行实现。

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace FR_Scene
{
    public abstract class FR_Character : MonoBehaviour
    {
        private void Awake()
        {
            InitCharacterAction();
        }

        protected abstract void InitCharacterAction();
    }
}
```

框架的 `FR_Character` 类只抽取最基本的方法，暂时这样就可以了。
这个类不仅是玩家控制的角色要继承的，也可以是地图上 NPC 的父类。

还有一些通用的方法，例如头上显示的心情气泡：

![显示气泡](https://pic.imgdb.cn/item/610f96975132923bf8463606.jpg)

不管是 NPC 还是玩家控制的角色都可以显示气泡。
这些就是公共方法，可以全部在 `FR_Character` 中实现，然后子类就可以方便的调用了。

上面只声明了一个初始化方法 `InitCharacterAction`，实际上的角色类比这个复杂得多，本文只记录开发框架的思想，而不是记录开发框架的具体实现，因此不会把要用的代码全部贴上，只展示部分示例，下文同理。

### 地图控制器
角色移动到某张地图的时候，需要判断是否满足条件自动触发剧情，以及更新玩家当前存档所在位置，并且根据是否触发事件来决定是否创建或隐藏角色的节点。这些逻辑的判断就是地图控制器需要做的事情，下面是一个简单的地图抽象类。

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace FR_Scene
{
    public abstract class FR_Map : MonoBehaviour
    {
        protected string mapName;
        [HideInInspector]
        public FR_MainScene mainScene;
        [HideInInspector]
        public FR_Player player;

        private void Awake()
        {
            InitMapAction();
        }

        private void Start()
        {
            OnLoadAction();
        }

        protected abstract void InitMapAction();
        protected abstract void OnLoadAction();

        /**
         * 销毁地图
         */
        public abstract void DestroyMap();

        /**
         * 创建玩家节点
         */
        public abstract FR_Player CreatePlayer();
    }
}
```

#### 地图节点
地图是由一个根节点以及许多个子节点构成的：

![地图的节点关系](https://pic.imgdb.cn/item/610f9b015132923bf85471c1.jpg)

假如要控制一个 NPC 的移动，就必须要获得这个节点的对象。
原来的做法是在场景中先将 NPC 放在屏幕外面看不见的地方。

![地图中的NPC](https://pic.imgdb.cn/item/610f9b425132923bf85542ee.jpg)

地图的脚本里添加几个 `GameObject` 类型的变量用来保存节点：

```
public GameObject npcBaGu, npcMysticMan, npcLouLuo;
```

然后在 Unity 的面板中，将这些 NPC 逐个拖到组件的变量里。

![组件中的节点](https://pic.imgdb.cn/item/610f9bbc5132923bf856a884.jpg)

这样在脚本中就可以用 `npcBaGu, npcMysticMan, npcLouLuo` 这三个变量来控制节点了。
一般的游戏基本都是这么做的，这样做没什么不对，但却有效率上的问题。
比如在一个 NPC 很多的地图：

![NPC很多的地图](https://pic.imgdb.cn/item/610f9cd05132923bf859df19.jpg)

整个场景就变得乱糟糟的，每一个 NPC 都要单独创建一个节点，而且还必须放到地图里面并且一个个拖到脚本组件上。切换到这个场景的时候，这些节点都会加载到内存里，浪费内存资源不说，时间一长，哪些 NPC 对应哪个剧情都难以分辨，极大的影响开发效率。

为了解决这个问题，地图中的 NPC 节点需要改成动态生成的方式。
剧情需要的时候进行创建，不需要的时候就不创建。
在创建 NPC 节点的时候，将其加入到 `Dictionary<string, GameObject>`（字典）中。
这样地图控制器就能通过节点名称来获取到地图中指定节点。

##### NPC 节点
这里又有新的问题，如果每一个 NPC 节点都做成 Prefab（预制体），那么当 NPC 多了起来又会无法分辨哪个是哪个。我不希望一个文件夹里存放着成百上千个 NPC 的预制体，因为这样会增加项目的管理难度。

NPC 节点真的需要单独创建吗？
仔细想想的话，每一个 NPC 其实都存在许多共通的部分。
比如每一个 NPC 都有行走图，都有移动的方法，玩家与 NPC 接触时，按下调查键都可以进行对话。

这样就可以把 NPC 节点的公共部分抽取出来，子类只需要单独编写特有的事件处理即可。

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace FR_Scene
{
    public abstract class FR_Character : MonoBehaviour
    {
        private void Awake()
        {
            InitCharacterAction();
        }

        protected abstract void InitCharacterAction();

        /**
         * 设置位置（本地坐标）
         */
        public void SetLocalePosition(float x, float y, float scaleX = 0)
        {
            Vector3 pos = transform.position;
            pos.x = x;
            pos.y = y;
            transform.position = pos;

            if (scaleX != 0)
            {
                SetLocalScale(scaleX);
            }
        }

        public void SetLocalScale(float x, float y = 1)
        {
            Vector3 scale = transform.localScale;
            scale.x = x;
            scale.y = y;

            transform.localScale = scale;
        }
    }
}
```

上面是一个简单的例子，里面包含了设置节点坐标和朝向的方法。

##### 角色节点
同样地，玩家控制的角色节点，也需要封装一个抽象类：

```
using UnityEngine;
using System.Collections;

namespace FR_Scene {
    public abstract class FR_Player : FR_Character
    {
    
    }
}
```

目前创建这样一个空壳就可以了。

##### 在游戏中调用
创建一个简易的测试地图：

![测试地图](https://pic.imgdb.cn/item/6111f2265132923bf8bfbf70.jpg)

然后将地图节点拖到 Resources 目录做成预制体：

![地图预制体](https://pic.imgdb.cn/item/6111f7eb5132923bf8c972d4.jpg)

因为是重构的系统，为了避免与原来的代码产生冲突，我新建了一个 `Reactor` 文件夹用来存放使用框架的代码。

![新系统的代码](https://pic.imgdb.cn/item/6111f24e5132923bf8bfffa7.jpg)

分别创建三个类：`AbstractMap`、`MainScene`、`PlayerCharacter`。

`AbstractMap` 类是游戏中实际地图的基类，包含了地图需要用的基本方法。如获取角色所在图层以及创建玩家节点和销毁地图的方法。

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using FR_Scene;

namespace Refactor.Scene
{
    public abstract class AbstractMap : FR_Map
    {
        protected Transform characterDiv;

        protected override void InitMapAction()
        {
            characterDiv = transform.Find("Div_3");
        }

        public override void DestroyMap()
        {
            Destroy(gameObject);
        }

        public override FR_Player CreatePlayer()
        {
            string path = "Prefabs/Character/PlayerCharacter";
            GameObject prefab = Resources.Load(path) as GameObject;
            GameObject obj = Instantiate(prefab, characterDiv);

            player = obj.GetComponent<PlayerCharacter>();

            return player;
        }
    }
}
```

`MainScene` 类是主场景类，继承框架的主场景同时编写一些游戏特有的方法。

```
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace Refactor.Scene
{
    public class MainScene : FR_Scene.FR_MainScene
    {
        [HideInInspector]
        public AbstractMap currentMap;
        [HideInInspector]
        public GameObject mainCamera;

        protected override void InitMainSceneAction()
        {
            LocaleManager.LoadLocaleSetting();
            mainCamera = GameObject.FindWithTag("MainCamera");
        }

        protected override void OnLoadAction()
        {
        }

        public override void LoadMap(string mapName, Action callback = null)
        {
            if (currentMap != null)
            {
                currentMap.DestroyMap();
            }

            GameObject prefab = Resources.Load("Prefabs/Map/" + mapName) as GameObject;
            GameObject obj = Instantiate(prefab);
            currentMap = obj.GetComponent<AbstractMap>();
            currentMap.mainScene = this;

            if (callback != null)
            {
                callback();
            }
        }
    }
}
```

如在 `InitMainSceneAction` 方法中，初始化本地语言配置文件以及获得当前场景的摄像机节点；`LoadMap` 是加载地图的方法，这里只进行简单的实现。

`PlayerCharacter` 就是玩家控制的角色节点的逻辑处理类：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace Refactor
{
    public class PlayerCharacter : FR_Scene.FR_Player
    {
        public float moveSpeed = 3.8f;

        private Rigidbody2D rb;
        private float horizontal;

        private Vector3 originalLocalScale;

        protected override void InitCharacterAction()
        {
            rb = GetComponent<Rigidbody2D>();
            originalLocalScale = transform.localScale;
        }

        private void Update()
        {
            horizontal = Input.GetAxisRaw("Horizontal");
        }

        private void FixedUpdate()
        {
            rb.velocity = new Vector2(horizontal * moveSpeed, 0);

            if (horizontal != 0)
            {
                float scaleX = horizontal > 0 ? originalLocalScale.x : -1 * originalLocalScale.x;
                SetLocalScale(scaleX);
            }
        }
    }
}
```

上面实现了简单的控制移动。

![角色的移动](https://pic.imgdb.cn/item/6111f4325132923bf8c3444d.gif)

现在做的事情跟原来没什么不同，还花了那么多时间来修改代码，这样做的意义何在？
其实主要还是为了规范化的开发，而且现在的演示也没法体现出框架的好处，越往后开发框架的优点才越清晰。

##### 游戏对象建造器
上面的代码中，直接调用 Unity 的实例化方法创建节点：

```
// 生成角色节点
public override FR_Player CreatePlayer()
{
    string path = "Prefabs/Character/PlayerCharacter";
    GameObject prefab = Resources.Load(path) as GameObject;
    GameObject obj = Instantiate(prefab, characterDiv);

    player = obj.GetComponent<PlayerCharacter>();

    return player;
}

// 生成地图节点
public override void LoadMap(string mapName, Action callback = null)
{
    if (currentMap != null)
    {
        currentMap.DestroyMap();
    }

    GameObject prefab = Resources.Load("Prefabs/Map/" + mapName) as GameObject;
    GameObject obj = Instantiate(prefab);
    currentMap = obj.GetComponent<AbstractMap>();
    currentMap.mainScene = this;

    if (callback != null)
    {
        callback();
    }
}
```

在创建游戏节点（GameObject）的时候发现了“冗余”的代码：

```
GameObject prefab = Resources.Load("Prefabs/Map/" + mapName) as GameObject;
GameObject obj = Instantiate(prefab);
currentMap = obj.GetComponent<AbstractMap>();
```

在加载动态资源的时候要用到 `Resources.Load`，然后又要实例化预制体要用到 `Instantiate`……这个过程非常繁琐，而且还都是重复性劳动，对于开发体验极其不友好。

框架的作用就体现出来了，只要封装一个专门用来“生成节点”的类就好了。
创建一个专门用来生成游戏对象的类 `FR_ObjectBuilder`：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace FR_Builder
{
    public class FR_ObjectBuilder : MonoBehaviour
    {
        public static GameObject Generate(string prefabPath, Transform parentTrans = null)
        {
            GameObject prefab = Resources.Load<GameObject>(prefabPath);

            return Generate(prefab, parentTrans);
        }

        public static GameObject Generate(GameObject prefab, Transform parentTrans = null)
        {
            GameObject obj = Instantiate(prefab);

            if (parentTrans != null)
            {
                obj.transform.SetParent(parentTrans);
            }

            return obj;
        }
    }
}
```

然后上面创建地图和创建角色对象的方法就可以简化为：

```
// 创建地图
public override void LoadMap(string mapName, Action callback = null)
{
    if (currentMap != null)
    {
        currentMap.DestroyMap();
    }

    GameObject obj = FR_Builder.FR_ObjectBuilder.Generate("Prefabs/Map/" + mapName);
    currentMap = obj.GetComponent<AbstractMap>();
    currentMap.mainScene = this;

    if (callback != null)
    {
        callback();
    }
}

// 创建角色节点
public override FR_Player CreatePlayer()
{
    string path = "Prefabs/Character/PlayerCharacter";
    GameObject obj = FR_Builder.FR_ObjectBuilder.Generate(path);

    player = obj.GetComponent<PlayerCharacter>();

    return player;
}
```

##### 角色节点生成器
地图中的 NPC 以及玩家控制的主角，都叫做角色节点，本质都是同一种类型的节点。
只是它们的控制逻辑不同，玩家角色需要玩家输入键盘指令来控制移动，NPC 则是由系统 AI 控制。

可以把角色节点想象为一团橡皮泥，刚开始的时候是一团不知道什么类型的泥。
如果要捏一个 NPC，就捏出 NPC 的形状；如果要捏一个主角，就捏出主角的形状。
它们最开始都是 `FR_Character` 类型的，它们都有获取精灵和动画组件的方法，再根据需要在子类编写特有的方法。

```
public abstract class FR_Character : MonoBehaviour
{
    private SpriteRenderer walkingGraphRenderer;
    private Animator animator;

    private void Awake()
    {
        walkingGraphRenderer = GetComponent<SpriteRenderer>();
        animator = GetComponent<Animator>();

        InitCharacterAction();
    }

    public void SetWalkingGraphSprite(Sprite sprite)
    {
        walkingGraphRenderer.sprite = sprite;
    }

    // 省略其他方法……
}
```

`FR_Character` 类定义了一团“橡皮泥”，它没有任何“形状”，它提供了一个 `SetWalkingGraphSprite` 方法用于实现“变形功能”（替换行走图）。

接着为 `FR_Map` 类添加生成 NPC 节点的方法：

```
// 定义一个字典用于保存场景中的NPC节点
private Dictionary<string, GameObject> npcs = new Dictionary<string, GameObject>();

public GameObject CreateNPC(FR_Data.CharacterData data)
{
    GameObject obj = new GameObject();

    obj.name = data.name;
    SpriteRenderer spriteRenderer = obj.AddComponent<SpriteRenderer>();
    spriteRenderer.sprite = Resources.Load<Sprite>(data.walkingGraphPath);
    spriteRenderer.sortingOrder = data.sorting;

    npcs.Add(data.name, obj);

    return obj;
}
```

`GameObject` 是 Unity 所有游戏对象的基类，也就是创建一团空白的橡皮泥。
要用橡皮泥捏出一个形状，可以是小动物、也可以是人，也可以是建筑物……
它们的区别就在于形状和大小、颜色，这些就是“参数”。
在创建 NPC 的时候，需要传入 NPC 的行走图、位置等等参数，这样就可以根据参数“定制”出一个 NPC：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace FR_Data
{
    public class CharacterData
    {
        public string name;
        public string walkingGraphPath;
        public int sorting;
        public Vector3 localePosition;

        // …… 还可以定义更加详细的参数，如缩放等等
    }
}
```

然后在游戏地图的方法中测试一下：

```
CreateNPC(new FR_Data.CharacterData
            {
                name = "npc_1",
                walkingGraphPath = "Sprites/WalkingGraph/xiaoxing",
                sorting = 300,
                localePosition = new Vector3(0, -1.35f),
            });
```

通过继承得到了 `CreateNPC` 方法，传入基本的参数。
运行游戏测试：

![动态创建NPC](https://pic.imgdb.cn/item/611287ba5132923bf8e87721.jpg)

可以看到左上角的节点树多了一个 `npc_1` 的节点，就是动态创建的 NPC。
虽然节点可以存在重名，但是需要人为的避免重名的情况，因为这样不利于查找节点。

现在，框架已经提供了一个可以动态创建 NPC 的方法，NPC 由脚本控制而不需要在地图中手动添加。
而且创建 NPC 的节点不依赖任何 Prefab（预制体），仅需要提供 `FR_Data.CharacterData` 参数即可。

如此一来，节省了手动创建节点的时间，而且地图中也会变得非常干净。
但是缺点就是没有运行游戏的话，就不知道 NPC 会出现在哪里，总体而言是利大于弊的。

##### 查询节点
在调用 `CreateNPC` 创建节点的时候，已经将节点的对象添加到 `Dictionary<string, GameObject> npcs` 字典里了。查找地图中的某个 NPC 只需要知道 NPC 的名字就可以直接获得。

查询节点是剧情动画中必不可少的一个功能，比如控制地图中哪个 NPC 走动，只要知道名字就能直接拿到对应的 NPC，非常方便。

#### 自动加载事件
事件的处理包括：

- 事件条件的判断
- 事件内容的处理

首先判断事件的条件是否成立，比如玩家切换到地图的时候，要自动进入一段剧情事件，然后下一次再回到这个地图，这个事件就不应该继续再执行了。也就是说，第一次切换到地图的时候，因为还没执行过事件，就执行一次，然后设置一个标志证明已经执行过事件了，下次回到这个地图，判断是否存在这个标志，如果有这个标志就不进行处理。

这个标志即事件的触发条件，比如一个开关，一个变量，都可以作为事件的条件判断依据。
判断依据是游戏中特有的逻辑，需要单独实现，框架只提供一个抽象类用于继承。

##### 事件触发器
事件触发器是通用的，不仅仅只是用来做地图自动触发事件的判断，它可以用在任何需要进行条件判断的地方。
它的作用有点类似状态机，但是实现原理与状态机不同。

触发器的作用相当于 `if` 结构，即如果……就……。对于事件的判断用 `if` 结构肯定没问题，但是游戏的剧情逻辑十分复杂，用 `if` 来处理就不实际了，通俗的讲事件触发器就是比较高级的 `if` 判断语句。

事件触发器如果用可视化的界面比喻就是 RPG Maker 系列了的“事件页”系统：

![RPG Maker 事件页](https://pic.imgdb.cn/item/610fb72c5132923bf89e9528.jpg)

当满足左上角设置的开关、独立开关以及变量的条件，事件就会显示当前页的逻辑处理。

> RPG Maker 的事件页里还可以再添加条件判断语句来执行更加细致的条件判断，如上面的“条件分歧”。

事件页模式有一个缺点就是只能单向处理事件。比如有 A、B、C 三个事件，首先触发器的逻辑会从最后一页开始判断，当满足 C 的条件时，就不再继续判断 B 和 A 的条件，而是开始执行 C 的逻辑处理。通常在处理完 C 的事件后，会再添加一个空白的 D 事件，在 C 事件处理完成之后就不再执行。

比如有一个宝箱，打开之后得到一些金钱，玩家打开宝箱后就要开启一个独立开关，增加一个空白页防止无限获得金钱。

这样就存在一个问题——无法返回前置的事件，因为 C 事件执行完之后就已经打开了 D 事件的开关，下次重新载入这个事件也只能执行 D 事件，无法回头执行 A、B 事件。

典型的例子就是当玩家同时完成两个任务，而任务的委托人是同一个 NPC，这样在提交完其中一个任务，另一个任务就无法提交了。
为了避免这种情况，我改进了这种单向的事件页模式。

添加新的命名空间 `FR_Trigger` 用来管理事件触发器类。


```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace FR_Trigger
{
    public class FR_TriggerManager
    {
        protected static List<FR_TriggerEventContainer> containers = new List<FR_TriggerEventContainer>();
        protected static bool isHandle;

        public static FR_TriggerEventContainer CreateContainer()
        {
            FR_TriggerEventContainer container = new FR_TriggerEventContainer();
            containers.Add(container);

            return container;
        }

        public static void Handle()
        {
            if (isHandle) return;

            foreach (var container in containers)
            {
                var data = container.GetSatisfiedPage();
                if (data != null)
                {
                    isHandle = true;
                    data.HandleEvent();

                    return;
                }
            }
        }
    }
}
```

改进的事件页模式增加了容器的概念，通过 `CreateContainer` 来创建一个新的事件容器，一个事件容器就是 RPG Maker 的一个事件（包含多页），容器存在执行顺序，当满足了第一个容器之后就不会再执行其他容器事件，避免多个容器的事件被同时执行。

##### 事件容器
事件容器存储了多页事件的触发条件以及对应的事件处理，相当于 RPG Maker 里的一个“事件”。

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace FR_Trigger
{
    public class FR_TriggerEventContainer
    {
        public List<FR_TriggerEventPage> pages = new List<FR_TriggerEventPage>();

        public FR_TriggerEventContainer AddTriggerEvent(FR_TriggerEventPage eventData)
        {
            pages.Add(eventData);

            return this;
        }

        public FR_TriggerEventPage GetSatisfiedPage()
        {
            for (int i = pages.Count - 1; i > 0; i--)
            {
                if (pages[i].Check())
                {
                    return pages[i];
                }
            }

            return null;
        }
    }
}
```

单页事件的数据：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace FR_Trigger
{
    public class FR_TriggerEventPage
    {
        public List<FR_TriggerConditionAbstract> conditionAbstracts;
        public FR_TriggerEvent triggerEvent;

        public bool Check()
        {
            foreach (var condition in conditionAbstracts)
            {
                if (condition.Check() == false) return false;
            }

            return true;
        }

        public void HandleEvent()
        {
            triggerEvent.Success();
        }
    }
}
```

##### 事件触发条件
触发条件是一个抽象类，子类只要继承此类，然后实现一个 `Check` 检验条件是否成立即可。

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace FR_Trigger
{
    public abstract class FR_TriggerConditionAbstract : MonoBehaviour
    {
        public abstract bool Check();
    }
}

```

##### 抽象事件类
框架只需要封装一个标准的事件类，具体的处理要在游戏实际的场景编写。

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace FR_Trigger
{
    public class FR_TriggerEvent : MonoBehaviour
    {
        protected System.Action successAction;

        public FR_TriggerEvent(System.Action successAction = null)
        {
            this.successAction = successAction;
        }

        public void Success()
        {
            if (successAction == null) return;

            successAction();
        }
    }
}
```

事件页包含了成功时的处理方法以及构造函数用于给方法赋值。

#### NPC 事件
NPC 事件与地图的自动触发事件原理一样，只是触发的时机不同。
地图自动触发事件是在地图加载时就进行判断，而 NPC 事件则是玩家走到 NPC 面前，按下调查键才会触发的。

#### 剧情演出系统
一段剧情由很多个「事件动作」组成，例如控制 NPC 移动，然后弹出对话。NPC 说完话后离开场景，接着又轮到玩家控制的角色说话……诸如此类。剧情演出系统是由一连串的动作组成，这些动作的具体实现是子类要做的事情，框架只需要提供一个可以触发剧情演出的管理器即可。

关于剧情演出系统，在上一篇文章已经实现了：[如何优雅的控制游戏中的剧情事件？](https://huotuyouxi.com/2021/08/06/game-maker-001/)

剧情演出系统由三个部分构成：

- 剧情演出管理器
- 事件容器
- 事件动作

##### 管理器

管理器负责调用事件容器：

```
using UnityEngine;
using System.Collections;
using System.Collections.Generic;

namespace FR_Event
{
    public class EventManager : MonoBehaviour
    {
        public static EventContainer eventContainer;

        public static EventContainer StartNewQueue()
        {
            eventContainer = new EventContainer();
            return eventContainer;
        }
    }
}
```

##### 事件容器

事件容器负责存储事件动作以及执行当前动作：

```
using System;
using System.Collections.Generic;
using UnityEngine;

namespace FR_Event
{
    public class EventContainer
    {
        public Queue<EventActionAbstract> queue = new Queue<EventActionAbstract>();

        public EventContainer Append(EventActionAbstract eventAction)
        {
            queue.Enqueue(eventAction);
            return this;
        }

        public void StartEvent()
        {
            if (queue.Count == 0) return;

            var current = queue.Dequeue();
            current.Handle();
        }
    }
}
```

##### 抽象事件动作

最后抽象出事件动作，提供一个父类让游戏的实际使用场景继承：

```
namespace FR_Event
{
    public abstract class EventActionAbstract
    {
        protected void EventEnd()
        {
            FR_Event.EventManager.eventContainer.StartEvent();
        }

        public abstract void Handle();
    }
}
```

##### 文件规范

上一篇文章是把事件动作的具体实现放到框架的文件夹里：

![事件动作所在目录](https://pic.imgdb.cn/item/610f99965132923bf8500b27.jpg)

但是为了与游戏逻辑解耦，需要移除这个文件夹，并放到游戏的脚本目录里：

![新建存放事件动作的文件夹](https://pic.imgdb.cn/item/610f9a015132923bf8514a7a.jpg)

框架的代码不能包含当前游戏的相关逻辑，这样以后才可以抽取出来用在不同的游戏里。

完成之后，剧情的演出系统就可以通过如下这种链式调用来组织一段剧情的演出：

```
 FR_Event.EventManager
          .StartNewQueue()
          .Append(new DialogAction("00_test/01_text"))
          .Append(new DialogAction("00_test/02_text"))
          .StartEvent();
```

## 场景系统结构图
大致的组织结构图如下：

![场景系统结构图](https://pic.imgdb.cn/item/6112904f5132923bf8fccccd.jpg)

## 结尾
目前的框架只是一个基本的雏形，而且后面的大都是理论没有实际测试。我打算把原来做好的部分逐步替换为新框架，根据实际使用的情况来完善这个框架。

PS.这里发现了自己命名也是不规范的，加上 `namespace`（命名空间）后，文件和类的名字就不用再加上 `FR_` 的前缀了，但是因为担心和游戏实际代码的类混淆，所以有时候会加上前缀，有时候又忘了加……总的来说，就算是同一个人写的代码，要保持相同的风格也是不靠谱的……