---
title: 【Unity小技巧】横版平台游戏之残影效果
date: 2021-08-21 02:23:39
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
小时候玩的《泡泡堂》里面有非常酷炫的“幻影效果”。
商城售价￥9.8，最早去网吧 5 毛钱就能玩一个小时，后来涨到一块钱。
十块钱意味着可以去网吧玩 10-20 个小时，所以这个幻影当时是狠下心才买的，效果如下：

![泡泡堂幻影](https://pic.imgdb.cn/item/611fd1064907e2d39c36b43a.jpg)

这个效果十分酷炫，如果能加入到自制的平台跳跃游戏里，就更酷了！

## 如何实现幻影效果？
幻影效果的原理其实很简单，就是在角色当前位置创建一个与角色相同的图像，然后过一段时间让其自动消失。

## 快速开始
### 预制体
幻影对象是一个 Prefab（预制体），最开始是一个空的 Sprite 对象，在创建的时候设置它的 Sprite，让它变成跟角色一模一样的图像。
直接在游戏场景创建一个 2DSprite 就行了，然后把它拖到 `Resources/Prefab` 文件夹里备用。

![预制体](https://pic.imgdb.cn/item/611fd34c4907e2d39c3dc044.jpg)

### 幻影脚本
需要给预制体加上脚本用来控制幻影的显示。
幻影脚本很简单，暴露一个 `Init` 方法用来对幻影的参数进行赋值。

新建 `ShadowSprite` 脚本：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ShadowSprite : MonoBehaviour
{
    public float time = 0.5f;

    private SpriteRenderer spriteRenderer;

    private void Awake()
    {
        spriteRenderer = GetComponent<SpriteRenderer>();
    }

    private void Start()
    {
        Destroy(gameObject, time);
    }

    public void Init(Sprite sprite, Vector3 position, Vector3 scale)
    {
        spriteRenderer.sprite = sprite;
        transform.position = position;
        transform.localScale = scale;
    }
}
```

脚本写好之后，把它挂在预制体上面。

### 生成幻影
控制角色移动的方法就省略了，只保留生成幻影相关的代码。
在 `Awake` 获得当前角色的 `SpriteRenderer`，用来获取角色的图像。
然后在 `Update` 方法里生成幻影对象，每一帧都在角色当前位置生成一个与角色相同图像的对象：

```
public class Player : MonoBehaviour
{
    private SpriteRenderer spriteRenderer;

    private void Awake()
    {
        spriteRenderer = GetComponent<SpriteRenderer>();
    }

    private void Update()
    {
        // ... 此处省略无关代码
        GameObject obj = Resources.Load<GameObject>("Prefab/ShadowSprite");
        GameObject shadow = Instantiate(obj);
        ShadowSprite shadowSprite = shadow.GetComponent<ShadowSprite>();
        shadowSprite.Init(spriteRenderer.sprite, transform.position, transform.localScale);
    }
}
```

好了，这样就可以生成残影效果了，进入游戏测试：

![残影效果](https://pic.imgdb.cn/item/611fd4a04907e2d39c40aaf6.gif)

## 特效优化
因为透明度与角色的一样，所以幻影的角色很难分清。
为了区分幻影，需要调整幻影的透明度，并且可以让它逐渐淡出消失。

修改幻影脚本：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ShadowSprite : MonoBehaviour
{
    public float fadeSpeed = 1f, initAlpha = 0.5f;
    private SpriteRenderer spriteRenderer;

    private void Awake()
    {
        spriteRenderer = GetComponent<SpriteRenderer>();
        spriteRenderer.color = new Color(1f, 1f, 1f, initAlpha);
    }

    public void Init(Sprite sprite, Vector3 position, Vector3 scale)
    {
        spriteRenderer.sprite = sprite;
        transform.position = position;
        transform.localScale = scale;
    }

    private void FixedUpdate()
    {
        float alph = spriteRenderer.color.a - Time.deltaTime * fadeSpeed;
        spriteRenderer.color = new Color(1f, 1f, 1f, alph);

        if (spriteRenderer.color.a <= 0)
        {
            Destroy(gameObject);
        }
    }
}
```

优化后的幻影脚本移除了 `time` 变量，而是改用 `fadeSpeed`（淡出速度）来控制幻影消失的时间，这个值越大，幻影淡出的速度越快。然后新增了一个 `initAlpha` 变量用来控制幻影生成时的初始透明度。

在 `Awake` 方法中，赋值初始透明度：

```
spriteRenderer.color = new Color(1f, 1f, 1f, initAlpha);
```

这样幻影在生成的时候就是半透明的了。
`FixedUpdate` 方法里对幻影的淡出效果进行处理：

```
private void FixedUpdate()
{
    // 计算淡出的透明度
    float alph = spriteRenderer.color.a - Time.deltaTime * fadeSpeed;

    // 改变当前的透明度
    spriteRenderer.color = new Color(1f, 1f, 1f, alph);

    // 当完全透明的时候销毁对象
    if (spriteRenderer.color.a <= 0)
    {
        Destroy(gameObject);
    }
}
```

进入游戏测试：

![淡出效果的幻影](https://pic.imgdb.cn/item/611fd7584907e2d39c45f99d.gif)

## 原地不生成幻影
因为是在 `Update` 方法里不断生成残影对象，当角色站着不动的时候也会源源不断的产生残影，这样看起来就很奇怪。

![站着不动也会产生幻影](https://pic.imgdb.cn/item/611fdb2a4907e2d39c4d0db7.gif)

我们希望只在移动的时候产生幻影，只需要改变角色的控制器方法：

```
private void Update()
{
    // ... 此处省略无关代码

    if ((inGround && rb.velocity.x != 0) || (!inGround && jumping))
    {
        GameObject obj = Resources.Load<GameObject>("Prefab/ShadowSprite");
        GameObject shadow = Instantiate(obj);
        ShadowSprite shadowSprite = shadow.GetComponent<ShadowSprite>();
        shadowSprite.Init(spriteRenderer.sprite, transform.position, transform.localScale);
    }
}
```

当角色站在地板并且水平方向的速度不为 0，即站在地板上走动的时候，可以产生幻影；
当角色不在地板并且处于跳跃状态时，可以产生幻影。

![原地不生成幻影](https://pic.imgdb.cn/item/611fdc724907e2d39c4f6bef.gif)

## 幻影的数量
幻影的数量不宜太多，否则观感也不好。
我们可以设定一个值用来控制幻影生成的频率。

修改角色控制脚本：

```
public class Player : MonoBehaviour
{
    // 控制生成幻影的频率
    public float shadowCoolTime = 0.1f;

    // 计算下一次生成幻影的时间
    private float shadowTime;

    private void Update()
    {
        // ... 省略无关代码

        if(Time.time >= shadowTime)
        {
            if ((inGround && rb.velocity.x != 0) || (!inGround && jumping))
            {
                shadowTime = Time.time + shadowCoolTime;
                GameObject obj = Resources.Load<GameObject>("Prefab/ShadowSprite");
                GameObject shadow = Instantiate(obj);
                ShadowSprite shadowSprite = shadow.GetComponent<ShadowSprite>();
                shadowSprite.Init(spriteRenderer.sprite, transform.position, transform.localScale);
            }
        }
    }
}
```

进入游戏测试，看起来好多了：

![幻影的数量](https://pic.imgdb.cn/item/611fde254907e2d39c52b648.gif)

## 性能的问题
幻影对象需要频繁的创建/销毁，但其实幻影对象都一模一样，频繁创建和销毁它们相当于做了许多“无用功”，创建好的幻影对象能不能不销毁，而是拿来复用呢？比如当幻影消失的时候，不使用 `Destroy` 脚本将它销毁，而是把它的 `Active` 设置为 `false`，在下次生成幻影对象的时候，直接将它激活就可以。

答案是可以，而且实现起来并不难。思路是创建一个用来保存所有 `ShadowSprite` 对象的变量，最开始的时候先生成 N 个幻影对象，并且将其全部藏起来，等到实际要用的时候再逐个拿出来。

> List、数组、链表、队列其实都可以实现。

比较推荐的变量类型是：List、队列、链表。
因为这些数据类型是 `c#` 的基本类型，提供了方便的获取和删除方法，而如果要用数组实现的话，就要自己写方法了。

### 实现思路
用一个链表结构保存幻影对象。

- 初始化创建 N 个对象，将其 `Active` 设置为 `false`，保存在链表中
- 生成幻影时，从链表取出一个，并对其进行初始化
- 幻影消失时，将其重新放回链表，并且 `Active` 设置为 `false`

### ShadowComponent
为了保持代码的简洁，将原来写在角色控制器的幻影相关代码抽取出来，新建一个 `ShadowComponent` 脚本：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ShadowComponent : MonoBehaviour
{
    // 用于存储幻影对象的链表
    private LinkedList<ShadowSprite> shadowSprites;

    private void Awake()
    {
        InitShadowSprites();
    }

    // 初始化生成最初的幻影对象
    private void InitShadowSprites()
    {
        shadowSprites = new LinkedList<ShadowSprite>();

        // 因为生成幻影的频率不高，所以设置5个就够了，多了也会浪费内存
        int count = 5;

        for (int i = 0; i < count; i++)
        {
            var shadow = CreateShadowSprite();
            shadowSprites.AddFirst(shadow);
        }
    }

    // 生成单个幻影对象的方法
    private ShadowSprite CreateShadowSprite()
    {
        GameObject prefab = Resources.Load<GameObject>("Prefab/ShadowSprite");
        GameObject obj = Instantiate(prefab);
        ShadowSprite shadow = obj.GetComponent<ShadowSprite>();
        shadow.SetShadowComponent(this);

        return shadow;
    }

    // 从链表取出幻影对象，如果链表没有可取的对象就创建一个新的对象
    public ShadowSprite GetShadow()
    {
        ShadowSprite shadow;

        if (shadowSprites.First != null)
        {
            shadow = shadowSprites.First.Value;
            shadowSprites.RemoveFirst();
        }
        else
        {
            shadow = CreateShadowSprite();
        }

        return shadow;
    }

    // 回收幻影对象，并且将其设置为隐藏状态
    public void PutShadow(ShadowSprite shadow)
    {
        shadow.gameObject.SetActive(false);
        shadowSprites.AddFirst(shadow);
    }
}
```

将这个脚本挂在角色对象上。

### 幻影脚本
然后再修改幻影脚本：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ShadowSprite : MonoBehaviour
{
    public float fadeSpeed = 1f, initAlpha = 0.5f;
    private Player player;
    private SpriteRenderer spriteRenderer;
    private ShadowComponent shadowComponent;

    private void Awake()
    {
        spriteRenderer = GetComponent<SpriteRenderer>();
    }

    private void OnEnable()
    {
        spriteRenderer.color = new Color(1f, 1f, 1f, initAlpha);
    }

    public void SetShadowComponent(ShadowComponent shadowComponent)
    {
        this.shadowComponent = shadowComponent;
    }

    public void Init(Sprite sprite, Vector3 position, Vector3 scale)
    {
        spriteRenderer.sprite = sprite;
        transform.position = position;
        transform.localScale = scale;
    }

    private void FixedUpdate()
    {
        float alph = spriteRenderer.color.a - Time.deltaTime * fadeSpeed;
        spriteRenderer.color = new Color(1f, 1f, 1f, alph);

        if (spriteRenderer.color.a <= 0)
        {
            shadowComponent.PutShadow(this);
        }
    }
}
```

`OnEnable` 是当对象从 `Active` 从 `false` 变为 `true`，或者对象第一次创建的时候就会自动调用的 untiy 内置的生命周期函数。当幻影对象被重新激活时，需要将透明度重新调整为初始值，因为幻影对象在消失的时候透明度被设置为 0，即使重新激活，透明度也不会自动初始化，需要手动修改才行。

### 获取幻影对象
最后在角色控制器类上面添加如下代码：

```
public class Player : MonoBehaviour
{
    private ShadowComponent shadowComponent;

    private void Awake()
    {
        shadowComponent = GetComponent<ShadowComponent>();
    }

    private void Update()
    {
        // ... 此处省略无关代码

        Shadow();
    }

    private void Shadow()
    {
        if (Time.time >= shadowTime)
        {
            if ((inGround && rb.velocity.x != 0) || (!inGround && jumping))
            {
                shadowTime = Time.time + shadowCoolTime;
                var shadow = shadowComponent.GetShadow();
                shadow.Init(spriteRenderer.sprite, transform.position, transform.localScale);

                shadow.gameObject.SetActive(true);
            }
        }
    }
}
```

在生成幻影对象的时候，从 `shadowComponent` 组件获取，然后调用幻影脚本的 `Init` 方法设置图像和位置、缩放。

最后的演示效果：

![幻影对象不断被回收利用](https://pic.imgdb.cn/item/611fefd24907e2d39c6dba21.gif)

可以发现，最开始生成了 5 个对象，然后不断的激活/失活（即被循环利用了）。

## 对象池
上述方法实现「循环利用」相同的对象，这种模式叫做：对象池模式。
对象池模式的特点就是预先创建好需要用到的资源，然后放进“池子”里备用，在需要用到的时候就可以直接从“池子”直接取出来，用完了再放回去，从而大幅减少创建资源需要的开销。

这种模式在很多框架里都有相关的实现，比如 MySQL、Redis 连接池，以及 PHP-FPM 等等。
但是这些对象池比较复杂，因为 MySQL 和 Redis 连接可能会超时断开，因此需要另一种叫做「心跳检测」的机制定期检查连接是否被断开，这里就不再扩展了。

## 对象池的应用场景
对象池在游戏中主要用在需要频繁创建物体，并且对性能的开销很大的地方，例如：弹幕游戏中满屏幕的子弹，魔兽 War3 中不断刷新怪物的“经验房间”、“金币房间”等等。

## 注意事项
对象池在获取对象的时候一定要注意重置对象的初始属性。
尤其是如果对象包含动画组件或者产生了变形，一定要在激活时对其初始化。
比如上面的幻影对象，幻影在消失的时候，透明度变成了 0，而将其放回池子的时候，透明度依然是 0，如果直接拿出来用，因为是完全透明的会导致看不见，导致产生了“莫名其妙的 BUG”，而且这种 BUG 很容易被遗忘，以为放进池子里再拿出来就会默认变为初始状态，实际上并不会。

> 特别特别要注意「动画效果」的初始化！