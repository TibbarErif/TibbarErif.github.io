---
title: Github 提高 push 速度的方法
date: 2022-05-10 18:19:09
tags:
 - Github

categories:
 - 日常学习

---
## 前言
在使用 GitHub 托管代码的时候经常会提交失败或者速度超级慢，本文提供了一种解决此问题的方法。
## 报错问题
常见报错是 SSL 超时问题，报错信息如下：

```
LibreSSL SSL_read: error:02FFF03C:system library:func(4095):Operation timed out, errno 60
```

又或者是这样的报错：

```
Failed to connect to github.com port 443 after 75010 ms: Operation timed out
```

## 解决方法
网上提供的一般是下面这条命令：

```
git config --global http.sslVerify false
```

这条命令确实可以解决 SSL 证书认证的问题，即直接跳过了认证。
而提交速度慢的原因却不是因为证书，而是因为 GitHub 的主机是在国外，国内的访问速度很慢。

## push 超时的原因
为了解决这个问题要先科普一个网页开发的知识。当用户在浏览器的地址栏敲下一个地址并且进行访问的时候，首先需要将域名解析成 IP 地址，然后定向到 IP 地址所在的服务器，服务器根据用户的请求返回 HTML 文档，浏览器再解析文档变成可视化的页面 UI。

GitHub 提交慢的一个原因就在于把域名解析成 IP 的过程，解决方法就是我们自己手动设定 IP，跳过 DNS 解析环节。
## 解决解析问题
IP 查询：[https://www.ipaddress.com/](https://www.ipaddress.com/)
（在查询的时候需要翻墙，记得开启可以访问外网的 VPN）

分别查询下面这三个地址：

1、github.com
2、github.global.ssl.fastly.net
3、assets-cdn.github.com

查到地址之后，执行 `sudu vim /etc/hosts` 命令编辑本地的解析文件：

![hosts](https://img3.qq.tc/2022/05/10/imagef7d58c18db25e0f1.png)
（IP 地址要改成自己查询的结果，因为 CDN 服务器分布在全世界，每个人查到的可能都不同）

Window 系统则是找到 hosts 文件，再以管理员身份运行。
保存好之后再使用 push 命令就会直接推送了！
因为本地设定了 IP 地址就直接跳过了 DNS 解析的过程，速度直接起飞。