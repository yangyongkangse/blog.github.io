---
layout: post
title: "Xshell实现windows上传文件到linux主机"
subtitle: ""
date: 2020-08-18 09:51:10 +0800
author: "YangYongKang"
header-style: text
catalog: true
archive:
    - Xshell
    - Linux
tags:
    - Xshell
    - Linux
---

##### 配置文件传输协议
* 下载路径表示从Linux下载文件保存到windows的位置
* 加载路径表示Linux从windows处获取文件
* 上传协议选择Zmodem

![image.png](/img/linux.png)

##### 安装lrzsz工具包
lrzsz是一款在linux里可代替ftp上传和下载的程序，提供XMODEM、YMODEM和ZMODEM文件传输协议的unix通信包。
* sudo apt-get install lrzsz      -- ubuntu
* sudo yum -y install lrzsz       -- centos

##### 使用rz、sz命令进行文件互传
* rz：表示Linux端接收文件(r代表receive，z代表zmodem协议)。在终端输入rz，会弹出弹框然后选择文件，文件默认保存到Linux当前路径下。
* sz：表示Linux端发送文件给windows(s代表send，z代表zmodem协议)。在终端输入sz  文件名(中间空格)，会弹出弹框然后选择windows保存路径如果你之前未设置下载路径。

![image.png](/img/rz.png)
