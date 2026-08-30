---
title: 在本地部署Memos以及使用Cloudflare Zero Trust进行外部访问
tags:
  - Memos
  - Docker
  - 群晖
categories: 折腾
abbrlink: c99a4de6
date: 2023-04-01 23:08:50
---

　　Memos是一个轻量可以自己部署的碎片知识管理系统，有点类似于flomo。支持多标签、多用户、搜索、自定存储等功能。

# 在 Docker 中部署

　　本文使用Docker compose方式部署；

```yaml
version: "3.0"
services:
  memos:
    image: neosmemo/memos:latest
    container_name: memos
    volumes:
      - /volume1/docker/memos:/var/opt/memos # 持久化路径:容器路径
    ports:
      - 5254:5230                            # 外部端口:容器端口
```

　　现在你可以运行`docker-compose up -d`来启动容器了，访问 http://localhost:5254/ 就可以访问memos的主页创建账户了。

<!--more-->

# 在 Cloudflare 开通 Zero Trust 并且创建隧道

　　在群辉套件中心添加套件来源 https://packages.synocommunity.com/，并且安装套件 Cloudflared。

![image-20230401234057185](https://s2.loli.net/2023/04/03/Ljk8oJGEbI9aWKv.png)

![image-20230403024320410](https://s2.loli.net/2023/04/03/Kza24VZYS73y8il.png)

　　访问 https://dash.cloudflare.com/ 并且登录自己的账户，转到 Zero Trust → Access → Tunnels → Create a tunnel，复制自己 Tunnel token，启动群晖的套件，粘贴该 token。

　　转到 Public Hostname ，然后 Add a public hostname ，添加自己域名和主机。

![image-20230403025156437](https://s2.loli.net/2023/04/03/hG8Oa97ZxrbSRg4.png)

　　保存设置后网站就可以访问了。
