---
title: Github 遇到 443 报错的解决方法
date: 2023-09-07 19:17:34
tags:
 - Github

categories:
 - 日常学习
---
## 前言
由于某些不可名状的原因，在提交到 github 的时候会遇到报错：

```
Failed to connect to github.com port 443
```

而如果我们本地有开启了代理（fanqiang）但却还是报这个错误，说明提交的时候没有走代理，这个时候需要设置 git 的代理才可以。

> 不可名状的原因：Github 间歇性被墙，因此有时能正常提交，有时候就不行了。

## 方法
以 shadowsockets 为例，如下图所示，选择偏好设置。

![iShot_2023-09-07_19.22.06.jpg](https://s2.loli.net/2023/09/07/tFPdNr9csQknzZo.jpg)

接着选择 HTTP，然后将“开启HTTP代理”，“全局模式时，在系统代理设置中设置HTTP代理服务器”一起勾选。

![iShot_2023-09-07_19.20.29.jpg](https://s2.loli.net/2023/09/07/vLurOifRqcnBmda.jpg)

注意上面的 HTTP 代理服务器端口号码。

接着设置 git 全局代理，以我的为例，上面的 HTTP 代理端口号码是 1087，因此命令如下：

```
# http
git config --global http.proxy http://127.0.0.1:1087

# https
git config --global https.proxy http://127.0.0.1:1087
```

加上 `--global` 参数即代表全局，这个参数可以去掉，去掉的话就变成单独为当前仓库设置代理，有的时候我们会用国内的 coding 或者 gitee，如果用代理反而会让提交速度变慢，只有提交到 github.com 才需要用到代理。

这个时候再 push 就不会遇到 443 错误了，如果需要取消全局代理呢？

用下面的命令：

```
git config --global --unset http.proxy
git config --global --unset https.proxy
```
