---
title: 比较的完美对话系统
date: 2021-09-14 01:33:01
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
从 cocos creator 开始到现在用 unity 制作了很多个失败的作品，并非一无所获。
虽然玩家们不知道我们到底做了什么，但实际上我们没有发布出来的失败作品差不多有五六个。
如果我推翻重构的也算上去，大概失败了有十次以上吧。

基本每一个项目都离不开对话系统，因此对话系统反反复复重置也有好几个版本。这是用新的方法实现的第五版对话系统。如何实现 GalGame 那样的对话系统一直都是一个难题，如今终于有一个比较完美的解决方案了。

那么就来看看是如何实现的吧！

## 基本思路
最开始要考虑的问题在于如何存储动则几万字的文本。

游戏中剧本很多的情况下就不能用简单的直接在代码上面输出文本。
而且还需要考虑到以后有多语言化的需求，比如要出一个官方英文版。
总不能直接在代码里把中文改成英文的，也就是说不能“硬编码”把文本与代码耦合在一起。

如下面这种就属于硬编码，代码与文本数据耦合在一起：

```
// 假设这是一个对话处理类
Dialog.ShowText("你好，世界！");
```

游戏发售后，发现海外用户有需求，steam 上面 “we need english.” 呼声很大。
因此制作者决定出一个英文版，修改原来的代码：

```
Dialog.ShowText("hello,world！");
```

如果以后又要出一个日文版，又得再改成日文的。
上面这种硬编码只适合一些文本量很小的游戏。

### 存储容器
WEB 开发用来存储数据的是数据库之类的东西，而单机游戏就是本地文件了。
下面几种方法都可以当做对话文本的存储器：

- excel 表格
- txt 文本文档
- json 文件

上面三个任意一个都可以用来作为保存文本的存储器，txt 和 excel 比较直观，适合多人开发且策划为不懂技术的，策划不懂技术但是可以直接在 excel 或者 txt 上面配置，这也是一般的做法；因为我就是游戏程序并且负责录入文本，所以我这里选择方便读取的 json 文件作为保存对话的容器。

### 插件下载
在 `c#` 中可以用 `LitJson` 插件来实现读取 json 格式的文件。

LitJson 下载：[https://github.com/LitJSON/litjson](https://github.com/LitJSON/litjson)
将 dll 文件放到项目里，unity 就会自动读取这个插件了。

使用这个插件还有一个好处是可以直接把 json 字符串转换成对应的类的实例，下文会详细解释。

### 功能说明
对话系统是一个很庞大的系统，它不仅仅只是展示剧本，更是一个剧情演出的处理器。剧情演出就是画面闪烁、屏幕震动、视野改变或者让玩家等待 1 秒钟这种和游戏流程有一定关系的处理。

例如下面这是一个游戏的剧本，剧本里包含了演出效果：

```
[播放BGM：教室内]
小明：这次又考砸了。
小明：我的好基友不会背叛我的，一定是这样！

[播放音效：脚步声]
小明：小黑，这次考了多少啊。
小黑：唔……刚好及格。

[屏幕震动]
[播放音效：震惊]
小明：什么？！
小明：那这次我岂不是垫底了！
```

剧本通常都包含上面这样的演出效果，所以对话系统并不只是显示文字而已，也应当包含这些演出效果的处理。

### 数据驱动
如果是基于代码驱动，如下：

```
// 播放背景音乐
Sound.BGM("教室内");

// 显示第一段对话
Dialog.ShowTextCommand("0001");

// 播放音效
Sound.Play("脚步声音效");

// 显示第二段对话
Dialog.ShowTextCommand("0002");

// 屏幕震动效果
Screen.Shock();

// 播放音效
Sound.Play("震惊");

// 显示第三段对话
Dialog.ShowTextCommand("0003");
```

我们不可能每一段演出都要手写，如果想要节约人力，最好的方法就是用数据驱动的方式来实现。数据驱动就是通过定义一些关键词，通过解释器读取这些关键词，然后根据配置的参数自动处理。

例如定义下面的数据：

```
[
    {"type":"bgm", "id":"教室内"},
    {"type":"text", "id":"0001"},
    {"type":"sound", "id":"脚步声音效"},
    {"type":"text", "id":"0002"},
    {"type":"shock"},
    {"type":"sound", "id":"震惊音效"},
    {"type":"text", "id":"0003"},
]
```

在一个 json 文件配置好上面的参数，再写一个解释器用来将数据解析成对应的脚本逻辑。比如只要用一个 `switch` 结构就可以实现：

```
JsonData json = {"type":"bgm", "id":"教室内"};

switch(json["type"]) {
    case "bgm":
        Sound.BGM(json["id"]);
        break;
    case "text":
        Dialog.ShowTextCommand(json["id"]);
        break;
    // …… 
}
```

上面的例子就是利用数据驱动来处理演出效果。

### 可视化配置
使用数据驱动最大的问题就在于配置数据表，上面的 json 字符串完全是我手写的，如果是完整的剧本，要让我手写的话估计会疯掉。因此如何实现可视化配置就是一个问题了，今天来不及实现，明天我将会用老本行 WEB 开发的方式来制作一个「剧本生成器」。

## 命令模式
其实上面已经不知不觉接触到了「设计模式」，在设计模式中有一个叫做「命令模式」的，就是用数据驱动来实现的。关于命令模式可以自行网上查阅，这里只用一个简单的比方进行介绍：

比如夏天到了我们要开空调，然后用遥控器打开空调开关，然后按加减键来调节温度。这就是命令模式了，空调插上电处于待机状态，等待接收遥控器发来的「指令」，根据发来的指令进行下一步的操作。

命令模式一共有三个角色：

- 发布命令的人
- 接受命令的人
- 命令的传递介质

比如上面的遥控器就是发布命令的人，空调就是接受命令并执行的人，而遥控器发出的信号就是传递的介质，遥控器通过信号把命令传达给空调，如果是与安全相关的还需要加密处理。

在对话系统中，三个角色就是：

- 命令本身
- 命令发布者
- 命令的执行者

命令本身就是类似：“我要你显示 xxx 文本。”，命令发布者要把这条消息发给命令的执行者，它才知道要做什么。命令发布者就是供外界调用的公开方法，别人点一下场景中的 NPC 它就会说一段话，就是调用命令发布者来执行对话的。而命令的执行者就是对话系统本身了，它具备了所有的功能，但是自己不会主动执行，而是等着别人告诉他要怎么做。

### Command（命令数据）
“我要你显示 xxx 文本。”，要想让执行者知道这是什么意思，就要定义一个数据结构。

所有命令的父类 `Command`：

```
namespace Dialog
{
    public class Command
    {
        public string type;
    }
}
```

然后再定义各种不同的命令，比如显示对话的命令 `DialogCommand`：

```
namespace Dialog
{
    public class DialogCommand : Command
    {
        public string name;
        public string content;
        public string tachie;
        public string tachie_position;
    }
}
```

播放音效的命令 `PlaySeCommand`：

```
namespace Dialog
{
    public class PlaySeCommand : Command
    {
        public string path;
    }
}
```

诸如此类。
之所以要定义单独的类，主要还是考虑到规范的问题，避免后期数据不统一。
如果时间太久了，很可能会忘掉，但是如果有类的结构约束就不会犯错了。

还有，就是上面说到的 `LitJson` 本身内置一个很好用的功能：将 json 字符串转化成对应的类。

### Invoker（祈求者）
这是一个祈求命令的类，就是命令发布者。外界只要调用这个类的 `Execute` 方法就会开始执行对话；这里声明了一个 List 结构用来存储命令数据，这样做的好处是可以方便的回溯到任意一个命令开始的地方，比如玩家想看之前的一段剧情，就只要修改 `currentIndex`（当前命令的索引）即可从任意的位置重新开始，比如我就定义了一个 `Prev`（上一条对话），避免玩家因为手速太快没看清楚，还可以重新再看一遍。

因为把所有的命令都保存下来了，所以要实现『历史对话记录』也很简单。

![历史对话记录(可以看到所有已经播放过的对话文字)](https://pic.imgdb.cn/item/613f872044eaada739e95d22.jpg)

`Invoker` 代码如下：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using LitJson;
using System;

namespace Dialog
{
    public class Invoker : MonoBehaviour
    {
        public Receiver receiver;

        bool isDisabled, isPressed;
        int currentIndex;
        List<Command> commands = new List<Command>();

        public void Execute(string command)
        {
            receiver.SetPrevBtnEnabled(false);

            TextAsset textAsset = Resources.Load<TextAsset>("Dialog/" + command);
            JsonData data = JsonMapper.ToObject(textAsset.text);

            foreach (JsonData item in data)
            {
                Debug.Log(item["type"]);
                AddCommond(item);
            }

            currentIndex = 0;
            receiver.Execute(commands[currentIndex]);
        }

        private void AddCommond(JsonData item)
        {
            string json = item.ToJson();

            switch (item["type"].ToString())
            {
                case "DialogCommand":
                    commands.Add(JsonMapper.ToObject<DialogCommand>(json));
                    break;
                case "PlaySeCommand":
                    commands.Add(JsonMapper.ToObject<PlaySeCommand>(json));
                    break;
                case "WaitCommand":
                    commands.Add(JsonMapper.ToObject<WaitCommand>(json));
                    break;
            }
        }

        public void OnExecutedCallback(bool autoNext = false)
        {
            if (autoNext)
            {
                NextCommand();
            }
            else
            {
                isDisabled = false;
            }
        }

        public void NextCommand()
        {
            isDisabled = true;

            currentIndex++;

            if (currentIndex >= commands.Count)
            {
                receiver.OnCompleted();
            }
            else
            {
                receiver.SetPrevBtnEnabled(true);
                receiver.Execute(commands[currentIndex]);
            }
        }

        public void PrevCommand()
        {
            isDisabled = true;

            currentIndex--;

            if (currentIndex <= 0)
            {
                currentIndex = 0;
                receiver.SetPrevBtnEnabled(false);
            }

            receiver.Execute(commands[currentIndex]);
        }

        private void Update()
        {
            if (isDisabled || isPressed) return;

            if (Input.GetKeyDown(KeyCode.Space))
            {
                isPressed = true;
            }
        }

        private void FixedUpdate()
        {
            if (isPressed && Input.GetKeyUp(KeyCode.Space))
            {
                isPressed = false;
                NextCommand();
            }
        }
    }
}
```

### Receiver（命令接收和执行者）
首先在场景中拼好对话 UI：

![对话 UI](https://pic.imgdb.cn/item/613f854d44eaada739e630e7.jpg)

`Receiver` 类就是对话系统本身，它包括解析命令数据，并且将命令转化为真正的逻辑。它每次只执行一条命令，执行完毕之后立刻通知 `Invoker`，告诉它命令已经执行完了，然后再由 `Invoker` 来决定接下来要执行的新命令。

```
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using Core;

namespace Dialog
{
    public class Receiver : MonoBehaviour
    {
        public Invoker invoker;
        public Image tachie;
        public GameObject panel, nameBG, contentBG;
        public GameObject skipBtn, prevBtn, autoBtn, logBtn, saveBtn, loadBtn;
        public Text nameText, contentText;

        // 立绘缓存
        Dictionary<string, Sprite> tachieCaches;

        private void Awake()
        {
            tachieCaches = new Dictionary<string, Sprite>();
        }

        Sprite GetTachieFromCache(string key)
        {
            if (tachieCaches.ContainsKey(key))
            {
                return tachieCaches[key];
            }

            string fullPath = "Tachie/" + key;
            Sprite sprite = Resources.Load<Sprite>(fullPath);

            if (sprite == null)
            {
                Debug.LogError(fullPath + "不存在");
            }

            tachieCaches.Add(key, sprite);

            return sprite;
        }

        public void Execute(Command command)
        {
            switch (command.type)
            {
                case "DialogCommand":
                    DialogAction(command as DialogCommand);
                    break;
                case "PlaySeCommand":
                    PlaySeAction(command as PlaySeCommand);
                    break;
                case "WaitCommand":
                    StartCoroutine(WaitAction(command as WaitCommand));
                    break;
            }
        }

        /**
         * 非对话命令结束后，需要调用以隐藏对话框
         * 同时重置对话框状态，如隐藏立绘
         */
        void SetDefaultState()
        {
            panel.SetActive(false);
            tachie.gameObject.SetActive(false);
        }

        IEnumerator WaitAction(WaitCommand command)
        {
            SetDefaultState();

            yield return new WaitForSeconds((float)command.time);

            invoker.OnExecutedCallback(true);
        }

        void PlaySeAction(PlaySeCommand command)
        {
            SetDefaultState();
            invoker.OnExecutedCallback(true);
        }

        void DialogAction(DialogCommand command)
        {
            panel.SetActive(true);

            // 是否展示立绘
            if (command.tachie != "")
            {
                string path = command.name + "/" + command.tachie;
                tachie.sprite = GetTachieFromCache(path);
                tachie.gameObject.SetActive(true);

                // 防止图片变形恢复原本大小
                tachie.rectTransform.sizeDelta = tachie.sprite.rect.size;
            }
            else
            {
                tachie.gameObject.SetActive(false);
            }

            // 是否展示名称
            if (command.name != "")
            {
                nameBG.SetActive(true);
                nameText.text = LocaleManager.GetLocaleText("name", command.name);
            }
            else
            {
                nameBG.SetActive(false);
            }

            contentText.text = LocaleManager.GetLocaleText(command.name, command.content);
            invoker.OnExecutedCallback();
        }

        public void SetPrevBtnEnabled(bool res)
        {
            prevBtn.SetActive(res);
        }

        public void OnCompleted()
        {
            Destroy(gameObject);
        }
    }
}
```

### LocaleManager（本地化处理）
为了方便以后扩展出其他语言版本，所以提前做好本地化的准备：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using LitJson;

namespace Core
{
    public class LocaleManager : MonoBehaviour
    {
        public static Dictionary<string, JsonData> locales = new Dictionary<string, JsonData>();

        public static void LoadLocaleSetting()
        {
            string locale = GetLocaleLanguage();

            string path = "Locale/" + locale;
            TextAsset[] textAssets = Resources.LoadAll<TextAsset>(path);

            foreach (var text in textAssets)
            {
                locales[text.name] = JsonMapper.ToObject(text.text);
            }
        }

        public static string GetLocaleText(string field, string key)
        {
            if (locales.ContainsKey(field))
            {
                return locales[field][key].ToString();
            }

            return "null";
        }

        public static string GetLocaleLanguage()
        {
            return "zh-cn";
        }
    }
}
```

这个类实现了读取本地语言的文件功能，目前只有 `zh-cn`（简体中文）。

### 语言文件
由于本地化语言都是动态读取的，因此放到 `Resources` 文件夹下面。

![本地化语言文件目录](https://pic.imgdb.cn/item/613f882d44eaada739eb0789.jpg)

如 `name.json` 文件用来保存人物名字：

```
{
    "huzi": "虎子",
    "huahua": "花花"
}
```

还有对应角色的台词，如 `huzi.json`：

```
{
    "00001": "虎子第一句对话。",
    "00002": "虎子第二句对话。"
}
```

最后是命令也需要一个 json，编号为 `00001`：

```
[
    {
        "type": "DialogCommand",
        "tachie": "normal",
        "tachie_position": "left",
        "name": "huzi",
        "content": "00001"
    },
    {
        "type": "PlaySeCommand",
        "path": "movement"
    },
    {
        "type": "WaitCommand",
        "time": 1
    },
    {
        "type": "DialogCommand",
        "tachie": "normal",
        "tachie_position": "left",
        "name": "huahua",
        "content": "00001"
    },
    {
        "type": "DialogCommand",
        "tachie": "smile",
        "tachie_position": "left",
        "name": "huzi",
        "content": "00002"
    },
    {
        "type": "DialogCommand",
        "tachie": "smile",
        "tachie_position": "left",
        "name": "huahua",
        "content": "00002"
    }
]
```

命令里加入一个 type 字段来识别不同的类（用来规范化）。

## 演示效果
调用起来也非常简单：

```
GameObject obj = Util.ObjectBuilder.CreateObject("Prefab/UI/Dialog", canvas);
Dialog.Invoker invoker = obj.GetComponent<Dialog.Invoker>();
invoker.Execute("00001");
```

上面的 `00001` 即是命令文件的名字，即 `00001.json`。

最后进入游戏演示：

![演示效果](https://pic.imgdb.cn/item/613f8a9644eaada739eeaf3c.gif)

## 作者的话
这是历经了很多次失败最终总结的成果，虽然游戏没做出来一个，系统倒是完善了不少。

这套对话系统的可扩展性很强，以后应该都可以基于这套系统进行深度化的定制了。