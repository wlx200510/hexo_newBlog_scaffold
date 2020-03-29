---
title: 对列表hover效果的小探讨
date: 2016-05-14 11:27:08
categories:
- 技术分享
tags:
- CSS
- HTML
---
原来用list写的nav标签效果，博客转载到新网站。
<!-- more -->
## 有hover效果的页面导航的制作思路

<p style="text-indent:32px;">在做顶部页面导航时，很多情况下网站需要鼠标悬停的底部高亮线的效果。这部分nav一般是采用ul+lian+a的方式来做，li设置display为行内块元素。今天的想法是让这个悬停产生的线如何跟这个字的宽度相同，而不是li的宽度？</p>

<p style="text-indent:32px;">这个想法的关键在于这个悬停线统一是用伪元素来实现的，伪元素的宽度就是这个悬停线的宽度，因此css类选择器应该是 .nav ul li a:hover::after的格式。但还是出现了一些小问题，比如这个a的宽度只能是被里面文字撑开的宽度，不能设置值 一般对a就设置width:100%即可。 但导航的文字之间是有空隙的，这个是个技巧，也就是设置li的margin值，从而让导航按钮之间有个等大的间隔。 具体实现代码如下：</p>

```HTML
<div class="nav">
   <ul>
      <li><a href="#">首页</a></li>
      <li><a href="#">特卖</a></li>
      <li><a href="#">儿童</a></li>
      <li><a href="#">学习机</a></li>
      <li><a href="#">加入我们</a></li>
      <li><a href="#">APP下载</a></li>
   </ul>
</div>
```

```CSS
.nav{width:450px; height:26px; position:absolute; right:24%; bottom:18px;}
.nav ul::after{content: ""; display: block; clear:both;}
.nav ul li{ float: left; height:26px; line-height: 26px; position:relative;margin:0 12px;}
.nav ul li a{font-size:18px; color:#464646; display: block; }
.nav ul li a:hover::after{content:"";width:100%; 
border-bottom:2px solid #49d7e7; position: absolute;
left:0; bottom:-18px;}
```

*一点小记录，希望对新人有启示作用。*