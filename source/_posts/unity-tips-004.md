---
title: 【Unity小技巧】像搭积木一样简单，组件化的开发思想！
date: 2021-08-23 22:17:41
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
昨天和今天都在研究平台小游戏的跳跃、攀墙等等基本功能。
然后意外的发现了一个值得研究的项目：[https://github.com/mixandjam/Celeste-Movement](https://github.com/mixandjam/Celeste-Movement)
这个项目是模仿《蔚蓝》实现操作手感的优化。
然而最重要的不是里面如何优化操作手感，而是这位老外的开发方式非常值得学习。

PS.组件化开发不仅可以运用在游戏上面，而且对于 WEB 开发、APP 开发等等，均有运用的场景，编程思想是通用的。

## 组件
Unity 本身就是组件化的开发，场景中的对象基本上都是由一个 `Empty GameObject`（空对象）构造而成：

![空对象](https://pic.imgdb.cn/item/6123aebd44eaada739a43af8.jpg)

创建一个新的空对象，包含最基本的属性 `Transform`：

![Transform](https://pic.imgdb.cn/item/6123aefe44eaada739a4ce73.jpg)

然后在面板上点击 `Add Component` 即可添加组件：

![Add Component](https://pic.imgdb.cn/item/6123af2644eaada739a52be4.jpg)

给它添加 `Sprite Render`（精灵渲染器），它就可以显示一张图片；
给它添加 `RigidBody 2D` 就可以让它具有 2D 刚体属性；
甚至还可以给它添加 `Camera` 就可以让它变成场景中的摄像机。

也就是说，在 unity 场景中的游戏对象都是从一个空物体再「装上」不同的组件构成的。
打个比方，宝可梦中的伊布，如果给它雷之石，它就会进化成雷伊布，给它水之石就会进化成水伊布，以此类推。

这种就是「组件化」的编程思想。

## 类的继承
通常情况下为了提高代码的复用性，我们会使用继承来实现代码的复用。
比如定义一个 `Animal`（动物）作为父类，父类包含一个 `Walk` 方法，
`Cat`、`Dog` 继承 `Animal`，因此这两个类也都拥有了 `Walk` 方法。

继承是通过抽象出共有的方法来实现代码的复用，但是当层级比较多的时候，就会使得关系十分复杂。
比如现在要新增一个鸟类，鸟类也是动物，鸟类可以飞，猫和狗不能飞；
因此需要再抽象出一个 `Bird`（鸟）和 `Mammals`（哺乳动物）。
所有的鸟类继承 `Bird`，所有的哺乳动物继承 `Mammals`。

但是新的问题又来了，有一些鸟是不会飞的，比如已经灭绝的渡渡鸟。
那么又要抽象出 `FlyBird` 和 `UnFlyBird`。

当父类比较多的时候，就需要层层继承，使得继承的关系变得复杂。

除此之外，有的哺乳动物也能飞，比如鼯鼠。
这个时候作为程序员简直要崩溃了……又得把哺乳动物分成会飞的跟不会飞的。

为了解决这个问题，下面的组件化开发就诞生了。

## 组件化
动物的基本行为实际上是类里的一个方法，比如 `Walk`（行走），`Fly`（飞行）。
这些可以视为它们的“能力”，即猫有行走的能力，鸟可以在天上飞。

那么我们可以把这些能力提取出来，就叫做“能力”。
然后动物是一个本体，一个没有任何能力的基本模型，所有的能力都需要后天添加上去。

比如创建一个动物模型，然后把「飞」的能力添加到这个模型上，让它具备飞的能力。
接着再把「潜水」的能力附加到这个模型上，它就可以潜水了，这样即会飞行又会潜水的动物，就是「海雀」。
接着再创建一个新的动物模型，把「胎生」和「飞行」添加到模型上，这样这只动物既是哺乳动物又可以飞，就是「鼯鼠」。

通过这种方法，我们可以把各种动物的能力组合在一起，从而实现现实中存在的所有动物。
不仅如此，连现实中不存在的动物都可以实现，例如「哺乳动物」和「蛋生」的能力附加到一个模型上面，然而现实中所有的哺乳动物都是胎生的。

组件化开发的原理是基于一个基本的「模型」，为其添加不同的组件使其具备不同的功能。
组件化开发是弱关联关系的，比如下面这样给 unity 中的摄像机加上刚体组件：

![给摄像机加上刚体组件](https://pic.imgdb.cn/item/6123b4ce44eaada739b2871f.jpg)

这样做有什么意义吗？让摄像机符合物理学运动规律？很明显，没有什么特别的意义。
但是，它确实是可以这样做的，因为组件化的开发不存在逻辑上必然的关联。
你可以给精灵加上刚体，也可以给 `Text` 加上刚体……你可以给 unity 中的任何对象都加上刚体。

从一个基础的模型 `Empty GameObejct` 开始，给它加上不同的组件，使其具备特殊的功能。
这样做的优点是可以非常灵活的创建出不同的物体，而不需要受到逻辑上的约束。

继承关系是无法做到这点的，因为继承的原则就是 **将一系列相同的方法抽象出来**，通过继承得到的物体是从一个模子里刻出来全部一样的东西。而组件化的开发是为了解决「差异化复用」的问题，通过后期添加不同的组件使其成为不同的物体。

继承关系必须在子类实现差异化的方法，而组件化的开发则是把差异化也变成可以复用的东西，下面通过 unity 的例子来证明这一点。

## 开发需求
游戏中存在角色和敌人，它们都具有共同点：

- 有生命值，受伤会死亡
- 可以跳跃和行走

就以上述特征来比较继承式开发和组件化开发的不同。

## 传统的继承式开发
将共同的特征抽取出来，创建一个 `Character`（角色类），然后让 `Player`（玩家角色）和 `Enemy`（敌人角色）继承 `Character`，再在子类中实现差异化的方法，思路很简单，下面开始实现。

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public abstract class Character : MonoBehaviour
{
    public int maxHP;

    private int currentHP;

    private void Awake()
    {
        currentHP = maxHP;
    }

    public void TakeDamage(int damage)
    {
        currentHP -= damage;
        if (currentHP <= 0)
        {
            Dead();
        }
    }

    protected abstract void Dead();
    protected abstract void Move();
    protected abstract void Jump();
}
```

所有的角色都会受伤，因此 `TakeDamage` 是一个通用的方法，在父类进行实现；
但是角色死亡会 GameOver，而敌人死亡会爆金币，这两个并不通用，因此设为抽象方法，在子类进行实现。
同理，玩家的行走和跳跃是要按键控制的，而敌人的行走和跳跃却是通过 AI 来控制的，因此也要在子类进行实现。

接着创建一个 `PlayerCharacter` 类：

```
using UnityEngine;
using System.Collections;

public class PlayerCharacter : Character
{
    protected override void Dead()
    {
    }

    protected override void Jump()
    {
    }

    protected override void Move()
    {
    }
}
```

接着再创建一个 `EnemyCharacter` 类：

```
using UnityEngine;
using System.Collections;

public class EnemyCharacter : Character
{
    protected override void Dead()
    {
    }

    protected override void Jump()
    {
    }

    protected override void Move()
    {
    }
}
```

具体的实现这边就不写了，仅做示例。
这样就是一个典型的继承关系，从结果来看并没有什么问题。

但是，这样代码的可复用性太低，子类只能调用父类有的方法，父类没有的方法就得自己写。

## 组件化开发
如果是组件化的开发，那么可以把上面的需求拆分成：

- 生命体特征组件，有生命值以及受伤死亡相关的方法
- 行为特征组件，有行走跳跃等动作相关的方法

我们先来创建两个特征组件：

生命体特征组件 `LifeComponent`:

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class LifeComponent : MonoBehaviour
{
    public int maxHP;

    private int currentHP;

    private void Awake()
    {
        currentHP = maxHP;
    }

    public void TakeDamage(int damage)
    {
        currentHP -= damage;
        if (currentHP <= 0)
        {
            Dead();
        }
    }
}
```

接着角色类不需要继承任何自定义的类，只需要继承 unity 基础的 `MonoBehaviour` 即可。

```
using UnityEngine;
using System.Collections;

public class PlayerCharacter : MonoBehaviour
{
    private LifeComponent lifeComponent;
    private ActionComponent actionComponent;

    private void Awake()
    {
        lifeComponent = GetComponent<LifeComponent>();
        actionComponent = GetComponent<ActionComponent>();
    }

    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.LeftArrow) || Input.GetKeyDown(KeyCode.RightArrow))
        {
            actionComponent.Move();
        }
        else if (Input.GetKeyDown(KeyCode.Space))
        {
            actionComponent.Jump();
        }
    }
}
```

`PlayerCharacter` 本身并不需要实现方法，只需要获取身上的组件，然后通过组件来调用对应的方法即可。
敌人类同理，这边就不赘述了。

最终，只需要在 unity 创建一个空的游戏对象，然后把这些脚本逐一挂在上面：

![添加组件](https://pic.imgdb.cn/item/6123bc7d44eaada739c3c288.jpg)

如此一来，这个空物体就被赋予了“生命体特征”、“动作特征”和“玩家特征”。
它现在就是一个可以被玩家控制的角色了！

## 实际的运用
上面的例子因为没有实际在游戏中调试，所以可能难以展示出组件化开发的优点。
那么可以参考下面这几个例子。

### 跳跃优化组件

上一篇关于手感优化的：[https://huotuyouxi.com/2021/08/21/unity-tips-003/](https://huotuyouxi.com/2021/08/21/unity-tips-003/)
文章里有一个可以优化跳跃手感的组件，其实是一个普通的 `MonoBehaviour` 类文件，可以挂在任何物体上面：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class BetterJumping : MonoBehaviour
{
    private Rigidbody2D rb;
    public float fallMultiplier = 2.5f;
    public float lowJumpMultiplier = 2f;

    void Start()
    {
        rb = GetComponent<Rigidbody2D>();
    }

    void Update()
    {
        if(rb.velocity.y < 0)
        {
            rb.velocity += Vector2.up * Physics2D.gravity.y * (fallMultiplier - 1) * Time.deltaTime;
        }else if(rb.velocity.y > 0 && !Input.GetButton("Jump"))
        {
            rb.velocity += Vector2.up * Physics2D.gravity.y * (lowJumpMultiplier - 1) * Time.deltaTime;
        }
    }
}
```

神奇的地方在于，它不依赖任何其他类，只要挂在一个拥有刚体属性并且有跳跃功能的角色上就可以自动优化跳跃的手感。

### 幻影组件

接着是再上一篇文章：[https://huotuyouxi.com/2021/08/21/unity-tips-002/](https://huotuyouxi.com/2021/08/21/unity-tips-002/)

这是一个幻影组件 `ShadowComponent`：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ShadowComponent : MonoBehaviour
{
    private LinkedList<ShadowSprite> shadowSprites;

    private void Awake()
    {
        InitShadowSprites();
    }

    private void InitShadowSprites()
    {
        shadowSprites = new LinkedList<ShadowSprite>();

        int count = 5;

        for (int i = 0; i < count; i++)
        {
            var shadow = CreateShadowSprite();
            shadowSprites.AddFirst(shadow);
        }
    }

    private ShadowSprite CreateShadowSprite()
    {
        GameObject prefab = Resources.Load<GameObject>("Prefab/ShadowSprite");
        GameObject obj = Instantiate(prefab);
        ShadowSprite shadow = obj.GetComponent<ShadowSprite>();
        shadow.SetShadowComponent(this);

        return shadow;
    }

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

    public void PutShadow(ShadowSprite shadow)
    {
        shadow.gameObject.SetActive(false);
        shadowSprites.AddFirst(shadow);
    }
}
```

只要挂上这个幻影组件，角色就可以生成幻影特效，只需要调用 `GetShadow` 取得幻影即可。

### 碰撞监听组件
这是老外写的一个碰撞监听检测组件，它可以检测角色是否与地板、墙壁接触：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Collision : MonoBehaviour
{

    [Header("Layers")]
    public LayerMask groundLayer;

    [Space]

    public bool onGround;
    public bool onWall;
    public bool onRightWall;
    public bool onLeftWall;
    public int wallSide;

    [Space]

    [Header("Collision")]

    public float collisionRadius = 0.25f;
    public Vector2 bottomOffset, rightOffset, leftOffset;
    private Color debugCollisionColor = Color.red;

    // Start is called before the first frame update
    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {  
        onGround = Physics2D.OverlapCircle((Vector2)transform.position + bottomOffset, collisionRadius, groundLayer);
        onWall = Physics2D.OverlapCircle((Vector2)transform.position + rightOffset, collisionRadius, groundLayer) 
            || Physics2D.OverlapCircle((Vector2)transform.position + leftOffset, collisionRadius, groundLayer);

        onRightWall = Physics2D.OverlapCircle((Vector2)transform.position + rightOffset, collisionRadius, groundLayer);
        onLeftWall = Physics2D.OverlapCircle((Vector2)transform.position + leftOffset, collisionRadius, groundLayer);

        wallSide = onRightWall ? -1 : 1;
    }

    void OnDrawGizmos()
    {
        Gizmos.color = Color.red;

        var positions = new Vector2[] { bottomOffset, rightOffset, leftOffset };

        Gizmos.DrawWireSphere((Vector2)transform.position  + bottomOffset, collisionRadius);
        Gizmos.DrawWireSphere((Vector2)transform.position + rightOffset, collisionRadius);
        Gizmos.DrawWireSphere((Vector2)transform.position + leftOffset, collisionRadius);
    }
}
```

只要挂上脚本，简单的填写监听点的位置参数，就可以自动检测是否与墙体、地板发生碰撞，是不是很方便？而且还自带绘图功能，可以直观的看出碰撞监听的范围以便调试。

### 动画自动销毁组件
这是我自己写的一个能让动画播放完毕之后就销毁的组件 `OnceAnimate`：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class OnceAnimate : MonoBehaviour
{
    private Animator animator;
    private AnimatorStateInfo animatorStateInfo;

    private void Awake()
    {
        animator = GetComponent<Animator>();
    }

    private void Update()
    {
        animatorStateInfo = animator.GetCurrentAnimatorStateInfo(0);

        if (animatorStateInfo.normalizedTime >= 1)
        {
            Destroy(gameObject);
        }
    }
}
```

例如子弹击中墙壁，生成了一个爆炸动画，爆炸动画是一个预制体，在动画播放完毕之后就要自动消除，只需要挂上这个组件就行了！连代码都不用写！

### 更多的组件
同样请参考老外写的这个模仿《蔚蓝的项目》：[https://github.com/mixandjam/Celeste-Movement/tree/master/Assets/Scripts](https://github.com/mixandjam/Celeste-Movement/tree/master/Assets/Scripts)

这里的脚本都是组件，你甚至可以直接复制粘贴到你的项目中使用——这就是组件化开发的最大魅力。
它们不依赖任何其他类，不管任何地方都可以使用。

> 编程的最高境界即是“无码”，不需要写代码就能实现所需的功能。——《兔子语录》

组件化开发的复用性很强，强到可以跨项目使用，因此游戏的开发不一定要依赖框架，可以完全依赖这些组件直接从零快速搭建游戏的雏形。

## 继承式开发与组件化开发的比较
### 逻辑的不同
继承式开发适用于关联性比较强的「面向对象」式开发，而组件化开发则偏向于面向过程式开发。
继承式因为是面向对象的，因此注重“关联关系”，整体的结构井井有条。

如同俗语说的“龙生龙凤生凤”，虽然子类会有一些差别，但大体都是相同的。
组件化的开发却是“合成兽”，比如奇美拉那样的，把不同的怪物的一部分组合到一起，形成新的怪物。

### 适用场景
如果类的复杂程度不高，就可以用继承式，一般的 WEB 开发不会用到那么复杂的类，所以用继承式的就足够了。
主要看拆分出组件是否有必要，如果这种差异化能够反复使用，那么也可以考虑组件化。

不过一般的 WEB 框架都会使用“依赖注入”的方式，其功能与组件化类似，一般用依赖注入即可满足需求。
对于游戏开发来说，大部分的差异化功能是可以反复利用的，所以组件化的开发可以大大节省时间。

不管是继承还是组件，都是基于“复用代码”的角度来考虑的，根据实际场景，哪一种可以减少开发者写代码的量就选用哪种。

### 性能的差别
上面的例子中，如果不用组件化，而是把所有的代码挤在一个类，就可以实现「变量的复用」从而节省内存。但是拆分成组件之后，需要大量用到 `GetComponent` 方法来获取组件，这在 unity 是比较耗费性能的一件事。

如果是采用组件化开发并且场景中有大量的物体，那么应该考虑结合对象池来降低这种无端的性能消耗。

### 代码简洁之道
组件化的开发将每一个功能拆分成最小模块，因此代码极度简洁。
而继承式开发，一旦项目变大，后期上千行的代码都有可能，维护难度随项目变大而增加。

## 总结
组件化开发如同搭积木一样将不同的功能组装在一起，从而创造出新的东西。
只需要写好组件，后续不需要编程，只要动动手指把脚本拖到物体上面——这是多么酷的事！

接下来我也要开始专注于组件式的开发了！