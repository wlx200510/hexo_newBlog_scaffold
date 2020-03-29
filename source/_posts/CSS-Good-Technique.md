---
title: 样式代码编写习惯约定——高逼格
date: 2017-2-12 09:03:33
categories:
- 工作总结
tags:
- CSS
- 技巧
---
参加工作后，接触到更多的是sass的编写，这里把公司约定的CSS样式的编写习惯移步于此，方便查询并与大家分享
<!-- more -->
## CSS代码规范

**写在最前面的一句话：sass不要嵌套过深，不允许超过3层**

### **命名规范**

**RULE1.  命名**全部**小写**，不允许出现大写或者驼峰命名，使用**短中划线**，不允许出现下划线命名

**bad**


```
.someSelector {
  margin: 0;
}
```

```
.some_selector {
  margin: 0;
}
```


**good**

```
.some-selector {
  margin: 0;
}
```


**RULE2**. 尽量使用**有意义的**命名，**不允许**出现1,2,3,4的后缀区分，尽量避免使用拼音，除非耳熟能详的，比如分享的weixin，weibo这种的一看就知道意义的

**bad**

```
.selector1 {
  margin: 0;
}
```

```
.biaoti {
  font-size: 18px;
}
```


**good**

```
.title {
  font-size: 18px;
}
```

**RULE3**. 引入`JavaScript`钩子，`.js-*` classes 来表示行为(相对于样式)，但是不要在 CSS 中定义这些 classes。大家要习惯于使用

**bad**

```
`<div class="`selector`"></div>`
```

```
`$('`.selector`').height();`
```

**good**

```
<div class=“selector js-selector"></div>
```

```
$('.js-selector').height();
```

### **选择器**

**RULE4**. 在一个widget里面最多使用一个声明widget意义的id，其他不得滥用id提高选择器的权重

**bad**

```
<div id="section">
  <div id=“header"></div>
</div>
```

**good**

```
<div id="section">
  <div></div>
</div>
```


**RULE5**. 如无必要，不得为** **`id`、`class`** **选择器添加类型选择器进行限定。 比如div#id，p.class之类

**bad**

```
div#header {
  font-size: 20px;
}
```

**good**

```
#header {
  font-size: 20px;
}
```


**RULE6.** 多选择器规则之间必须换行

**bad**

```
`.element, p {
  margin: 10px 0 @variable*2 10px;
  `font-size: 12px;`
}`
```

**good**

```
`.element,
p {
  margin: 10px 0 @variable*2 10px;
  `font-size: 12px;`
}`
```

### **编码书写规范**

**RULE7.** sass使用四个空格

**bad**

```
.element {
    margin: 10px 0 @variable*2 10px;
    `font-size: 12px;`
}
```

**good**

```
.element {
  margin: 10px 0 @variable*2 10px;
  `font-size: 12px;`
}
```


**RULE8.** 属性定义必须另起一行

**bad**

```
.element {
  margin: 10px 0 @variable*2 10px;`font-size: 12px;`
}
```

**good**

```
.element {
  margin: 10px 0 @variable*2 10px;
  `font-size: 12px;`
}
```

**RULE9.** 每个属性后面必须加 ';'

**bad**

```
.element {
  margin: 10px 0 @variable*2 10px
}
```

**good**

```
.element {
  margin: 10px 0 @variable*2 10px;
}
```


**RULE10**. 文本使用双引号

**bad**

```
.element:after {
  `content: '.'`;
}
```

**good**

```
.element:after {
  `content: "."`;
}
```

**RULE11**. sass变量的计算用括号，便于阅读

**bad**

```
.element {
  margin: 10px 0 @variable*2 10px;
}
```

**good**

```
.element {
  margin: 10px 0 (@variable * 2) 10px;
}
```


**RULE12**. 选择器与 `{ `之前`必须`要有空格

**bad**

```
.element{
  margin: 10px 0 (@variable * 2) 10px;
}
```

**good**

```
.element {
  margin: 10px 0 (@variable * 2) 10px;
}
```


**RULE13**. 属性名的` : `后必须要有空格, 属性名的` : `前禁止加空格

**bad**

```
.element {
  margin : 10px 0 (@variable * 2) 10px;
}
```

**good**

```
.element {
  margin: 10px 0 (@variable * 2) 10px;
}
```

**RULE14**. 数值单位的属性值，如值为零，则不得带单位。如不能有 0px的书写出现

**bad**

```
.element {
  margin: 10px 0px;
}
```

**good**

```
.element {
  margin: 10px 0;
}
```


**RULE15**. 在可以使用缩写的情况下，尽量使用属性缩写（建议）

**bad**

```
.element {
  margin-top: 10px;
  margin-right: 5px;
  margin-bottom: 2px;
  margin-left: 0
}
```

**good**

```
.element {
  margin: 10px 5px 2px 0;
}
```


**RULE16**. 当数值为 0 - 1 之间的小数时，省略整数部分的** **`0`

**bad**

```
.element {
  margin: 0.1rem;
}
```

**good**

```
.element {
  margin: .1rem;
}
```


**RULE17**. `url()`** **函数中的路径不加引号

**bad**

```
.element {
  background: url("test.png");
}
```

**good**

```
.element {
  background: url(test.png);
}
```


**RULE18**. 在自适应的布局上，小图片要用px单位并且后面必须添加?__sprite的标记

**bad**

```
.element {
  background: url(test.png);
  width: 2em;
  height: 2em;
}
```

**good**

```
.element {
  background: url(test.png?__sprite);
  width: 32px;
  height: 32px;
}
```


**RULE19. **`自适应布局Media Query`** **不得单独编排，必须与相关的规则一起定义。

**bad**

**good**

**RULE20**. 需要使用hack的地方先想想自己哪地方做的不好，是否真的需要hack，目前浏览器需要用到hack的地方着实不多

**RULE21**. 清除浮动不得添加空标签的方式进行，多使用伪类，或者去了解BFC的相关规则，基本上能覆盖开发中的全部情况

**bad**

```
<div class="selector">
  <div class="clearfix"></div>
</div>
```

```
.clearfix {
  clear: both;
}
```

**good**

```
<div class="selector"></div>
```

```
.selector {
 &:after {
  display: table;
  clear: both;
  content: "";
 }
}
```


**RULE22**. 带私有前缀的属性由长到短排列，按冒号位置对齐

**bad**

```
-webkit-user-select: none;
-moz-user-select: none;
-ms-user-select: none;
user-select: none;
```

**good**

```
-webkit-user-select: none;
   -moz-user-select: none;
    -ms-user-select: none;
        user-select: none;
```

#### rule23. 组件的样式中不使用id选择器

**bad**

```
#news-list-wrap {
    color: #000;
}
```

**good**

```
// 如果担心组件样式被覆盖，可在class后面加上 -xxx 以保证唯一性
// 基本原则就是通过给class加后缀来表示特殊性
.news-list-wrap-sth {
    color: #000;
}
```



### **属性的书写顺序**

1. 定位相关, 常见的有：display position left top float 等
2. 盒模型相关, 常见的有：`width` `height` `margin` `padding` `border` 等
3. 其他属性

示例 ：

```
.content {
  /* 定位 */
  display: block;
  position: absolute;
  left: 0;
  top: 0;
  /* 盒模型 */
  width: 50px;
  height: 50px;
  margin: 10px;
  border: 1px solid black;
  / *其他* /
  color: #efefef;
}
```

按照这样的顺序书写可见提升浏览器渲染dom的性能
简单举个例子，网页中的图片，如果没有设置width和height，在图片载入之前，他所占的空间为0，但是当他加载完毕之后，那块为0的空间突然被撑开了，这样会导致，他下面的元素重新排列和渲染，造成重绘（repaint）和回流（reflow）。我们在写css的时候，把元素的定位放在前头，首先让浏览器知道该元素是在文本流内还是外，具体在页面的哪个部位，接着让浏览器知道他们的宽度和高度，border等这些占用空间的属性，其他的属性都是在这个固定的区域内渲染的，差不多就是这个意思吧~ （这个咱们遵守的最不好的，以后注意书写顺序，做一个有逼格的csser）

另外，大家多挖掘sass，发挥他的威力，别以为只是一个方便嵌套的玩意，for，each map等能带来很多书写的便利提升效率

