---
title: 在Windows上切换node版本的实践
date: 2017-03-23 22:27:43
categories:
- 软件
tags:
- 工作
---
因工作需要，下载了nvm-windows对node进行不同版本的切换来满足不同项目的需求，不料踩坑若干，分享如下
<!-- more -->
### 下载node切换软件

在MAC下有大名鼎鼎的nvm，网上有很多成熟的教程。windows下也有“著名”的[nvm-windows](https://github.com/coreybutler/nvm-windows/releases) 实打实来说，这个挺好用的，不过我的打开方式有点歪~  现在来看一下这个go语言写成的工具有哪些需要注意的地方。
项目的github链接为：[nvm-windows](https://github.com/coreybutler/nvm-windows/) 可以点击上一段的链接下载1.1.3版本的切换软件，如果更新了，那就要按照github中给出的最新文档来，这次有点费力就是吃了没看英文文档的亏。

### 卸载电脑上已有的NODEJS和全局安装包

重要的事儿本来该说三遍，这里只说一遍(管不着我~)，在控制面板中删除了nodejs后，一定要到`C:\Users\wanglixing\AppData\Roaming` 下搜罗下npm文件夹，这是nodejs的全局安装包的位置，打开最好截个图，从而得知之前安装了啥，很有可能各个项目都有依赖!! 
截图后删除npm文件夹，给nvm一个干净的安装环境，这一点网上有些教程就略去了，所以要注意这些提前工作，截图是为了安装好nvm-nodejs后给补回来。 这一步推荐看[这篇文章](http://www.w2bc.com/article/189615)，个人觉得是一堆文章里的一股清流，mac和windows下的提前准备工作都说的比较清楚。 可惜我是安装完了才删除的，提前工作没做好~。

### 安装NVM软件

这一步其实没啥技术含量，下载好软件，以管理员身份运行，规定安装到`C:\nvm`下，一直下一步就行了，没有什么需要警惕的全家桶(大雾)，安装好之后，我就百度了一下使用方法，get了几个指令，推荐看这个知乎上的教程：[安装管理多个版本的node.js](https://zhuanlan.zhihu.com/p/24698499)。如install、use、list之类的简单指令，其实用这个切换一点也不复杂，这几个够用了，不过用起来可费了一番力气。若教程失效，其实直接看github上的文档就行，这块的说明很简单的。

### 切换安装源

这就是最大的坑，我看了几篇教程，打开setting文件各种设置都不管用，最后返璞归真，从github的文档中才发现如何在国内切换到正确的安装源上。
我一开始没切换，结果使用nvm install命令总是报连不上服务器，没法获取版本地址，大概试了10次，后来在说明文档上发现需要用命令进行设置，其实本质也是写到setting文件中，具体为：
- `nvm node_mirror https://npm.taobao.org/mirrors/node/` 切换到淘宝的node镜像
- `nvm npm_mirror https://npm.taobao.org/mirrors/npm/` 切换到淘宝的npm镜像
这之后就顺畅多了，基本一路安装，我安装了4.4.4和6.10.1两个版本

### 补回失去的全局模块

对照第二步中的截图，一般情况下，在国内全局安装的第一个包都是cnpm，所以直接`npm install cnpm -g`即可，接下来就是对照自己的项目需要，全局安装各种包即可，不过要记住每个版本都需要安装一遍，别嫌烦，切换就用use命令就行，然后cnpm各种包就好了。

*到这里就基本结束了，后面发现bug，则此博文未完待续···*