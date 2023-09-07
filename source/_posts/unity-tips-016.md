---
title: 【Unity小技巧】预制体（Prefab）的妙用！
date: 2021-12-24 18:49:15
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
最近几天在实现游戏的技能系统。
游戏类型依然是我最喜欢的策略类回合制。

## 思路
游戏里有许许多多的技能，一个个实现显然是不实际的。
因此通常都是用配置表的方法实现：

```
[
    {
        "id": "0",
        "skill_type": "atk",
        "attribute": "sword",
        "target": "enemy",
        "base_value": 1,
        "atk_power": 100,
        "mat_power": 0,
        "animate": "Hit1",
        "spent_point": 30,
        "hit_count": 1,
        "effects": [
            {
                "target": "enemy",
                "state": "pofang",
                "time": 1,
                "rate": 40
            },
            {
                "target": "enemy",
                "state": "du",
                "time": 1,
                "rate": 50
            }
        ]
    },
    {
        "id": "1",
        "skill_type": "atk",
        "attribute": "sword",
        "target": "enemy",
        "base_value": 1,
        "atk_power": 50,
        "mat_power": 0,
        "animate": "Hit1",
        "spent_point": 50,
        "hit_count": 3,
        "effects": [
            {
                "target": "enemy",
                "state": "pofang",
                "time": 1,
                "rate": 40
            },
            {
                "target": "enemy",
                "state": "du",
                "time": 1,
                "rate": 50
            }
        ]
    },
    {
        "id": "2",
        "skill_type": "recover_hp",
        "attribute": "none",
        "target": "self",
        "base_value": 10,
        "atk_power": 0,
        "mat_power": 0,
        "animate": "test_play_animate",
        "animate_type": "self",
        "spent_point": 20,
        "hit_count": 1,
        "effects": []
    }
]
```

像上面这样配置一些参数，然后用脚本解析参数实例化对应的技能效果。
这样对于一些简单的回合制来说已经足够了。
但我现在制作的是策略性的回合制，技能的效果不仅仅只是有几率上个 debuff 之类的。

策略类回合制的特点是技能具有多样性。
下面是几个简单的例子：

- 【必杀斩】当敌人HP低于 10% 时，直接击杀目标。
- 【连续斩】攻击敌人，并附加 1 层【流血】状态，当敌人身上存在 3 层流血状态时，移除所有流血，每 1 层流血状态额外造成 20% 伤害。
- 【治疗术】回复自身 20% HP，并解除所有负面效果。

这样的技能系统，直接用配置的方法是很难实现的。
你无法把所有可能的效果全部做成配置，那样会使得整个配置表凌乱不堪。

## 拆分功能
其实主动技能的效果虽然很多，但其实可以分成几个大类：

- 攻击类
- 回血类
- 增益、负面效果类

基本就是以上三大类型。
然后子类可以再互相结合，比如攻击命中敌人给敌人上个 debuff 什么的。
技能的种类虽然多，但始终是围绕上面三类变化的。

既然大部分都具有共通性，那么这些都可以抽取出来。
我们要处理的是那些比较【刁钻】的技能效果。

> 【连续斩】攻击敌人，并附加 1 层【流血】状态，当敌人身上存在 3 层流血状态时，移除所有流血，每 1 层流血状态额外造成 20% 伤害。

上面的连续斩就是一个最好的例子。
这样的技能效果很难用配置的方法实现，因而需要特殊处理。

## 预制体制作技能
配置表依然是需要的，但为了兼容丰富多样的技能效果。
新的配置只抽取出技能最基本的几个参数：

```
{
    "name": "技能名称",
    "animate": "技能动画",
    "power": "技能威力",
    "skill_type": "技能类型（如物理攻击）",

    // .. 其他自定义参数

    // 特殊字段
    "instance": "实例脚本"
}
```

上面的配置添加了一个特殊的字段 `instance`，这个字段**存储预制体的名字**。
然后我们在 Unity 的编辑器中，新建一个空白的 GameObject 对象，假设命名为：“lianxuzhan”。

接着创建一个技能脚本的基类：

```
public abstract class Skill {

    // 技能的实际处理方法，参数包括技能配置参数SkillData，施法者attacker和目标target
    public abstact void Handle(SkillData skillData, Character attacker, Character target);

}
```

上面是一个简单的单体目标技能基类。
然后创建一个脚本 `Skill_LianXuZhan`：

```

public class Skill_LianXu_zhan : Skill {

    public override void Handle(SkillData skillData, Character attacker, Character target) {
        // .. 实现具体的技能逻辑
    }

}

```

然后玩家释放技能的时候，实际上是在游戏场景中创建了这个预制体。
技能的逻辑其实是在预制体上处理的。
技能基类还可以再添加一些回调，当技能动画结束的时候，轮到下一个角色行动之类的。

对于一些很常见的技能效果，比如：攻击敌人，造成100%物理伤害。
这种简单技能只有技能威力的区别，直接封装成一个预制体即可实现通用。

还有一些比如：攻击敌人，造成100%物理伤害，30%几率附加流血状态。
这种也属于简单类的技能，就是攻击和附加状态的组合，也可以封装成一个通用的预制体。

参数可以参考：

```
{
    "id": "1",
    "skill_type": "atk",
    "attribute": "sword",
    "target": "enemy",
    "base_value": 1,
    "atk_power": 50,
    "mat_power": 0,
    "animate": "Hit1",
    "spent_point": 50,
    "hit_count": 3,
    "effects": [
        {
            "target": "enemy",
            "state": "pofang",
            "time": 1,
            "rate": 40
        },
        {
            "target": "enemy",
            "state": "du",
            "time": 1,
            "rate": 50
        }
    ]
},
```

这里的 effects 字段就是追加几率状态效果：

```
"effects": [
    {
        "target": "enemy",
        "state": "pofang",
        "time": 1,
        "rate": 40
    },
    {
        "target": "enemy",
        "state": "du",
        "time": 1,
        "rate": 50
    }
]
```

如此一来，虽然技能的种类很多，但只要我们把一些基本的都封装成通用预制体，通过参数进行配置即可；而对于一些难以用配置实现的技能，也可以单独书写一个类，来实现这些特殊技能。

## 补充说明
预制体其实只需要一个，并且是空的 GameObject 对象即可。
它们的差别在于 Component（组件），即我们只需要给它们动态添加脚本即可。

在 Unity 中，可以使用如下方法添加脚本：

```
/**
    * 角色使用技能
    */
public void UseSkill(FormatterSkill skill)
{
    var obj = ObjectBuilder.CreateObject("BattleSkill");
    var type = Type.GetType(skill.component);

    var component = obj.AddComponent(type) as Battle_SkillAbstract;
    component.Handle(skill, this);

    isUsingSkill = true;
}
```

上面是我游戏中使用技能的方法，`skill.component` 是一个字符串，该字符串将会转化为对应的脚本组件，例如传入一个普通攻击技能的组件名字：`Skill_NormalAttack`，那么该方法就会自动为 GameObject 添加此脚本组件，只需要新建一个脚本：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Skill_NormalAttack : Battle_SkillAbstract
{
    public override void Handle(FormatterSkill skill, Battle_Character user)
    {
        this.skill = skill;
        this.user = user;

        target = user.GetEnemy();

        ShowSkillName();

        user.animator.PlayAttackAnimate();
       
        // ... 省略其他逻辑
    }
}
```

如此一来，即使技能的种类繁多，我们亦可实现，并且代码会十分简洁。

## 结尾
不仅技能的实现可以用预制体，剧情也可以用这种方法。
当满足剧情的触发条件，就创建一个预制体，把剧情动画、对话等等全部在预制体的类里面实现，而不是直接在场景的脚本里面写，如果直接把所有的代码写在场景的脚本，估计能上千行，这对于后期维护是不利的。

这样后期十分容易维护，比如我们想改动某段剧情的演出效果，只要修改对应那个 prefab 的脚本即可。

**把复杂的逻辑处理，用预制体实现吧！**