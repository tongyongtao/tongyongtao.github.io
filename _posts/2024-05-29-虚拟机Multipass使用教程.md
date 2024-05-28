---
layout:     post     # 使用的布局（不需要改）
title:      虚拟机Multipass使用教程
subtitle:   Multipass使用方式
date:       2024-05-29 # 时间
author:     Tong # 作者
header-img: img/post-bg-2015.jpg #这篇文章标题背景图片
catalog: true # 是否归档
tags: #标签
    - Multipass
    - 教程
typora-copy-images-to: ../assets/images
typora-root-url: ../../blog
---

### 1、账号设置与使用ssh连接虚拟机

multipass 默认会给所有实例生成名为“ubuntu”的账号，所以首先我们给ubuntu设置密码，输入以下命令然后输入我们要设置的密码。

```shell
sudo passwd ubuntu
```

然后设置 root 账户密码，输入如下命令后进行密码设置。

```shell
sudo passwd root
```

设置完root账号密码后通过 su root 命令切换到root账户下，进行root账号的ssh 连接权限进行配置。

````shell
su root
````

编辑 /etc/ssh/sshd_config 文件，运行：

```shell
sudo vi /etc/ssh/sshd_config
```

在打开的文件中，找到 找到 #Authentication，在其下面添加以下内容（允许root账号通过远程ssh进行连接）：

```shell
PermitRootLogin yes
passwordAuthentication yes
```

重启ssh 服务：

```shell
sudo service ssh restart
```

使用第三方ssh工具连接即可：

![image-20240527220811419](/assets/images/image-20240527220811419.png)



### 2、未完待续。。。
