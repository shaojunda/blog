---
title: 腾讯云轻量应用服务器 SSH 配置
tags:
  - ssh
  - 服务器
categories:
  - 环境搭建
date: 2020-11-25 00:57:53
keywords:
  - ssh
  - lighthouse
  - 腾讯云轻量服务器
description: 腾讯云轻量应用服务器 SSH 配置
---

今年双十一在腾讯云买了一台 1 核 2 G 3M 的轻量应用服务器，3 年 253 感觉做一些简单的服务和测试应该够用了。记录一下通过 SSH 连接轻量云服务器的步骤。

## 重制密码
1. 登陆服务控制台
2. 在服务器列表中找到相应的实例
3. 关机
4. 在「实例信息」栏中点击「重置密码」

## 配置密钥
1. 登录轻量应用服务器控制台，并单击左侧导航栏中的「密钥」
2. 在密钥列表页面，单击「创建密钥」
3. 在弹出的「创建SSH密钥」窗口中，设置密钥的所属地域，选择密钥的创建方式，单击「确定」
4. 在密钥列表选择要绑定的 ssh 密钥，单击「绑定/解绑实例」
5. 下载密钥文件到本地
<!--more-->
## 配置 ssh

修改 ssh 配置文件 `~/.ssh/config` 增加
```
Host light
    HostName 「公网 IP」
    port 22
    User 「用户名」
    IdentityFile 「密钥文件路径」
    TCPKeepAlive yes
    ForwardAgent yes
```

## 连接服务器
更新好 ssh 配置文件之后即可通过命令行登陆到服务器 `ssh light`

从购买到配置整体感觉比阿里云更容易上手，指引也更明确。
