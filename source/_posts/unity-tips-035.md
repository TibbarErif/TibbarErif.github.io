---
title: 【Unity小技巧】使用有限状态机实现怪物AI
date: 2022-10-28 10:48:53
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
想要在游戏中实现敌人 AI，使用简单的 `if-else` 结构，虽然能实现功能，但是最终代码会乱成一团，有限状态机可以很好的解决敌人 AI 问题，并且使代码逻辑清晰、简洁。
## 有限状态机
先来看看维基百科的说明：
> 有限状态机（英语：finite-state machine，缩写：FSM）又称有限状态自动机（英语：finite-state automaton，缩写：FSA），简称状态机，是表示有限个状态以及在这些状态之间的转移和动作等行为的数学计算模型。

简单地说，状态机就是一种用于切换状态的机制，而有限则表示所有的状态数量是可以确定的。
例如水有三种状态，常温下是液态，高温下会蒸发为气态，低温下又会凝结成固态。
气温的变化就是状态转换的【条件】，而不同形态是【结果】。
## 敌人行为
以横版卷轴游戏为例，玩家前面有一只哥布林，哥布林会在前面来回巡逻，而玩家如果靠得太近，哥布林就会追上来打玩家，而如果玩家快速逃跑，哥布林又会回到巡逻点继续巡逻。

![敌人的巡逻状态](https://pic1.imgdb.cn/item/635b4f2c16f2c2beb1a0264f.gif)
![敌人的追击状态](https://pic1.imgdb.cn/item/635b4f2c16f2c2beb1a02659.gif)

这里的状态可以简化为三种：1、巡逻，2、追击，3、攻击。

而状态的转换条件分别是：
1、当玩家距离太远时，哥布林没发现玩家就会在附近巡逻
2、当玩家距离哥布林很近时，哥布林会发现玩家然后前来追击
3、当哥布林追上玩家时就会用它手里的棍子敲打玩家

从上面的例子可以知道【距离远近】就是状态转换的条件。

接下来我们就可以在哥布林的 `Update` 方法内写下面的代码：

```
# 用于判断哥布林当前是否处于攻击状态
bool isAttack = false;

if 玩家与哥布林距离 ≤ 10 && isAttack == false
    向玩家移动
else
    返回巡逻点
endif

if 玩家与哥布林距离 ≤ 1
    isAttack=true
    使用棍子攻击玩家
endif

if 哥布林攻击动画结束
   isAttack=false
endif
```

业务代码的逻辑大致就是这样，用很多个 `if-else` 和布尔型变量来控制敌人的 AI，这样虽然可以实现功能，但维护性和复用性太差了。
## 状态机
只需要写一个用于处理不同状态的【状态机】即可解决上述问题，状态机的代码十分简单，既然是【状态】，那我们就先定义一个状态接口：

```
namespace FireRabbit
{
    public interface IState
    {
        void OnEnter();
        void OnExit();
        void OnUpdate();
    }
}
```

每个状态都有 `OnEnter`（进入状态时的初始化操作），`OnExit`（状态被转换时的回调处理），`OnUpdate`（状态的实际处理方法），接下来我们再写一个用来处理状态转换的功能类——状态机：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace FireRabbit
{
    public class FSM
    {
        Dictionary<object, IState> states = new Dictionary<object, IState>();
        IState currentState;

        public void Add(object key, IState state)
        {
            states.Add(key, state);
        }

        public void Change(object key)
        {
            Debug.Log("切换状态：" + key.ToString());

            currentState?.OnExit();
            currentState = states[key];
            currentState.OnEnter();
        }

        public void Working()
        {
            currentState?.OnUpdate();
        }
    }
}
```

好了，状态机就是这么简单，上面的代码可以直接复制到项目中使用，只需要：

```
# 引入命名空间
using FireRabbit;

FSM fsm = new();
```

还需要定义几个状态类：

```
# 敌人待机状态
public class SkirmisherIdleState : IState
{
    FSM fsm;

    public SkirmisherIdleState(FSM fsm)
    {
        this.fsm = fsm;
    }

    public void OnEnter()
    {
       
    }

    public void OnExit()
    {
    }

    public void OnUpdate()
    {
       
    }
}

# 敌人攻击状态
public class SkirmisherAttackState : IState
{
    FSM fsm;

    public SkirmisherAttackState(FSM fsm)
    {
        this.fsm = fsm;
    }

    public void OnEnter()
    {
       
    }

    public void OnExit()
    {
    }

    public void OnUpdate()
    {
       
    }
}
```
状态类实现 `IState` 接口，在构造函数内传入状态机以便用于切换状态。
定义好你需要的不同状态之后，就可以将这些状态加入状态机：

```
using FireRabbit;

FSM fsm = new();
fsm.Add("idle", new SkirmisherIdleState(fsm));
fsm.Add("attack", new SkirmisherAttackState(fsm));
```

可以用一个字符串作为状态的名称，为了规范，可以使用枚举变量（enum）：

```
enum STATE { IDLE, ATTACK };
```

这样我们就可以：

```
using FireRabbit;

FSM fsm = new();
fsm.Add(State.IDLE, new SkirmisherIdleState(fsm));
fsm.Add(State.ATTACK, new SkirmisherAttackState(fsm));
```

接着要执行状态机的方法，让状态机在敌人脚本的 `Update` 方法：

```
using FireRabbit;

public abstract class Enemy_Skirmisher : Enemy
{
    enum STATE { IDLE, CHASE, ATTACK, PATROL, HURT, DEAD };

    /// <summary>
    /// 巡逻点
    /// </summary>
    public List<Transform> patroals;

    [HideInInspector]
    public List<Vector3> patroalPoints;

    void Start() 
    {
        FSM fsm = new();
        fsm.Add(State.IDLE, new SkirmisherIdleState(fsm));
        fsm.Add(State.ATTACK, new SkirmisherAttackState(fsm));

        // 将巡逻点转化成坐标
        patroalPoints = new List<Vector3>();
        foreach (var item in patroals)
        {
            patroalPoints.Add(item.position);
        }

        // 敌人的初始状态
        fsm.Change(STATE.IDLE);
    }

    void Update() 
    {
        fsm.Working();
    }
}
```

这样就设置好敌人的 AI 了，代码非常简单！
敌人的巡逻点可以用几个空对象作为标记，方便在场景直观看到：

![敌人巡逻点和“眼睛范围”](https://i.niupic.com/images/2022/10/28/a9US.jpg)

敌人的“眼睛”其实就是一个碰撞盒子（触发器），用来检测玩家是否在触发器内：

![检测玩家的触发器](https://i.niupic.com/images/2022/10/28/a9UT.jpg)

在写一个简单的类用来判断是否“看见玩家”：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemyComponent_Finder : MonoBehaviour
{
    [HideInInspector]
    public bool isFind;

    private void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.CompareTag(Constant.TAG_PLAYER))
        {
            isFind = true;
        }
    }

    private void OnTriggerExit2D(Collider2D collision)
    {
        if (collision.CompareTag(Constant.TAG_PLAYER))
        {
            isFind = false;
        }
    }
}
```

这样当玩家进入触发器内，就能判断敌人“看见”玩家了。
敌人的 AI 只需要在状态类中具体实现即可：

```
# 待机类，敌人会在原地待机1秒钟，然后进入巡逻状态
public class SkirmisherIdleState : IState
{
    FSM fsm;
    Enemy_Skirmisher enemy;

    float timer;

    public SkirmisherIdleState(FSM fsm, Enemy_Skirmisher enemy)
    {
        this.fsm = fsm;
        this.enemy = enemy;
    }

    public void OnEnter()
    {
        timer = Time.time + 1f;
        enemy.animator.Play("idle");
    }

    public void OnExit()
    {
    }

    public void OnUpdate()
    {
        if (Time.time >= timer)
        {
            fsm.Change(STATE.PATROL);
        }
    }
}
```

用于处理敌人巡逻的逻辑，当敌人发现玩家时就会转换成追击状态。

```
public class SkirmisherPatrolState : IState
{
    FSM fsm;
    Enemy_Skirmisher enemy;
    int index;

    public SkirmisherPatrolState(FSM fsm, Enemy_Skirmisher enemy)
    {
        this.fsm = fsm;
        this.enemy = enemy;
    }

    public void OnEnter()
    {
        enemy.animator.Play("run");
    }

    public void OnExit()
    {
        Debug.Log("结束" + index);

        index++;
        if (index == enemy.patroalPoints.Count)
        {
            index = 0;
        }
    }

    public void OnUpdate()
    {
        var target = enemy.patroalPoints[index];
        enemy.SetFaceTo(target);

        var pos = new Vector3(target.x, enemy.transform.position.y);
        enemy.transform.position = Vector2.MoveTowards(enemy.transform.position, pos, enemy.moveSpeed * Time.deltaTime);

        if (enemy.finder.isFind)
        {
            fsm.Change(STATE.CHASE);
        }
        else
        {
            if (Vector3.Distance(enemy.transform.position, target) <= 1f)
            {
                fsm.Change(STATE.IDLE);
            }
        }
    }
}
```

这里列举的仅用于演示，就不逐一把所有状态都贴上来了。

除了受伤状态之外，其他状态都是在对应状态类内实现转化，而受伤类比较特殊，必须在敌人被玩家攻击的时候才会判定，因此需要在敌人的受伤方法内调用状态机进行状态转换，敌人的父类方法：

```
/// <summary>
/// 敌人受到伤害（死亡在状态机进行判定）
/// </summary>
/// <param name="damage"></param>
public override void TakeDamage(int damage)
{
    if (invincible || isDead) return;

    ShowDamageText(damage);
    currentHP = Mathf.Max(0, currentHP - damage);

    OnHurtCallback(damage);
}
```

敌人子类实现回调方法：

```
protected override void OnHurtCallback(int damage)
{
    Debug.Log("剩余HP：" + currentHP);
    fsm.Change(STATE.HURT);
}
```

最后测试敌人死亡：

![敌人死亡状态](https://pic1.imgdb.cn/item/635b4f2c16f2c2beb1a02663.gif)
## 游戏日记
国庆期间用了一周多的时间写了横版卷轴新作的剧本，结果发现剧情超过预期的长度于是就废弃了（白白浪费一周时间），现在开始加急赶进度，将游戏的基本框架搭好，然后开始正式制作游戏流程，争取在今年内发布 Demo 版。

游戏的素材还是很缺，只能先找一些网络素材替代了。