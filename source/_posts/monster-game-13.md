---
title: 《名为怪物的游戏》框架开发篇（素材的处理）
date: 2021-08-13 10:00:40
tags:
 - unity
 - 开发技术

categories:
 - 新游开发
 - 名为怪物的游戏
---
## 素材说明
素材即游戏中需要的图片、音乐等等。
图片素材主要是行走图的处理，本作中使用「序列帧」动画制作行走图。

## 素材规格
图片在游戏中叫做 Sprite（精灵），一般都是单张图片，也可以是多张图片合成在一起，合成在一起的图片叫做 Sheet（精灵表/图集）。

RPG Maker 行走图教程：[行走图教程](https://jobyu.gitbooks.io/rpgmakerpixeltutorials/content/di-shi-qi-zhang-ff1a-xing-zou-tu.html)

RPG Maker 系列的行走图示例：

![RPG Maker行走图](https://pic.imgdb.cn/item/611339af5132923bf8c860cd.jpg)

一般都是用专门的打包工具合成这样的图集。
图集打包工具：[TexturePacker 官方网站](https://www.codeandweb.com/texturepacker)

## 图集
通常的游戏开发引擎都可以读取图集。
使用打包工具合成一张图集之后还会得到一个文本文件：

![图集文本](https://pic.imgdb.cn/item/61133ad15132923bf8c9ff99.jpg)

文本记录了一个 JSON 格式的字符串，用来保存偏移位置等等图片的信息。
游戏开发引擎可以读取这个文本从而实现图片的切割。

### 图集的优点
游戏引擎有个 `DrawCall`（绘图次数），Draw Call 就是 CPU 调用图形编程接口，比如 DirectX 或 OpenGL 来命令 GPU 进行渲染的操作。

简单地说这个值越低越好，数值越高表示要画图的次数越多，这样游戏的性能就会下降。
使用图集可以降低 `DrawCall`（具体的原理还未深究）。
但是一次性加载一张图片和每次画图都要加载单张图片相比，效率肯定高得多。

除了提高画图的性能，还有一个优点就是方便开发者调用。
如果是单张图片开发者不使用脚本读取的话，就要一张张拖到组件里；但如果使用图集打包之后，只要拖一次。如果使用脚本动态读取，图集只要加载一次，而单张图片却要加载 N 次，不管是性能还是操作上，图集都要优于单张图片。

### 图集的缺点
因为图片打包在一起，所以修改其中一张就得重新打包。
而且如果图集里包含了不需要用到的图片就白白加载了多余的资源。

## 格式标准
为了避免改图必须重新全部打包的问题，我们决定图集只包含单个动作。

攻击动作：
![攻击动作](https://pic.imgdb.cn/item/61133e2e5132923bf8cf7718.jpg)

倒地动作：
![倒地动作](https://pic.imgdb.cn/item/61133e3e5132923bf8cf935c.jpg)

像这样每一个动作都是一张图集，如果要改动其中一个动作就不必全部重新打包了。

## 图集切割
使用 TexturePacker 将行走图打包成单张图片：

![行走图](https://pic.imgdb.cn/item/611385e95132923bf8561a85.jpg)

可以使用 Unity 内置的分割程序将图片切分成 4 等分。

![Unity切分图集](https://pic.imgdb.cn/item/611386315132923bf856bcd3.jpg)

## 图集插件

参考教程：[Unity3D-图集制作插件TexturePacker中文教程
](https://blog.csdn.net/ChinarCSDN/article/details/85059102)
（作者：Chinarcsdn）

Unity 商店提供了一款可以读取 TexturePacker 切割数据的插件：[TexturePacker Import](https://assetstore.unity.com/packages/tools/sprite-management/texturepacker-importer-16641)

![TexturePacker Import](https://pic.imgdb.cn/item/6115d6625132923bf84bbfe2.jpg)

从商店把资源添加到账户里，接着导入到游戏项目中：

![插件包管理](https://pic.imgdb.cn/item/6115d6bb5132923bf84c5770.jpg)

导入插件的时候，插件内的 Example（范例）是没什么用的，可以不勾上。

![移除不必要的文件](https://pic.imgdb.cn/item/6115d7555132923bf84d56fe.jpg)

第一次导入的时候，插件会查找本地的文件，如果有图集就会自动加载，这个过程比较漫长（如果本地图片多的话）。
趁这个时间，打开 TexturePacker，然后重新合成一份 unity 支持的图集类型：

![unity 支持的图集类型](https://pic.imgdb.cn/item/6115d8475132923bf84ef0f7.jpg)

打包完成后，有一个合并的图像文件和一个 `.tpsheet` 后缀的文件：

![tpsheet](https://pic.imgdb.cn/item/6115d8985132923bf84f7ed4.jpg)

这个文件是图集的配置，可以用文本文档打开：

![图集配置](https://pic.imgdb.cn/item/6115da755132923bf852a7f8.jpg)

将这两个文件一起拷贝粘贴到 Unity 工程里：

![粘贴到工程里](https://pic.imgdb.cn/item/6115d8d85132923bf84fe4cf.jpg)

可以看到，行走图已经自动被分割好了！这样就不用手动切图啦！

## 插件规范
在导入插件后发现目录出现了一个文件夹：

![插件的文件夹](https://pic.imgdb.cn/item/6115d94a5132923bf850a573.jpg)

直接放在根目录明显不美观而且也不规范。
创建一个 `Plugins` 文件夹用来存放插件：

![Plugins](https://pic.imgdb.cn/item/6115d98c5132923bf851194b.jpg)

虽然插件放在哪 Unity 都会自动加载，但是为了统一管理，添加一个用来保存插件的文件夹是最好的。

## 动画制作
行走图切割完成之后，如果使用 Unity 自带的动画程序就比较麻烦了。
每一个动画都要添加状态机，并且还要给每一个角色节点添加动画组件，总体的工程量非常大。

所以这里要在框架里面实现一个动画系统。

### 加载行走图
行走图切割完后放在 Resources 文件夹备用。
新建框架类 `Spriter`，添加一个读取切割好的图片的方法：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace FR
{
    public class Spriter : MonoBehaviour
    {
        public static Sprite[] LoadSprites(string path)
        {
            Sprite[] sprites = Resources.LoadAll<Sprite>(path);

            return sprites;
        }
    }
}
```

> 这里我重新修改了所有框架类的命名空间，全部只套一层 FR，不再区分更细的目录。

### 行走图动画
行走图的动画可以用 Unity 的动画系统实现，也可以自己手动实现，每个游戏都不一样，因此不应该放在框架中处理，而是要放到游戏本身的逻辑当中。

添加一个地图角色父类 `AbstractCharacter`，该方法继承框架的 `Character`，并且提供了载入行走图以及显示动画的功能。

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using FR;

namespace Refactor
{
    public abstract class AbstractCharacter : Character
    {
        protected bool isWalking;
        protected Dictionary<string, Sprite[]> animateSprites;

        protected int animateIndex;
        protected float walkingInterval = 0.15f;
        protected float spriteAnimateTime;

        protected void LoadAnimateSprites(string characterName)
        {
            animateSprites = new Dictionary<string, Sprite[]>();

            string[] fields = new string[] { "walk", "idle" };
            string basePath = "Sprites/Character/" + characterName + "/";

            foreach (var field in fields)
            {
                var items = Spriter.LoadSprites(basePath + field);

                if (items.Length != 0)
                {
                    animateSprites.Add(field, items);
                }
            }
        }

        protected virtual void AnimateMonitor()
        {
            WalkingAnimate();
            IdleAnimate();
        }

        private void SetCurrentAnimateSprite(string key)
        {
            if (animateSprites.ContainsKey(key) == false)
            {
                Debug.Log("没有找到对应的动画文件：" + key);
                return;
            }

            spriteAnimateTime = Time.time + walkingInterval;

            animateIndex++;

            if (animateIndex >= animateSprites[key].Length)
            {
                animateIndex = 0;
            }

            var sprite = animateSprites[key][animateIndex];
            SetSprite(sprite);
        }

        protected virtual void IdleAnimate()
        {
            if (isWalking == false && Time.time > spriteAnimateTime)
            {
                SetCurrentAnimateSprite("idle");
            }
        }

        protected virtual void WalkingAnimate()
        {
            if (isWalking && Time.time > spriteAnimateTime)
            {
                SetCurrentAnimateSprite("walk");
            }
        }

        protected virtual void InputMonitor() { }
        protected virtual void InputHandle() { }

        protected void Update()
        {
            InputMonitor();
        }

        protected void FixedUpdate()
        {
            WalkingAnimate();
            IdleAnimate();
            InputHandle();
        }
    }
}
```

上面的脚本会自动加载角色的图集文件：`walk`（行走） 和 `idle`（待机）。
需要在 Resources 添加好对应的图片，以后所有的角色都遵循这个标准，即包含一个行走图和待机图。

![图片放置](https://pic.imgdb.cn/item/6115db475132923bf8543d20.jpg)

`InputMonitor`（输入监听器） 和 `InputHandle`（输入处理器）可以在子类重写。

玩家控制的角色类 `PlayerCharacter`：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using FR;

namespace Refactor
{
    public class PlayerCharacter : AbstractCharacter
    {
        public float moveSpeed = 3.8f;

        private Rigidbody2D rb;
        private float horizontal;

        private Vector3 originalLocalScale;

        protected override void InitCharacterAction()
        {
            LoadAnimateSprites("ace");

            rb = GetComponent<Rigidbody2D>();
            originalLocalScale = transform.localScale;
        }

        protected override void InputMonitor()
        {
            horizontal = Input.GetAxisRaw("Horizontal");
        }

        protected override void InputHandle()
        {
            MoveHandle();
        }

        protected void MoveHandle()
        {
            rb.velocity = new Vector2(horizontal * moveSpeed, 0);

            if (horizontal != 0)
            {
                isWalking = true;
                float scaleX = horizontal > 0 ? -1 * originalLocalScale.x : originalLocalScale.x;
                SetLocalScale(scaleX);
            }
            else
            {
                if (isWalking == true)
                {
                    animateIndex = 0;
                }

                isWalking = false;
            }
        }

    }
}
```

NPC 的角色类 `NonePlayerCharacter`：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

using FR;

namespace Refactor
{
    public class NonePlayerCharacter : AbstractCharacter
    {
        protected override void InitCharacterAction()
        {
        }
    }
}

```

目前还未开始实现 NPC 角色，因此留空备用。

![行走测试](https://pic.imgdb.cn/item/6115d0f85132923bf8431441.gif)

## 角色类的新设计
与前一篇相比，角色基类的改动比较大。
最大的不同是节点的创建，在前一篇中是这样的：

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

上述脚本通过代码动态创建一个 NPC 节点。

但是我突然想到，NPC 节点有体型的差异，比如场景中如果出现一只 BOSS，那么它的体型可能比玩家大好几倍，这样的话碰撞器的范围就难以用脚本的方法控制了。因为素材是存在多余的透明区域的，而脚本只能读取到图片的大小，要计算出不透明的宽高就比较麻烦了；所以我决定还是创建一个角色预制体，角色的预制体不需要添加逻辑脚本，而只是一个没有“芯片”的机器人，每一个 NPC 的逻辑都不同，只要给它们植入“控制芯片”就可以了。

角色预制体相当于是角色的模型，只差给它们“注入灵魂”。