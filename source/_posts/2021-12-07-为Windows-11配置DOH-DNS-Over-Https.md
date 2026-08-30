---
title: 为Windows 11配置DOH(DNS Over Https)
categories: 折腾
tags:
  - Windows 11
  - DNS
  - DOH
abbrlink: 58192ec3
date: 2021-12-07 09:22:58
---

Windows 11在22000.51这个版本开始支持DND Over Https(下文简称DOH)，在此之前已经有不少浏览器已经支持DOH，例如Google Chrome和Mozilla Firefox。

DOH的出现意味着DNS客户端和DNS服务器之间的查询将通过安全的https连接而不再是纯文本，不论隐私还是安全都能够获得有效的提升，尽管这些在当下互联网中都是相对的。

<!--more-->

# Windows 11内置的支持DOH的DNS

为Windows 11配置DOH并不复杂，但是默认情况下，你只能选择内置的DNS服务商；

你可以使用下面这个Powershell命令来列出系统当前内置的DNS服务商：

```powershell
Get-DNSClientDohServerAddress
```

```
ServerAddress        AllowFallbackToUdp AutoUpgrade DohTemplate
-------------        ------------------ ----------- -----------
149.112.112.112      False              False       https://dns.quad9.net/dns-query
9.9.9.9              False              False       https://dns.quad9.net/dns-query
8.8.8.8              False              False       https://dns.google/dns-query
8.8.4.4              False              False       https://dns.google/dns-query
1.1.1.1              False              False       https://cloudflare-dns.com/dns-query
1.0.0.1              False              False       https://cloudflare-dns.com/dns-query
2001:4860:4860::8844 False              False       https://dns.google/dns-query
2001:4860:4860::8888 False              False       https://dns.google/dns-query
2606:4700:4700::1001 False              False       https://cloudflare-dns.com/dns-query
2606:4700:4700::1111 False              False       https://cloudflare-dns.com/dns-query
2620:fe::fe          False              False       https://dns.quad9.net/dns-query
2620:fe::fe:9        False              False       https://dns.quad9.net/dns-query
```

# 添加自定义DNS

如果你希望使用的DNS并不在此列表中，你也可以通过下面的Powershell命令来添加你自己喜欢的DNS；

```powershell
Add-DnsClientDohServerAddress -ServerAddress '<resolver-IP-address>' -DohTemplate '<resolver-DoH-template>' -AllowFallbackToUdp $False -AutoUpgrade $True
```

例如：

```powershell
Add-DnsClientDohServerAddress -ServerAddress '223.5.5.5' -DohTemplate 'https://dns.alidns.com/dns-query' -AllowFallbackToUdp $False -AutoUpgrade $True
```

# 为当前网络设置DOH

- "Windows 11设置"，选择"网络和Internet"；
- 选择"以太网"，编辑"DNS服务器分配"；
- 编辑DNS设置为手动，键入你的DNS IP地址；
- 将首选的DNS加密设置为"仅加密(通过HTTPS的DNS)"，保存。

以上设定也将作用于你的WLAN或者Wi-Fi网络。

# 三种模式的区别

**仅未加密**：默认选项，到指定DNS服务器的查询全部未加密，通过传统的纯文本查询；

**仅加密（通过HTTPS的DNS）**：所有DNS查询将通过HTTPS传输来获得最有效的保护，但是如果指定DNS服务器如果不支持DOH查询，则不会进行DNS解析；

**已加密的首选，未加密的允许**：将邮箱尝试通过DOH查询，如果目标DNS服务器不支持DNS则改为纯文本查询以获得最佳兼容性。
