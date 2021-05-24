---
title: ssh 使用 pem 文件登录远程服务器
date: 2021-05-23 23:45:52
tags:
  - ssh
  - 服务器
keywords:
  - ssh
  - pem
description: 使用 pem 文件登录远程服务器
---

今天尝试了一些 AWS 的 Lightsail 发现它的 ssh 是需要使用 pem 文件进行远程登录的，记录一下登录方式。
以我申请到的账号为例：
* 用户名: ubuntu
* IP: 192.168.1.101 （举例）
* pem name: LightsailDefaultKey-ap-northeast-1.pem

### 执行命令
* `ssh -i LightsailDefaultKey-ap-northeast-1.pem ubuntu@192.168.1.101`
<!--more-->
执行后报错如下：
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for 'LightsailDefaultKey-ap-northeast-1.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "LightsailDefaultKey-ap-northeast-1.pem": bad permissions
ubuntu@192.168.1.101: Permission denied (publickey).
```

* 执行 `chmod 0600 LightsailDefaultKey-ap-northeast-1.pem` 后上述报错会消失

* 执行 `ssh-add -K LightsailDefaultKey-ap-northeast-1.pem`

之后就可以通过 `ssh ubuntu@192.168.1.101` 登录到远程服务器了。


