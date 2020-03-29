---
title: fiddler抓包工具(windows)
date: 2017-11-04 21:04:09
categories:
- 学习
tags:
- 工作
- fiddler
---
简介fiddler工具的原理和在电脑上使用步骤，作为个人笔记使用
<!-- more -->
在打开fiddler后，会在客户端和服务器之间自动增加一层127.0.0.1:8090的代理层，其表征为客户端的所有请求都要先经过Fiddler，然后转发到相应的服务器，反之，服务器端的所有响应，也都会先经过Fiddler然后发送到客户端。此时IE和chrome的浏览器相应代理设置将自动更改，其他浏览器要进行代理设置


默认情况下fiddler只能获取http请求，需在菜单项 Tools->Fiddler Options->HTTPS的选项卡中 CaptureHTTPS CONNECTs是默认勾选的，要手动勾选Decrypt HTTPS traffic和Ignore servercertificate errors两项（首次点击会弹出是否信任fiddler证书和安全提示，直接点击yes就行）

如果要启用手机端抓包，需要额外设置  Tools->Fiddler Options->Connections 中勾选allow remote computers to connect，并确认默认监听端口(8090),如果端口号有更改，需要重启生效，否则会出现无法上网的问题。对手机端的设置步骤如下：
1. 手机和电脑连接同一个网络，打开手机浏览器，输入http://ip:端口号(http://10.232.23.21:8090)
2. 在出现的Fiddler Echo Service页，点击最下面的FiddlerRootcertificate下载并安装此证书(需要自行重命名)
3. 如果手机没设置密码，可能还要设置一下，以通过证书的安装认证
4. 此时可以通过让手机和电脑在一个局域网中，手机连上相应wlan后，设置次wifi的代理，主机名为电脑的ip，端口为fiddler的代理端口，点击确定即可

如果需要停止代理，手机上直接重置wifi的代理，并在设置——安全——受信任的凭证——用户 里点击相关安装证书删除即可。

[参考链接](http://blog.csdn.net/gld824125233/article/details/52588275)