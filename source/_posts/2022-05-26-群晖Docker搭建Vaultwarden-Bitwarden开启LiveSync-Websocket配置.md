---
title: 群晖Docker搭建Vaultwarden/Bitwarden开启LiveSync Websocket配置
tags:
  - Bitwarden
  - Vaultwarden
  - Websocket
  - 群晖
  - Docker
categories: 折腾
abbrlink: bc73beb2
date: 2022-05-26 21:27:42
---

　　自建Bitwarden是目前自托管密码多平台方案相当热门的方案，在网络上也有非常多的教程供参考，不过许多人搭建后都提到过Bitwarden的数据同步很多时候不是实时的，一般需要手动同步或者向下滑动来立刻同步，否则经常会有你在这个客户端建立好密码档案然而并没有在另一个客户端立刻显示。

　　对于不同客户端之间的实时同步，Bitwarden官方把这个功能叫做Live Sync，官方的博客中说这是基于Websocket技术来达成的功能；

> Live sync works by using a powerful technology called [*WebSockets*](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API). Live sync is now available in all Bitwarden apps, including mobile (mobile uses a different technology called *Push Notifications*).

　　在移动平台端，这个功能被叫做Push notification；

　　目前通常自建的Bitwarden大多使用的是Rust重构的开源项目Vaultwarden，这个项目也是支持开启Live Sync的，不过网上教程提到的非常少，因为我在这里整理说明下，一下是在群晖Docker上搭建后开启Live Sync的操作，其他平台和搭建方法达成的也都大同小异，自己按需修改。

<!--more-->

# 建立反向代理配置文件

　　建立一个反向代理配置文件，此文件基于群晖自带的反向代理配置修改，如果你已经使用过群晖的反向代理创建过配置，则该配置会保存在：

```
DSM6.X  = /etc/nginx/app.d/server.ReverseProxy.conf
DSM7.X  = /etc/nginx/sites-enabled/server.ReverseProxy.conf
```

　　文件在这里命名为"bitwarden.conf"，内容如下：

```
server {
    listen 5680 ssl http2;
    listen [::]:5680 ssl http2;

    server_name Bitwarden;

    include /usr/syno/etc/www/certificate/ReverseProxy_46af64d7-6523-4725-8c3f-88750cad6eb1/cert.conf*;
    include /usr/syno/etc/security-profile/tls-profile/config/ReverseProxy_46af64d7-6523-4725-8c3f-88750cad6eb1.conf*;
    add_header Strict-Transport-Security "max-age=15768000; includeSubdomains; preload" always;
	proxy_ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;

    location / {
        proxy_connect_timeout 15;
        proxy_read_timeout 15;
        proxy_send_timeout 15;
        proxy_intercept_errors off;
        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://localhost:5678;
    }
    location /notifications/hub {
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $http_connection;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_pass http://localhost:5677;
    }

    location /notifications/hub/negotiate {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://localhost:5678;
    }

    error_page 403 404 500 502 503 504 /dsm_error_page;

    location /dsm_error_page {
        internal;
        root /usr/syno/share/nginx;
        rewrite (.*) /error.html break;
        allow all;
    }
}
```

　　上侧配置中，端口5680是最终的访问端口，5678是容器映射的端口，5677是Websocket的端口。

# 将配置文件上传至群晖nginx对应配置目录并重启服务

　　将"bitwarden.conf"，上传至群晖，右键菜单属性中获取实际路径，复制到上面提到的DSM6.X或者DSM7.X反向代理配置文件目录；以下是基于DSM7.X演示：

![](https://s2.loli.net/2022/05/26/I5nRZCfkMTra7dY.png)

```
cp /volume1/docker/scripts/bitwarden.conf /etc/nginx/sites-enabled/bitwarden.conf
synow3tool --gen-all && systemctl reload nginx
```

# 测试效果

 　　2个客户端，一个是Windows客户端，一个是安卓客户端；修改一个项目确认后查看是否实时同步，控制台也同时会有日志产生。
![](https://s2.loli.net/2022/05/26/iCBWhMtd9Jmye6F.gif)

# 参考链接
[1]: https://bitwarden.com/blog/live-sync/	"LiveSyncBitwardenApps"
[2]: https://github.com/andyzhshg/syno-acme/issues/66
[3]: https://vaultwarden.discourse.group/t/need-explaining-websocket-and-push-notifications/87/9
[4]: https://gist.github.com/nstanke/3949ae1c4706854d8f166d1fb3dadc81
[5]: https://gist.github.com/eizedev/06a6727dc341745a4845fe04ccc97b05
[6]: https://github.com/dani-garcia/vaultwarden/wiki/Enabling-WebSocket-notifications

