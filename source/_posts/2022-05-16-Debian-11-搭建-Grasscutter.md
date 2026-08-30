---
title: Debian 11 搭建 Grasscutter
tags:
  - 原神
  - Genshin Impact
  - Grasscutter
categories: 折腾
abbrlink: 85f3d684
date: 2022-05-16 22:12:49
---

Grasscutter是一个动漫游戏的服务端，本文仅供学习。

# Debian 11的安装

这一部分自行解决。

# 部署Grasscutter

## 1. 安装MongoDB

```
su -
apt update
apt install gnupg
wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | apt-key add -
echo "deb http://repo.mongodb.org/apt/debian buster/mongodb-org/5.0 main" | tee /etc/apt/sources.list.d/mongodb-org-5.0.list
apt update
apt install mongodb-org
systemctl enable mongod --now
```

数据库文件位置：/var/lib/mongodb

日志文件位置：/var/log/mongodb

<!--more-->

## 2. 安装Open jdk 17

```
apt install -y openjdk-17-jdk
```

## 3. 拉取Grasscutter库（Dev分支）

```
git clone https://github.com/Grasscutters/Grasscutter -b development
cd Grasscutter
git clone https://github.com/Koko-boya/Grasscutter_Resources
mkdir resources
\cp -r Grasscutter_Resources/Resources/* resources
```

## 4. 构建

```
chmod +x gradlew
./gradlew jar
ls
```

构建完成后生成一个grasscutter-1.1.2.jar；

因为更新频繁建议重命名为commit编号便于识别和管理；

```
mv grasscutter-1.1.2.jar grasscutter-58df221.jar
```

## 5. 下载mitmproxy

```
wget https://snapshots.mitmproxy.org/8.0.0/mitmproxy-8.0.0-linux.tar.gz
tar zxvf mitmproxy-8.0.0-linux.tar.gz
nohup ./mitmdump -s proxy.py --ssl-insecure --listen-port 12345 --set block_global=false --set tls_version_client_min=UNBOUNDED
```

## 6. 运行服务端

```
java -jar grasscutter-58df221.jar
```

首次运行可能会报错，按Ctrl-C退出后，编辑生成的config.json，填写自己的IP地址或者域名还有端口号，注意不要冲突，然后重新运行。

## 7. 建立账户

```
//account create <用户名> <UID>
account create test 12345
```

其余命令可以参考项目Wiki。

## 8. 登录服务器

个人电脑可以使用系统自带的http代理，也可以使用Clash for windows。

## 9. 待补充（待续）

待补充。。。
