---
title: Javascript 获取 data 自定义数据时的注意事项
date: 2021-10-02 10:07:53
tags:
 - Javascript

categories:
 - 日常学习
---

## 前言
在 HTML 开发中，给标签加上 `data-xxx` 属性是很常见的一种做法，例如 `data-id="1"`，这样就可以很方便的取得 id 的值了。今天在给之前做的剧情编辑器添加“插入”功能时，突然踩了一个坑，虽然是一个很低级的错误……

![剧情编辑器新增插入数据功能](https://pic.imgdb.cn/item/6157c1342ab3f51d9179f5c3.jpg)

## 数组任意插入元素
数据是一个数组，在 JavaScript 想要往数组中间插入数据可以使用 `splice` 这个系统内置的方法：

```
let arr = ["aa", "bb", "dd"];
arr.splice(2, 0, "cc");

console.log(arr);
```

输出结果：

```
(4) ['aa', 'bb', 'cc', 'dd']
```

这个方法只要获取到需要插入的位置，即数组的下标索引就可以在该位置插入一个新元素。

## 踩坑过程
动态生成一个列表，有这样一个 a 标签用来执行插入事件：

```
<a class="insert-event" data-id="0" href="javascript:void(0);">插入</a>
```

`data-id` 用来存储数组的下标。

由于是动态生成的元素，因此不能用常规的 `on` 来绑定监听，否则会监听不到点击事件。
需要绑定父级标签，然后再监听子标签的点击事件：

```
$("#preview").on('click', '.insert-event', (e) => {
    let insertIndex = e.target.dataset.id;
});
```

绑定了监听事件之后就可以获取当前点击事件上面的属性了。

```
let insertIndex = e.target.dataset.id;
```

上面的方法即获取到标签上面的自定义属性 `data-id`。

接下来只要计算新的位置插入元素即可：

```
let newIndex = insertIndex + 1;
console.log("插入位置：" + newIndex);

commands.splice(newIndex, 0, data);
```

看上去没什么问题？
实际上却错了，这里的 `insertIndex + 1` 会被当做字符串相加，即变成了字符串连接操作……

正确的做法应该是在计算时强制转换为整型变量：

```
let newIndex = parseInt(insertIndex) + 1;
console.log("插入位置：" + newIndex);

commands.splice(newIndex, 0, data);
```

## 总结
在 HTML 标签中自定义属性都会被当做字符串，如果是整型数据则需要手动转换才能参与计算。