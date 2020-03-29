---
title: 学习科学上网
date: 2017-03-23 22:26:53
categories:
- 科学上网
tags:
- 总结
- 工作
---
经过一番痛苦的探索，目前觉得比较靠谱的科学上网的方法，作为笔记总结下~
<!-- more -->
_不过这个并非对非程序出身的人的教程，详细学习可以参考最后的《SS不完全指南》一文_

> 今天经过一番探索，对shadowsocks的使用方式有了一定理解，虽然没到用代码控制那一层，不过对于平时应用足矣
> 核心是使用ShadowSocks来进行网络代理，但具体的服务器可以有某些**分享网址**上可用，寻找适合的服务器就好，大部分免费的额度已经够了，不过随着时间的发展，或许需要部分资金支持。

### 下载ShadowSocks软件

现在这个软件的直接链接已经被屏蔽了，目前有两种方式：

- github地址[shadowsocks-windows](https://github.com/shadowsocks/shadowsocks-windows)
- 客户端官网的下载地址[比较老的版本](http://www.sshchina.net/) 2.x版本
- ishadow[下载地址](http://get.ishadow.website/) 3.x版本

下载后一般解压即可用，新老版本都支持的方式是conf服务器配置文件的方法，这个方法在各个服务器提供处都有涉及。其他方法简单看看就能上手，不过conf文件方法对于程序员是最直观的。

### 寻找Socks服务器源

- [分享网址一](http://51.ruyo.net/shadowsocks/)——不定期更新，不过不能用了可参考最后的不完全指南
- [分享网址二](https://www.atgfw.org/)——好像被屏蔽了，估计以后不能用

此次探索暂时以https://www.atgfw.org/为例子，下载服务器配置文件，放到软件同一目录下，右下角小飞机会自动读入配置信息，而后就可以安心科学上网了。
这个例子里面的VPN免费的流量还是不少，虽然整个交互真是作死，以后还是买一些指南里面说的付费稳定的才是上策，鉴于现在对国外流量需求不算大，先这么用着吧。

### 原理探究

由于是go语言写的，下面是一篇针对各级代理的原理性讲解：
- [GO Simple Tunnel](http://51.ruyo.net/p/3564.html)

下一篇是针对shadowSocks的详细探究和服务购买推荐
- [ShadowSocks不完全指南](http://www.auooo.com/2015/06/26/shadowsocks%EF%BC%88%E5%BD%B1%E6%A2%AD%EF%BC%89%E4%B8%8D%E5%AE%8C%E5%85%A8%E6%8C%87%E5%8D%97/#serverconfiguration)

*在此声明，这个是个人笔记，不作为分享~*