---
title: Chrome反复提示拒绝连接解决方法
tags:
  - Chrome
  - 网络
categories:
  - 日常
author: DuCheng
date: 2023-02-07 00:20:00
---

最近用 chrome 老是时不时抽风，显示无法访问 Internet。一开始用火绒的断网修复，说是 hosts 文件有问题，全部注释掉了之后确实好了一段时间，结果今天又出问题了。

后来百度了一圈，发现只有小米路由器有这种问题

## 解决方案

1. 打开 chrome`设置->隐私设置和安全性->安全`

2. 在高级中打开`使用安全DNS`

3. 将 DNS 自定义为`https://dns.alidns.com/dns-query`，问题暂时得到解决
