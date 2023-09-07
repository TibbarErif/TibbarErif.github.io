---
title: 【Unity小技巧】实现存档/读档系统以及存储文件本地化
date: 2021-09-26 10:26:15
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
存档系统基本上是所有游戏必备的，存档的本质就是将内存中的变量以文件的形式保存下来；而读档则是将本地文件转化为内存中的变量。

## 存储方式
unity 自带的存储功能适合存储简单的数据，如果需要保存复杂的数据，可以借助第三方插件实现。

### PlayerPrefs
unity 自带了一个 `PlayerPrefs` 来保存本地文件，`PlayerPrefs` 是一个简单的键值对形式数据存储系统，通过 unity 提供的内置方法可以简单的实现数据存储和读取，一般用来保存简单的数据，比如游戏的系统配置，以及一些小游戏的存档数据。

数据的存储示例：

```
//存储整型数据
PlayerPrefs.SetInt("yourKeyName",999); 

//存储浮点型数据
PlayerPrefs.SetFloat("yourKeyName",1.11f); 

//存储字符串数据
PlayerPrefs.SetString("yourKeyName","my name is kangkang");
```

数据的读取示例：

```
//取出key为"yourKeyName"的整型数据
int intVal = PlayerPrefs.GetInt("yourKeyName"); 

//取出key为"yourKeyName"的浮点型数据
float floatVal = PlayerPrefs.GetFloat("yourKeyName"); 

//获取key为"yourKeyName"的字符串数据
string strVal = PlayerPrefs.GetString("yourKeyName");
```

删除数据示例：

```
//删除所有存储数据
PlayerPrefs.DeleteAll();

//删除key为"yourKeyName"的数据
PlayerPrefs.DeleteKey("yourKeyName");
```

判断数据是否存在：

```
//查找是否存在key为"yourKeyName"的数据
bool exist = PlayerPrefs.HasKey("yourKeyName")
```

注意，由于是 unity 内置的方法，所以存储位置就由 unity 来决定，根据操作系统的不同，存储的位置也不同。

- 在 Mac OS X 平台下，存储在 ~/Library/Preferences 文件夹，名为 unity.[company name].[product name].plist。
- 在 Windows 平台下，存储在注册表的 HKEY_CURRENT_USER\Software[company name][product name] 键下。

其他需要注意的地方：当取一个不存在的键的值时，返回的是 `c#` 的默认值：

```
int intVal = PlayerPrefs.GetInt("yourKeyName");
Debug.Log(intVal);
```

上面的例子中，`yourKeyName` 并没有被设置过，因此返回 `int` 的默认值即 0。

`PlayerPrefs` 能存储多大的数据暂时在网上找不到答案，不过一般都是存储一些安全性不高的简单数据，所以不推荐将 json 转化成字符串存储到 `PlayerPrefs`，而且如果有玩家知道 unity 的特性就可以找到对应的文件然后随意修改，安全性不高。

`PlayerPrefs` 没办法存储复杂的数据类型，比如字典、列表，只能将其转换成 json 字符串，然后再保存为字符串形式，在读取的时候再将 json 字符串转化为 json 对象，这样的步骤十分繁琐。

对于一般的游戏存档数据，推荐用下面的方法。

### JSON
`c#` 和 unity 没有提供直接存储和读取 JSON 的方法，于是我们需要借助第三方插件来实现，JSON 插件有很多种，这里我使用的是 `LitJSON`。

LitJSON 官网：[https://litjson.net/](https://litjson.net/)

下载插件的 DLL 文件，然后拖进游戏目录就可以自动加载插件了。这样我们就可以在脚本通过 `using LitJson` 引入这个插件，就可以直接使用插件内置的方法。 

游戏存档数据是一个类文件，用来保存数据的字段，`Savefile`（存档数据）：

```
using System.Collections;
using System.Collections.Generic;
using Core.Information;

namespace Core.Save
{
    public class Savefile
    {
        // 当前场景
        public string currentScene;

        /**
         * 保存对话进度相关
         */
        public string currentDialogCommand;
        public int currentDialogIndex;

        /**
         * 触发器开关
         */
        public Dictionary<string, bool> switchKeys = new Dictionary<string, bool>();

        /**
         * 已收集的信息
         */
        public List<string> informationDatas = new List<string>();

        /**
         * 变量
         */
        public Dictionary<string, int> values = new Dictionary<string, int>();
    }
}
```

存档数据就是一个内存中的变量，要把变量存储到本地文件，或者把本地存档加载到内存变量，就需要一个用来读写文件的系统 `FileSystem`：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using LitJson;
using System.IO;
using Core.Save;

namespace Util
{
    public class FileSystem
    {
        public static string savefilePath = Application.dataPath + "/Savefile/";

        public static Savefile LoadSavefile(int index)
        {
            string realPath = GetRealPath(index);

            Savefile data = null;

            if (File.Exists(realPath))
            {
                string json = File.ReadAllText(realPath);
                data = JsonMapper.ToObject<Savefile>(json);
            }

            return data;
        }

        private static string GetRealPath(int index)
        {
            return savefilePath + "/save" + index + ".json";
        }

        public static void Save(Savefile data, int index)
        {
            string realPath = GetRealPath(index);
            string jsonData = JsonMapper.ToJson(data);

            File.WriteAllText(realPath, jsonData);
        }
    }
}
```

上面的文件读取系统中：

`savefilePath = Application.dataPath + "/Savefile/"` 用于指定保存存档文件的目录。

我们要保存成文件形式，需要使用 `System.IO` 来对本地文件进行处理，我给每一个存档文件一个 `index`（编号）用来区分不同的档案位置。

`JsonMapper` 是 LitJSON 提供的方法，它可以将对象转换成 json 字符串或者将 json 字符串转化为对象，如上面的示例中：

```
string json = File.ReadAllText(realPath);
data = JsonMapper.ToObject<Savefile>(json);
```

首先通过 `File.ReadAllText` 读取本地的文件，然后再调用 `JsonMapper.ToObject<T>(string)` 将字符串转化成 `T` 类型的类。

保存在本地的存档文件如下所示：

```
{"currentScene":"ArcadeHall","currentDialogCommand":"00001","currentDialogIndex":0,"switchKeys":{},"informationDatas":["People.huzi"],"values":{}}
```

调用加载存档的方法：

```
public static Savefile LoadSavefile(int index)
{
    string realPath = GetRealPath(index);

    Savefile data = null;

    if (File.Exists(realPath))
    {
        string json = File.ReadAllText(realPath);
        data = JsonMapper.ToObject<Savefile>(json);
    }

    return data;
}
```

上面的方法直接将本地文件转化成 `Savefile` 类型的变量，这样我们就实现了读取存档数据的功能。

## 执行新存档
现在只是读取到了存档的数据，只是把存档数据加载到内存变量里，可还没有真正的实现“读档”的功能。
角色还傻傻的停留在当前地图，真正的读档应该销毁现在的场景，让角色出现在新存档记录下的位置。

下面是我开发中的文字游戏的读档操作：

```
void LoadSavefile(Savefile savefile)
{
    // 将当前存档设置为读取的存档数据
    GlobalManager.SetSavefile(savefile);

    // 移除canvas下的所有对象
    foreach (Transform child in GlobalManager.gameManager.canvas.transform)
    {
        Destroy(child.gameObject);
    }

    // 当前场景设置为空
    GlobalManager.currentScene = null;
    // 加载新的场景
    GlobalManager.LoadScene(savefile.currentScene);
}
```

在读取存档数据之后，把当前的场景清空，重新加载存档的场景数据，这样才真正完成了读档功能。

在加载场景之后，场景的自动执行事件会被再执行一次。
因此需要注意事件的开关状态，或者在执行读档操作时，屏蔽自动执行事件。

## 存档的加密
现在存档数据是“明文”的，也就是说玩家找到存档文件就可以自由进行编辑，比如某个道具的数量改成 999，或者把金钱调成最大等等。我们并不希望玩家可以这样“作弊”，接下来就必须得给存档加密。

你可以自己编写加密方法，也可以引入第三方插件，或者直接使用 base64 进行简单加密。
总之，这一步就是把 json 字符串加密之后再保存到本地，如果使用了加密，在读取存档的时候要记得解密。

## 存档的升级
对于未完成的游戏，在给玩家发布了体验版之后，后续的更新如果修改了存档数据，那么旧版的存档就可能会出现不兼容报错的情况，为了解决这个问题，我们可以给存档数据加上一个字段：`string version = "0.0.1"`，这个字段用来保存当前游戏的版本号，这样玩家如果在更新了游戏版本之后，游戏内部添加一个检测机制，用来对不同版本的存档文件进行升级。


假如游戏版本升级到 `0.0.2`，并且存档数据多了一个字段 `string extraVal = "hahaha"`，但是玩家旧版的存档并没有这个字段，我们就需要给本地旧版的文件进行升级，加上这个字段：

示例代码：

```
if(savefile.version == "0.0.1") {
    // ...存档版本号更新
    savefile.version = "0.0.2";

    // ...给新添加的变量赋值
    savefile.extraVal = "hahaha";

    // ...重新保存本地文件
}
```

上面用伪代码简单示意，总之就是将旧版存档的字段进行更新再重新保存即可。