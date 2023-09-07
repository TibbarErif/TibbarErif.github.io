---
title: 《名为怪物的游戏》框架开发篇（优雅的实现菜单系统）
date: 2021-08-17 17:46:22
tags:
 - unity
 - 开发技术

categories:
 - 新游开发
 - 名为怪物的游戏
---
## 前言
新素材制作期间，我打算利用这段空闲的时间重构之前写的一些不好的地方。

游戏开发中 UI 的处理是比较复杂的部分。UI 包括窗口、状态栏等等，其中大部分都是展示作用，没有需要控制的地方。本篇文章介绍了一种通过生命周期来实现需要控制的 UI，如菜单系统，优雅的实现方法。

## 菜单示例
之前实现的主菜单/存档界面。

![主菜单/存档界面](https://pic.imgdb.cn/item/60e7f75e5132923bf8060305.gif)

具体的实现可以前往之前的日志：[菜单的控制权问题](https://huotuyouxi.com/2021/07/01/monster-game-version-100-1/#7%E6%9C%889%E6%97%A5)

菜单系统的难点在于操作控制权的变换。
比如先打开主菜单，然后选择存档，此时主菜单应该被“暂停”不能操控。
而关闭存档时，操作权才会返回主菜单。

这个原理是通过栈实现的，不再赘述。

原来的系统虽然已经没问题了，但是代码写的有点散乱。
在经过了一番学习之后，有了更加系统化的思路，所以我决定来优化一下菜单系统。

## UI 模型基本思路
菜单系统的权限控制非常繁琐，如果是面向过程的开发，简直要崩溃……
现在要做的事就是开发一个 UI 框架来自动处理这些麻烦的事情，解放自己的双手。

### 基本结构
UI 框架一共包括两个类：

- UIModel
- UIStack

`UIModel` 是菜单系统的父类，该类包含了 UI 从创建到销毁的处理。
`UIStack` 是菜单的管理类，用来控制菜单的权限自动化处理。

为了统一规范，所有的 UI 都应当用 `Image` 来实现，而不是 `SpriteRender`。
在 Unity 中，UI 类型的组件必须挂在 `Canvas`（画板）底下。
因此，在场景中创建一个画板，然后把 `UIStack` 脚本挂在画板里，实现 UI 的全局管理。

![UICanvas](https://pic.imgdb.cn/item/611b8c174907e2d39c14539f.jpg)

主场景中的结构如上图所示。
接着在主场景的脚本中添加一个获取 `UIStack` 的变量，并且将 `UICanvas` 拖进去：

![UIStack](https://pic.imgdb.cn/item/611b8c5c4907e2d39c1501b3.jpg)

这样主场景就可以对 UI 进行管理了。

### 生命周期
不管是什么框架都有着『生命周期』的概念。
即创建时、创建后、销毁时……诸如此类。

生命周期指的是一个系统从创建开始，直到运行结束销毁回收的过程。
为了让代码的条理更加清晰，新的系统也引入生命周期的概念。

> Unity 中的 Awake、Start、OnDestroy 就是生命周期函数

通过生命周期来管理一个菜单 `UIModel` 的创建以及销毁的过程。

### 控制权
菜单的基本实现就是利用了栈的特性，在入栈时将上级菜单暂停，在出栈时才解除暂停状态。
栈的结构可以用链表来轻松实现，`C#` 中就有链表类型 `LinkedList`。

> 这个类型也是我刚刚发现的……

链表主要提供两个方法：`Pop` 和 `Push`。

`Pop` 方法从尾部取一个元素，并将其从链表中删除。
`Push` 方法将一个元素添加到链表的尾部。

在这两个操作的过程中即可对菜单的权限进行处理。
在新菜单入栈时，将上级菜单暂停；当菜单关闭时，将上级菜单解除暂停。

## UIModel
`UIModel` 是菜单系统的父类，包含了对生命周期的处理。

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace FR
{
    public abstract class UIModel : MonoBehaviour
    {
        protected UIStack stack;

        private void Awake()
        {
            OnCreate();
        }

        private void Start()
        {
            OnLoad();
            OnLoadAction();
        }

        private void OnDestroy()
        {
            OnRelease();
        }

        private void Update()
        {
            OnUpdate();
        }

        private void FixedUpdate()
        {
            OnFixedUpdate();
        }

        public void BindStack(UIStack stack)
        {
            this.stack = stack;
        }

        protected abstract void OnLoadAction();

        /**
         * 让UI处于暂停状态，不可进行操作
         */
        public abstract void Pause();

        /**
         * 从暂停状态恢复
         */
        public abstract void Resume();

        /**
         * 关闭UI是一件复杂的行为
         * 此处暴露一个简单的方法以便外部直接调用
         */
        public void Close()
        {
            if (stack == null) return;

            stack.Pop();
        }

        /**
         * 仅提供给UIStack调用
         * 执行关闭动画，返回动画的时间
         * 当动画播放结束后，会被stack回收
         * 这里只需要执行关闭窗口的效果，而不需要执行Destroy，销毁由stack执行
         */
        public abstract float OnCloseAction();

        /**
         * 生命周期
         */
        protected virtual void OnCreate() { }
        protected virtual void OnLoad() { }
        protected virtual void OnUpdate() { }
        protected virtual void OnFixedUpdate() { }

        /**
         * 在被Destroy之前再调用一次OnCloseBefore
         * 然后就会被Destroy销毁
         */
        public virtual void OnCloseBefore() { }

        /**
         * 执行了Destroy之后最后调用一次OnRelease
         * 整个生命周期就结束了
         */
        protected virtual void OnRelease() { }
    }
}
```

相关的说明见注释部分。

## UIStack
`UIStack` 以栈结构保存所有的菜单，并且在出入栈时对权限进行管理。

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace FR
{
    [RequireComponent(typeof(Canvas))]
    public class UIStack : MonoBehaviour
    {
        private RectTransform rectTransform;
        private LinkedList<UIModel> uiModels = new LinkedList<UIModel>();

        private void Awake()
        {
            rectTransform = GetComponent<RectTransform>();
        }

        public UIModel Push(GameObject prefab)
        {
            if (!prefab)
            {
                Debug.LogError("prefab为空");
                return null;
            }

            var component = prefab.GetComponent<UIModel>();

            if (!component)
            {
                Debug.LogError("没有UIModel组件");
                return null;
            }

            // 如果在链表中存在其他的UI，则把最后的一个UI设置为暂停状态
            var last = (uiModels.Last != null) ? uiModels.Last.Value : null;
            if (last)
            {
                last.Pause();
            }

            // 调用Unity的Instantiate方法实例化UI对象
            var obj = ObjectBuilder.Generate(prefab, rectTransform);

            // 将当前类绑定到UI类中，以便其内部调用
            var model = obj.GetComponent<UIModel>();
            model.BindStack(this);

            // 将新加入的UI类添加到链表的尾部
            uiModels.AddLast(model);

            return model;
        }

        public void Pop()
        {
            if (uiModels.Last == null) return;

            // 取出最后一个UI类并将其移出链表
            var model = uiModels.Last.Value;
            uiModels.RemoveLast();

            // 移除绑定
            model.BindStack(null);

            /**
             * 执行窗口的关闭动画，并且获得动画关闭的等待时间
             * 其实这一步是并行执行的，动画仍在播放，但是先返回一个等待时间给UIStack调用
             * 直接返回动画的时间目的在于解耦
             * 并行处理的好处是降低代码的耦合，但坏处是关闭动画的时间太长时会明显感觉不协调
             * 即在关闭当前UI后，此时UI仍在播放关闭动画，可是上级UI却可以在播放动画期间进行操作
             * 因此在获得动画播放时间的时，进入协程等待状态（与动画播放时间相同）
             * 以此保证关闭动画结束时正好解除上级UI的暂停状态
             */
            float time = model.OnCloseAction();

            StartCoroutine(WaitCloseActionCompleted(model, time));
        }

        private IEnumerator WaitCloseActionCompleted(UIModel model, float time)
        {
            yield return new WaitForSeconds(time);

            var last = (uiModels.Last != null) ? uiModels.Last.Value : null;
            if (last)
            {
                last.Resume();
            }

            model.OnCloseBefore();

            Destroy(model.gameObject);
        }
    }
}
```

该类需要挂在场景的 `Canvas` 节点里。
`Push` 方法通过传入 `Prefab`（预制体）参数来实例化菜单节点。
同时将新增的菜单的脚本加入到栈里面，并且对上级菜单进行了暂停操作。

`Pop` 方法是将最后一个打开的菜单进行销毁，同时解除上级菜单的暂停状态。
此处添加一个协程用来处理关闭动画，也可以不加这个方法而是直接销毁菜单。

## 游戏中使用
这两个类属于框架类，不包含任何游戏相关的逻辑，比如菜单的按键控制等。
因此需要在游戏项目中，通过继承 `UIModel` 的方法来创建一个菜单。

### 菜单 UI
在场景中创建一个简单的白底菜单，包括几个选项。

![菜单UI](https://pic.imgdb.cn/item/611b96174907e2d39c2bc70e.jpg)

目前这个菜单只是一张静态的图，没有控制功能。

### 菜单逻辑
菜单有水平选项的，也有竖直方向的，还有格子排列的。
每一种菜单的控制操作都不同，但是其他的地方却基本相同，因此可以封装一个所有类型菜单的基类。
然后不同的菜单只要继承这个类，再去实现对应的控制方法即可。

#### 水平菜单
水平方向的菜单只能左右移动选项，选项只有一个横排。

![水平菜单](https://pic.imgdb.cn/item/611b97624907e2d39c2eaef0.jpg)

#### 格子菜单
格子菜单可以上下左右按键控制，选项至少有两排。

![格子菜单，选项可以上下左右操作](https://pic.imgdb.cn/item/611b96934907e2d39c2ce048.jpg)

#### 竖直菜单
竖直方向的菜单只能通过上下键移动选项，选项只有一个竖列。

![竖直菜单](https://pic.imgdb.cn/item/611b96e04907e2d39c2d8bdb.jpg)

## 具体实现
接下来演示如何实现一个竖直方向的菜单系统，其他类型的菜单原理大同小异。

### 菜单基类

首选创建所有菜单的基类 `MenuBase`：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using FR;

namespace Refactor
{
    public abstract class MenuBase : UIModel
    {
        public Transform itemLayout;
        public List<MenuItemBase> menuItems;
        protected int currentIndex;

        protected bool isPause, disableCancle, isDestorySelf, isDestoryParent;

        protected override void OnCreate()
        {
            InitStatusAction();
        }

        protected override void OnLoadAction()
        {
            SetDefaultStatus();
        }

        public void SetDefaultStatus()
        {
            menuItems[0].SetActiveStatus();
        }

        /**
         * 设置为禁止取消【X键】关闭菜单
         */
        protected void SetDisabledCancle()
        {
            disableCancle = true;
        }

        /**
         * 将菜单设置为自销毁类型
         * 即在确定的时候会调用关闭事件
         * 设置为false表示这是个上级菜单，在打开子菜单的时候保留窗口
         * 设置为true时，菜单在完成事件后会执行销毁操作（只是销毁自身不包括上级菜单）
         */
        protected void SetDestroySelfOnCompleted()
        {
            isDestorySelf = true;
        }

        /**
         * 菜单完成操作时，是否销毁自身及所有上级菜单
         * 即在确定的时候会调用关闭事件
         * 设置为false表示该菜单只关闭自身，上级菜单不受影响
         * 设置为true时，该菜单在完成操作后会连上级菜单也一并销毁
         */
        protected void SetDestroyParentOnCompleted()
        {
            SetDestroySelfOnCompleted();
            isDestoryParent = true;
        }

        public override void Pause()
        {
            isPause = true;
        }

        public override void Resume()
        {
            isPause = false;
        }

        /**
         * 菜单的初始化操作
         * 需要设置是否销毁自身或者关闭时是否销毁上级菜单等
         */
        protected abstract void InitStatusAction();

        /**
        * 监听竖直和水平方向按键
        */
        protected virtual void VeticalPrev() { }
        protected virtual void VeticalNext() { }
        protected virtual void HorizontalPrev() { }
        protected virtual void HorizontalNext() { }

        protected void Handle()
        {
            menuItems[currentIndex].Handle();

            if (isDestorySelf)
            {
                Close();
            }
        }

        protected override void OnRelease()
        {
            if (isDestoryParent)
            {
                stack.Pop();
            }
        }

        protected override void OnUpdate()
        {
            if (isPause) return;

            if (KeyManager.GetConfirmKeyDown())
            {
                Handle();
            }
            else if (Input.GetKeyDown(KeyCode.X) && disableCancle == false)
            {
                Close();
            }
            else if (Input.GetKeyDown(KeyCode.UpArrow))
            {
                VeticalPrev();
            }
            else if (Input.GetKeyDown(KeyCode.DownArrow))
            {
                VeticalNext();
            }
            else if (Input.GetKeyDown(KeyCode.LeftArrow))
            {
                HorizontalPrev();
            }
            else if (Input.GetKeyDown(KeyCode.RightArrow))
            {
                HorizontalNext();
            }
        }
    }
}
```

菜单基类定义了一个菜单具有的基本功能，其中要注意的是：

```
public override void Pause()
{
    isPause = true;
}

public override void Resume()
{
    isPause = false;
}
```

`Pause` 用来实现暂停菜单操作的功能；
`Resume` 用来解除菜单的暂停状态。
这里使用一个布尔型的变量 `isPause` 来控制即可。

一些菜单不希望可以按 `X` 键取消，比如战斗中的操作选项；
一些菜单在按下确定后，只希望关闭当前窗口，而不是连上级菜单也一起关闭；
一些菜单在按下确定键后，希望可以关闭所有打开的窗口。
这些都需要一个布尔型的变量来控制，菜单的基本配置在 `InitStatusAction` 进行设置。

除此之外，所有的菜单都有「选项」以及初始化选项的方法，具体的初始化行为在子类的 `OnLoadAction` 进行实现。

### 不同菜单

接着创建三个基本类型的菜单类：

![基本类型的菜单类](https://pic.imgdb.cn/item/611b99024907e2d39c32926b.jpg)

`HorizontalMenu` 是水平类型的菜单；
`VerticalMenu` 是竖直类型的菜单；
`GridMenu` 是格子类型的菜单。

这些基本大同小异，区别在于控制按键的方法，这边只演示竖直类型的菜单类：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace Refactor
{
    public abstract class VeritalMenu : MenuBase
    {
        protected override void VeticalNext()
        {
            menuItems[currentIndex].SetUnActiveStatus();
            currentIndex++;

            if (currentIndex >= menuItems.Count)
            {
                currentIndex = 0;
            }

            menuItems[currentIndex].SetActiveStatus();
        }

        protected override void VeticalPrev()
        {
            menuItems[currentIndex].SetUnActiveStatus();
            currentIndex--;

            if (currentIndex < 0)
            {
                currentIndex = menuItems.Count - 1;
            }

            menuItems[currentIndex].SetActiveStatus();
        }
    }
}
```

基本方法都在 `MenuBase` 写好了，所以子类只需要实现控制按键的部分即可。

### 测试菜单

接下来创建一个用于测试的菜单 `TestMenu`：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using FR;

namespace Refactor
{
    public class TestMenu : VeritalMenu
    {
        public override float OnCloseAction()
        {
            return 0f;
        }

        protected override void InitStatusAction()
        {
            SetDestroySelfOnCompleted();
        }

        protected override void OnLoadAction()
        {
            for (int i = 0; i < 5; i++)
            {
                GameObject obj = ObjectBuilder.Generate("Prefabs/Test/Refactor/TestMenuItem", itemLayout);
                obj.transform.localScale = Vector3.one;

                var item = obj.GetComponent<MenuItemBase>();

                item.BindParent(this);
                menuItems.Add(item);
            }

            SetDefaultStatus();
        }
    }
}
```

因为没有添加关闭菜单的动画效果，所以 `OnCloseAction` 直接返回一个零即可。
`InitStatusAction` 方法配置菜单的基本参数，将其设置为自销毁类型，即在确定后就关闭。
`OnLoadAction` 方法生成菜单的选项，这里直接用了框架里的 `ObjectBuilder` 来动态创建选项。

### 选项基类

接下来创建选项类的脚本，首先创建一个选项的父类 `MenuItemBase`：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace Refactor
{
    public abstract class MenuItemBase : MonoBehaviour
    {
        protected MenuBase parentMenu;

        protected virtual void Awake()
        {
            OnCreate();
            SetUnActiveStatus();
        }

        public void BindParent(MenuBase parentMenu)
        {
            this.parentMenu = parentMenu;
        }

        public abstract void OnCreate();
        public abstract void SetUnActiveStatus();
        public abstract void SetActiveStatus();
        public abstract void Handle();
    }
}
```

选项的父类包含了选项激活 `SetActiveStatus` 方法与 `SetUnActiveStatus` 未激活。
即当光标移动到该选项时，选项的变化效果，演示的菜单选项会在激活时改变背景颜色。

`BindParent` 方法绑定菜单面板类，方便在子类对父类的一些行为进行控制。
`OnCreate` 即在菜单选项初始化时的操作，比如将选项的文字进行本地化处理。
`Handle` 为菜单选项的实际逻辑，即选中并且按下确认键后要做什么。

### 菜单选项

接着创建 `TestMenuItem`：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

namespace Refactor
{
    public class TestMenuItem : MenuItemBase
    {
        private Image bg;

        public override void Handle()
        {
            Debug.Log("选中");
        }

        public override void OnCreate()
        {
            bg = GetComponent<Image>();
            SetUnActiveStatus();
        }

        public override void SetActiveStatus()
        {
            bg.color = new Color(1f, 0f, 0f);
        }

        public override void SetUnActiveStatus()
        {
            bg.color = new Color(0.5f, 0.5f, 0.5f);
        }
    }
}
```

在初始化方法 `OnCreate` 中，获得 `Image` 组件，并且在激活与未激活状态时对其进行变色处理。
`Handle` 方法打印了一个字符串，当按下确定键后就会打出这个字符串。

以上就完成了一个菜单系统。

### 调用菜单

在 `MainScene` 中调用：

```
public void CreateMainMenu()
{
    GameObject prefab = Resources.Load<GameObject>("Prefabs/Test/Refactor/TestMenu");
    var model = uiStack.Push(prefab);

    model.transform.localScale = Vector3.one;
    model.transform.localPosition = Vector3.zero;
}
```

只需要载入菜单的预制体，然后将预制体推送到 `UIStack` 即可创建出菜单了。

## 总结
新的菜单系统利用生命周期来控制菜单的行为，极大的减少了开发的负担。
而且这样一个条理清晰的结构，后期维护起来也简单。