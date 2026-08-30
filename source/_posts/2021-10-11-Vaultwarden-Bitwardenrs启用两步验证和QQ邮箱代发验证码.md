---
title: Vaultwarden/Bitwardenrs启用两步验证和QQ邮箱代发验证码
categories: 折腾
tags:
  - Bitwarden
  - QQ邮箱
  - 两部验证
abbrlink: c7341afb
date: 2021-10-11 22:16:04
---



在使用密码托管服务时，我最初只需要简单的同步，能够供手机使用即可。所以在使用1Password的时候，一直使用的是单机版。随着时间的推移，自己也有多平台需要，这种方式渐渐变的不方便，由于不太想使用1Password的网络服务，在前个月把密码托管逐步迁移到了自建Bitwarden服务。经过将近2个月的使用，个人感受相当良好。虽然有些小问题，但都在可以接受的范围内。

因为有多平台和外网需要，所以便开放了外网，因为安全考虑设置了2步验证并且关闭注册，本文主要说一下开放2步验证以及备用手段即QQ邮箱接收验证码。

<!-- more -->

# 搭建环境

我的Bitwarden服务通过Docker搭建在群晖上，虽然这样做有人觉得会存在不少问题，但我觉得人数不多且做好安全措施的话，可以接受这种方法。

# 设置两步验证

- 登录你的Bitwarden密码库网页后台；

- 点击右上角头像→我的账户→两部登录；

- 点击管理验证器应用，准备好Google Authenticator或者Microsoft Authenticator，根据自己的喜好来；

- 使用验证器扫描二维码→填入6位验证码→确定以及保存好恢复代码；

  > Bitwarden的恢复代码在设置里可以再次查看，请妥善保管。

# 设置邮箱服务以及代发验证码

我一开始觉得单纯的验证器已经够用，但是我遇到了TOTP服务器时间不正确的问题，一旦客户端和服务端时间差异超过1分钟，你的请求就是被服务端拒绝。保险起见我又多设置了邮箱服务，邮件验证码的有效期10分钟，是个很好的备选方案。

通过域名加上"/admin"访问Bitwarden的管理后台，输入admin token登录；

> Admin token在Dockers面板里设置，增加一条环境变量"ADMIN_TOKEN=xxx",xxx表示任意长度字符串，别太短就好，后期长时间不用就把这条删掉以保证安全性。

点击"SMTP Email Settings",进行如下设置：

```
Enable: true
Host: smtp.qq.com
Enable Secure SMTP :true
Force TLS :true
Port :465
From Address :user@qq.com #user就是你的用户名,这里不要用不是当前账户的地址，否则可能发送失败
From Name :Bitwarden
Username :user@qq.com #user就是你的用户名
Password :yourpassword #登录QQ邮箱Web页面，账户POP3设置里面生成授权码，需要关闭安全登录才会显示此选项
SMTP Auth mechanism :Plain
SMTP connection timeout :15
```

随后可以测试发送邮件是否成功，并且回到Bitwarden密码库网络后台的两部登陆中开启电子邮件验证码功能。

# TOTP服务端时间不同步

我也不知道为何会产生此问题，我最后把群晖的NTP服务器改成了"time.nist.gov"，之后一段时间都暂未遇到此问题。
