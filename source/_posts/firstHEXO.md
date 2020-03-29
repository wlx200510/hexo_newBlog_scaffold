title: 使用HEXO搭建个人博客遇到的几个问题
date: 2016-8-13 17:40:00
categories:
- 技术分享
tags:
- Github
- Hexo
- 域名
---
还好有些github的使用基础，因此hexo博客的搭建用了一天就基本完成了，但这个过程中还是有两个小问题在文档上没有找到，还好通过查资料得到了解决，及时分享出来，希望让他人少走一些弯路。
<!-- more -->
<style>
    .text{
        text-indent:32px;
        display: inline-block;
    }
</style>
### 这套博客搭建根据**hexo**官网的文档和Yelee模板文档解决了90%的问题，但仍有一些小的技巧性问题值得分享

 1. [hexo文档地址](https://hexo.io/zh-cn/docs/)
 2. [Yelee主题使用说明](http://moxfive.coding.me/yelee/)
 
<span class="text">我在本地搭建好整体的博客框架之后，通过`hexo clear && hexo g`在bash中运行后已经在本地可以预览到，但是通过deoloy却提示失败，通过索引相关的提示错误信息，将hexo配置文件中的部署地址由`https://github.com/用户名/用户名.github.io`改为`git@github.com:用户名/用户名.github.com`.问题得到解决，成功部署到了github上。</span>

<span class="text">域名更改这块的内容，我主要参考的是简书上的一篇文章，同时对域名这块的知识点补了补课: [自定义域名设置](http://www.jianshu.com/p/00a19bb425cb); 尤其是在命令行里输入ping username.github.io（请将username换成你的用户名）来获取IP的思维值得借鉴，不过这里我用的是github给出的那两个IP地址也没问题，所以两种方法都是可以的。建议通过阿里云万网购买的域名都可以按照这个思路来设置，截图如下:</span>

![设置图](http://upload-images.jianshu.io/upload_images/1830662-b76412650c334b19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
