---
title: 【Unity小技巧】对于空值异常NullPointerException的优雅处理方式
date: 2023-01-27 21:06:15
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
空指针异常（NullPointerException）在开发过程十分常见，在游戏开发中，因为基本所有的代码都在异步执行，很容易在某个地方销毁了对象，结果另一个地方却依然在调用，这就导致获取到空对象因而报错。
## 示例
在游戏剧情模式下，我们需要生成一些 NPC 用来演出，比如 Player（玩家），Jonna（洁娜）等等，现在我们生成了这两个对象，并且用一个 `Dictionary<string, GameObject>` 字典将 NPC 的名称和实例化出来的 GameObject 保存起来，现在我们就可以用下面的方法来调用 NPC 对象了：

```
/// <summary>
/// 获取NPC对象
/// </summary>
/// <param name="npcName"></param>
/// <returns></returns>
protected GameObject GetNPC(string npcName)
{
    return npcs[npcName];
}
```

可是这个方法根本没有任何除错措施，上面我们只定义了 Player 和 Jonna 两个 NPC，如果我们调用了 Char（卡尔）呢？我们根本就没创建过这个 NPC，当然会直接报错，字典里没有这个 Key！

为了防止调用不存在的 NPC，我们可以加入一个判断：

```
protected GameObject GetNPC(string npcName)
{
    if (npcs.ContainsKey(npcName))
    {
        return npcs[npcName];
    }

    return null;
}
```

好了，这样这段代码肯定不会报错了，可是，在其他地方，我们需要调用这个 NPC 进行移动，这里却返回了一个 null，不出意外的弹出空指针异常：

```
// 将 Char NPC 移动到 x=-7f 
Move(GetNPC("Char"), -7f);
```

那我们是不是也可以照猫画虎，在移动的方法里也写一个判断空值？

```
var npc = GetNPC("Char");
if(npc != null) {
    Move(GetNPC("Char"), -7f);
}
```

大功告成，这里也不会报错了！可是……如果下面又有一个演出事件，我们要让 Char 播放攻击动画呢？

```
var npc = GetNPC("Char");
if(npc != null) {
    PlayAnimate(npc, "Attack")
}
```

好了，只要在调用 NPC 对象的时候，判断他是否是空值就可以解决报错问题了！

我们可以进一步的优化一下，在调用 NPC 的方法里进行判断：

```
protected void PlayAnimate(string npcName, string anim) {
    var target = GetNPC(npcName);
    if(target != null) { 
        target.GetComponent<Animator>().Play(anim);
    }
}
```

然而，像这种操作 NPC 行为的方法有很多个，也不可能一个个都这么判断，冗余代码实在太多了！**除非你真的很喜欢写代码，否则不要写重复代码！**

## 解决
我们可以将获取 NPC 的方法封装起来，然后增加一个回调方法，如果获取 NPC 成功，则将 NPC 对象传给回调方法进行处理，否则就报错，如下所示：

```
 /// <summary>
/// 获取NPC对象，当获取成功时执行callback
/// </summary>
/// <param name="npcName"></param>
/// <param name="callback"></param>
/// <returns></returns>
protected void GetNPC(string npcName, System.Action<GameObject> callback)
{
    if (npcs.ContainsKey(npcName) && npcs[npcName] != null)
    {
        callback(npcs[npcName]);
    }
    else
    {
        SystemUtil.GetInstance().Print("NPC为空：" + npcName);
    }
}
```

好了，大功告成！在需要调用 NPC 对象的时候，只需要像下面这样：

```
GetNPC(npcName, (target) =>
{
    var anim = target.GetComponent<Animator>();
    anim?.Play(animate);
});
```

是不是优雅了很多呢？