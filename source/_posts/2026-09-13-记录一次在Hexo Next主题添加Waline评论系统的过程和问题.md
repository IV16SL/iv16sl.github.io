---
title: 记录一次在Hexo Next主题添加Waline评论系统的过程和问题
tags:
  - Hexo
  - Next
  - Waline
categories: Blog
abbrlink: 6fa8453b
date: 2026-09-13 18:38:53
---

　　周日高强度上网冲浪，看到不少博客都有评论系统，就萌发了自己博客也要弄一个的想法，于是说干就干，开始上网找。
　　一开始也不知道要找什么，便先从Next主题官网阅读相关文档开始，在 [Comment System](https://theme-next.js.org/docs/third-party-services/comments) 页还有导航页 [Plugins](https://theme-next.js.org/plugins/) 内都找到了相关信息，良久后觉得Valine还挺适合，但是好像停更了，还有一些其他问题。最后一番摸索找到了Waline，查了下比较简约，可以在Vercel部署，那不就非常适合我吗？开干！

<!--more-->

## 在Vercel上部署服务端

> 直接参考[官方文档](https://waline.js.org/guide/deploy/vercel.html) Vercel部署Waline

　　部署完本体后，接着按照文档在Neon上创建数据库，完成后记得在左侧菜单中SQL Editor中用官方仓库的 [waline.pgsql](https://github.com/walinejs/waline/blob/main/assets/waline.pgsql) 创建初始表，不少人少看了这一步，不建表后面会报错。
　　完成后需要回到Vercel中Waline的项目中Redeploy，但是这里我在后面遇到了第一个问题，就是时区不对影响了评论的时间，每条都晚8小时，默认用的是GMT时间。比较不幸的是Vercel的TZ环境变量是[保留变量](https://vercel.com/docs/environment-variables/reserved-environment-variables)，用户不可以更改，因此我们改用另外一种办法。
　　在Vercel创建项目时，已经替你在GitHub上Fork官方的仓库，打开仓库中的**index.cjs**，在最前面加入`process.env.TZ = 'Asia/Shanghai';`，然后提交，一定要第一行，然后回到项目执行Redeploy，一般这时候因为你的仓库有新提交，Vercel那里也自动Redelopy了，完成后时区就是GMT+8了。
　　最后绑定自己的域名，然后访问 https://yourwalineaddress/ui 注册管理员账号，并尽快开启两步验证保证安全性。

## 在Hexo Next上安装插件和设置

　　转到你博客的根目录下，安装@waline/hexo-next插件。

```
cd yourblog
npm install @waline/hexo-next --save
```

　　安装完成后在博客根目录下的配置文件`_config.yml`中增加以下设置：

```
# Waline 评论系统start https://www.npmjs.com/package/@waline/hexo-next
waline:
  enable: true
  serverURL: https://yourwalineaddress

  # Waline library CDN url, you can set this to your preferred CDN
  # libUrl: https://unpkg.com/@waline/client@v3/dist/waline.umd.js

  # Waline CSS styles CDN url, you can set this to your preferred CDN
  cssUrl: https://unpkg.com/@waline/client@v3/dist/waline.css

  # Custom locales 自定义区域设置
  # locale:
  #  placeholder:

  # 如果为false，评论数将只显示在文章页面，而不是在主页
  commentCount: false

  # 浏览量统计，注意：您不应该同时启用`waline.pageview`和`leancloud_visitors`。 leancloud_visitors在主题文件夹中
  pageview: false

  lang: zh-CN
  search: false #禁用表情包搜索
  reaction: false #文章反应
  imageUploader: false #图片上传

  # Cloudflare turnstile设置，评论区验证
  TURNSTILE_KEY: #Cloudflare turnstile key
  TURNSTILE_SECRET: #Cloudflare turnstile secret

  # Custom emoji
  # emoji:
  #   - https://unpkg.com/@waline/emojis@1.1.0/weibo
  #   - https://unpkg.com/@waline/emojis@1.1.0/alus
  #   - https://unpkg.com/@waline/emojis@1.1.0/bilibili
  #   - https://unpkg.com/@waline/emojis@1.1.0/qq
  #   - https://unpkg.com/@waline/emojis@1.1.0/tieba
  #   - https://unpkg.com/@waline/emojis@1.1.0/tw-emoji

  # Comment information, valid meta are nick, mail and link
  # meta:
  #   - nick
  #   - mail
  #   - link

  # Set required meta field, e.g.: [nick] | [nick, mail] 未登录必填字段
  requiredMeta:
    - nick

  # Word limit, no limit when setting to 0
  # wordLimit: 0

  # Whether enable login, can choose from 'enable', 'disable' and 'force'
  # login: enable

  # comment per page
  # pageSize: 10
# Waline 评论系统 end
```

有关于Cloudflare turnstile的设置，需要登录后在Dashboard左侧Application Security中Turnstile申请，请妥善保存Secret。之后前往Vercel项目中Environment Variables，新增`TURNSTILE_KEY`和`TURNSTILE_SECRET`两个变量，类型均为Secret，填入在Cloudflare申请到的值。

## Hexo博客重新生成

　　以上都完成后，在你的博客根目录下运行清理、重新构建、开启本地服务器预览一下效果：

```
hexo clean
hexo g
hexo s
```

　　发一下评论看下时间和效果是否都正常，如果一切都正常的，我遇到了后面一个问题，深色模式不生效的问题。

## 解决Waline在Next深色主题下没有生效的问题

　　正常情况下，Next主题就像[官方文档](https://waline.js.org/guide/features/style.html)所说的，使用`@media` 选择器通过 `prefers-color-scheme` 来根据设备颜色模式状态自动切换，我也没有去改动，理论上你只要在配置中加入`dark: 'auto'`就可以生效了，但是我实际遇到的是集中方法都试过了，完全没有效果，我也不知道为什么，始终有个背景色被劫持了。
　　最后不得以采用直接修改配置中引用的waline.css，增加如下代码，可自行浏览器Inspect按需修改：

```
@media (prefers-color-scheme: dark) {

  :root {

    --waline-bg-color: #333333 !important;

    --waline-color: #eeeeee !important;

    --waline-border-color: #eeeeee !important;

    --waline-disable-color: #eeeeee !important;

    --waline-info-color: #eeeeee !important;

  }



  .wl-editor,

  .wl-editor:focus,

  .wl-editor:active,

  .wl-input,

  .wl-input:focus {

    background-color: #333333 !important;

    background: #333333 !important;

    color: #eeeeee !important;

    border-color: #eeeeee !important;

  }

  .wl-btn:disabled {

    background: #3eaf64 !important;

  }

  .wl-card .wl-meta > span {

    background: #666 !important;

  }
}
```

　　浅色模式效果：
![浅色模式效果](https://i.iiii.im/file/47AQrgLM.png)
　　深色模式效果：
![深色模式效果](https://i.iiii.im/file/qKK5RlYV.png)

　　总的来说，整体效果我还是挺满意的，这套系统也很易用。
