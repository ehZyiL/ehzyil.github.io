---
title: 自建rss聚合阅读工具
author: ehzyil
tags:
  - rss
  - render
categories:
  - 记录
date: 2023-10-22
---

自建rss聚合阅读工具，极其轻量化配合Render⚡️一键部署，还能绑定自定义域名
<!-- more -->



## 前置准备

1.[一个Render账号](https://render.com/)
Render是一款可以白嫖的Paas提供商，不需要绑卡什么的，可以白嫖😄😄😄。

2.[一个GitHub账号](https://github.com/)

GitHub是一个可以保存自己代码的地方，代码托管。

3.[一个托管了域名的CloudFlare 账号](https://www.cloudflare.com/zh-cn/)(可选)

CloudFlare 是一家全球知名的CDN服务商，并且提供了免费的 CDN 套餐，还不限流量，所以我们完全不需要花一分钱就能使用它的 CDN 服务。

## 开始搭建

### Fork源代码到自己的仓库

项目名字：rss-reader

https://github.com/srcrs/rss-reader

进入上方链接进入该源码的Github仓库

![fe342c75935f0d5d0de29.png](https://i3.wp.com/telegra.ph/file/fe342c75935f0d5d0de29.png)Fork到自己仓库。

因为当前的代码配置完全是原作者的，我们可以配置成自己喜欢的rss。具体如何修改[README.md](https://github.com/srcrs/rss-reader#readme)中都有教程自己配置。

修改方式有两种，可以自己把代码拉到本地修改完再commit或者直接在自己的代码库修改。

### 使用Render进行一键部署

进入Render的[Dashboard](https://dashboard.render.com/)点击New，如下图所示

![7ed6dc85eacb8179a6a9a.png](https://i3.wp.com/telegra.ph/file/7ed6dc85eacb8179a6a9a.png)





点击Web Service

![c6fcf2aca98fba40f5ae4.png](https://i3.wp.com/telegra.ph/file/c6fcf2aca98fba40f5ae4.png)

绑定自己的Github后操作如下：

![44b88d10ab4db9eb4fe21.png](https://i3.wp.com/telegra.ph/file/44b88d10ab4db9eb4fe21.png)

找到你要部署的分支：

![44b88d10ab4db9eb4fe21.png](https://i3.wp.com/telegra.ph/file/44b88d10ab4db9eb4fe21.png)

填写部署的项目名称和部署方式，Docker

![ed9a5dc07b70f98e39e8d.png](https://i3.wp.com/telegra.ph/file/ed9a5dc07b70f98e39e8d.png)

选择实例类型，方然是Free白嫖了

![042a4a5a90689d3bfc71e.png](https://i3.wp.com/telegra.ph/file/042a4a5a90689d3bfc71e.png)

点击最下方的`Create Web Service`等待部署完成即可.

![ca8cd99885668f1c9aa2b.png](https://i3.wp.com/telegra.ph/file/ca8cd99885668f1c9aa2b.png)

部署完成后可以点击蓝色连接即可访问。

至此该项目搭建完成。

## 自定义域名（可选）

Render免费提供部署好的项目的域名🌹，看着太长不好记或者不喜欢可以更改成自己的域名。

进入Render进行以下操作

![c2bc4f1771abe3981e563.png](https://i3.wp.com/telegra.ph/file/c2bc4f1771abe3981e563.png)

![0a879d1cc36fdfff91310.png](https://i3.wp.com/telegra.ph/file/0a879d1cc36fdfff91310.png)

然后登录CloudFlare去你所要绑定域名的界面进行以下操作，红色框里的内容由上图复制。

**红色框里的Type改成CNAME（懒得编辑图片了）**

![88602faa3a7b95068dd56.png](https://i3.wp.com/telegra.ph/file/88602faa3a7b95068dd56.png)

然后回到Render，点击Verify

![a5ed85bcce9734facdc5f.png](https://i3.wp.com/telegra.ph/file/a5ed85bcce9734facdc5f.png)

显示如下，域名就绑定成功了

![9e91ec321724c49b326c6.png](https://i3.wp.com/telegra.ph/file/9e91ec321724c49b326c6.png)

现在就可以从自己配置的域名访问了。
