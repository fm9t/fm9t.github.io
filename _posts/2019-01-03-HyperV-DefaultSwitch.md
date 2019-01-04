---
layout:     post
title:      使用机器名访问HyperV虚拟机
subtitle:   使用机器名访问HyperV虚拟机
date:       2019-01-03
author:     BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - HyperV
    - Remote Desk
---
# 使用机器名访问HyperV虚拟机
HyperV创建虚拟机后，如果使用远程桌面访问，有时会出现能使用IP地址访问，不能使用机器名访问的情况，针对使用Default Switch网络的可以使用以下方式解决，首先在虚拟机器将网络设置为私有，然后按以下方式处理：

### 1.在网络和共享中心中打开网络适配器Default Switch：
![Default Switch状态](https://fm9t.github.io/img/blogimg/20190103001.jpg)

### 2.点击属性按钮进入Default Switch属性：
![Default Switch属性](https://fm9t.github.io/img/blogimg/20190103002.jpg)

### 3.打开Internet协议版本4的属性：
![Internet协议版本4属性](https://fm9t.github.io/img/blogimg/20190103003.jpg)

### 4.在高级中修改DNS，将mshome.net添加到附加这些DNS后缀里：
![DNS](https://fm9t.github.io/img/blogimg/20190103004.jpg)