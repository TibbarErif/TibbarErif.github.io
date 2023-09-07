---
title: 【Unity小技巧】Ajax 请求无法正常获取 JSON 数据问题
date: 2022-02-02 11:31:58
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
在制作对话的时候，我用 HTML 实现了一个可视化的剧情生成器，但是今天打开剧情编辑器却发现无法正常加载 JSON 数据，Ajax 请求既没有跨域，又没有发生异常，而是返回正常的 200 状态码，并且控制台也没报任何异常，但就是 success 方法没有执行，反而会执行 error 方法，打印出异常信息，返回的却是正常的数据。

这就十分离奇了。

相关的 JavaScript 代码如下：

```
// 读取角色名称
$.ajax({
    type: "GET",
    url: api + baseApi + "name.json",
    dataType: "json",
    success: (res) => {

        console.log("读取角色名称");

        names = res;

        for (let key in res) {

            $('#dialog-event-name').append(`<option value="${key}">${res[key]}</option>`);

            // 获取角色文本
            $.getJSON(api + dialogApi + key + ".json", (res) => {
                contents[key] = res;
            });
        }
    },
    error: function(e) {
        console.log(e)
    },
});
```

上面实现了读取角色名称以及该角色所有台词的数据加载，通常情况下是没有问题的，但是却发现控制台输出了一串字符串，也就是说发生了某种错误，但这个问题十分离奇，它本身并不是报异常，而是正常的返回状态码 200，并且控制台输出了：

> {readyState: 4, getResponseHeader: ƒ, getAllResponseHeaders: ƒ, setRequestHeader: ƒ, overrideMimeType: ƒ, …}

这不是一次正常的请求结果吗？那为什么会不执行 success，反而去执行 error 了？

## 解决方法
网上搜了一下相关的问题，有的人说是要加上：

```
dataType: "json"
```

用于严格的判断返回数据类型，可是加了也没用，显然不是这个原因。
真正的原因其实是因为 JSON 文件加了注释导致的，`name.json` 内容如下：

```
{
    // NPC名称
    "test": "测试NPC",
    "pangbai": "旁白",
    "suoer": "索尔",
    "lawei": "拉薇",

    // 敌人名称
    "dushe": "毒蛇",
    "tuntianmang": "吞天蟒"
}
```

只要把注释去掉就行了，对于 Ajax 请求，返回的数据必须是严格的 JSON 数据，不能加注释。

非要加注释的话，就设置成一个字段：

```
{
    "remark_1": "------------------角色的名称------------------",
    "test": "测试NPC",
    "pangbai": "旁白",
    "suoer": "索尔",
    "lawei": "拉薇",

    "remark_2": "------------------敌人的名称------------------",
    "dushe": "毒蛇",
    "tuntianmang": "吞天蟒"
}
```

只加入几行字段而已，无伤大雅，也不会影响什么效率之类的东西。