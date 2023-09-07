---
title: Centos 使用 squid 作为 HTTP 代理服务器
date: 2023-04-02 15:08:34
tags:
 - 开发技术

categories:
 - 日常学习
---
## 前言
最近大火的 ChatGPT 让我感受到新一代的工业革命即将到来，站在历史的转折点无比兴奋！于是就开始折腾一个自己的 ChatGPT 服务，因此需要对接到 OpenAI 的接口，但是 OpenAI 对大陆地区以及香港、台湾、澳门都有限制，导致我们很难使用，经过一番折腾，搞了一台美国服务器，用美国 IP 以及自己的谷歌账号注册了一个，发现账户余额只剩下 5 美元了（原来是 18 美元）看来是玩坏了……但是不要紧，虽然吃不上第一口热乎的，但本着科研精神，还是可以研究一下这个接口的。

同时为了潜心研究 AI 技术，特意买了个新域名：[huotutu.com](http://huotutu.com)，最新搭建的 ChatGPT 网页版已经实装了，注册即可在线体验。作为自己的第一个创业项目，也会适当的进行一些收费，毕竟购买美国服务器以及充值 ChatGPT 就是一笔不小的开支。

好了，接下来进入正题。

## squid 是什么？
这是一个可以让服务器变成代理服务器的东东，至于为什么要这东西，因为 ChatGPT 国内大陆地区不能访问接口，但是香港服务器可以，因此需要借助一台中间服务器代理访问 ChatGPT 的接口，而我尝试去搜了一下网上免费的代理 IP，发现全部都是假的，所以决定自己搞一台。

> 聪明的小伙伴可能会问，既然你都搞两台服务器了，为什么不直接部署在美国服务器或者香港上面？

这是因为我的域名是在国内备案过的，使用的是国内阿里云服务器，在国内延迟极低，可以做到秒开网页，延迟在 50ms 左右，而如果部署到香港或者美国，延迟都是 200ms 以上的，众所周知，影响网页体验的重要因素是打开速度，这么高的延迟明显不合适。

## 安装 squid
直接使用 yum 命令即可安装，但是需要先安装 openssl：

```
yum install -y openssl
yum install -y squid
```

以上就完成了 squid 的安装，接下来需要启动服务，并且将其设置为开机启动项目：

```
sudo systemctl start squid
sudo systemctl enable squid
```

通过以下命令可以查看运行状态：

```
sudo systemctl status squid
```

默认的端口是 3128，我们可以通过修改配置文件改变端口：

```
vim /etc/squid/squid.conf
```

找到下面一行，我将其改成了 3999：
```
# Squid normally listens to port 3128
http_port 3999
```

接着，重启一下服务：

```
sudo systemctl restart squid
```

查询本机所有端口：

```
netstat -ntlp
```

可以看到 3999 端口信息：

```
tcp6       0      0 :::3999                 :::*                    LISTEN      12341/(squid-1)     
```

但是这里显示的是 `:::3999`，表示只允许本地访问，本地测试如下：

```
curl -x 127.0.0.1:3999 https://api.openai.com/v1/chat/completions
```

因为安全性问题，默认只允许本地访问，而我们需要用其他服务器来访问这个端口，因此继续编辑配置文件：

```
vim /etc/squid/squid.conf
```

在配置文件加入两行：

```
acl myclient src 175.xxx.111.111
http_access allow myclient
```

> 如果要允许所有 IP 访问，可以添加一行 http_access allow all，但这种做法存在巨大风险，可能被他人利用。

`myclient` 是自己定义的一个字符，`src` 后面跟上你另一台服务器的 IP 地址，然后再设置 `http_access` 允许 `myclient` 这个自定义参数，这样就大功告成了，再重新启动一下服务器：

```
sudo systemctl restart squid
```

在 PHP 用 Guzzle 试一下：

```
$client = new Client();
$url = 'https://yousitedomain';

try {
    $res = $client->get($url, [
        'proxy' => '175.xxx.111.111:3999',
        'connect_timeout' => 10,
    ])->getBody()->getContents();
} catch (\Throwable $exception) {
    $res = $exception->getMessage();
}

var_dump($res);
```

然后去看看自己另一台服务器上面 nginx 的访问日志即可，可以看到，这个时候使用的是代理的那台服务器的 IP。

然鹅，事情没有这么简单！在尝试调用 ChatGPT 接口却报错：

```
curl: (52) Empty reply from server
```

尝试在本地运行：

```
curl -vx 175.xxx.111.111:3999 https://api.openai.com/v1/chat/completions
```

加上 `v` 参数，可以列举出详细的请求信息，找到最下面的信息：

```
Recv failure: Connection reset by peer
* OpenSSL SSL_read: Connection reset by peer, errno 54
* Failed receiving HTTP2 data
* Send failure: Broken pipe
* OpenSSL SSL_write: Broken pipe, errno 32
* Failed sending HTTP2 data
```

可以看到报错了：

```
OpenSSL SSL_read: Connection reset by peer, errno 54
```

初步判断应该是 ssl 证书的什么问题，连接被重置了，可能是 ChatGPT 追踪了来源……最后还是老老实实用香港服务器了（悲）。