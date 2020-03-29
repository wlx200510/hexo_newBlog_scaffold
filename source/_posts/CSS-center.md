---
title: 用CSS实现居中的总结
date: 2016-9-14 09:03:33
categories:
- 前端知识
tags:
- CSS
- HTML
---
无论是工作还是面试中，元素居中总是一个高频的问题，不同情况下需要灵活地选取不同的方法，因此有必要对常用方法做一个简要的总结。
<!-- more -->
### 行内或者行内块元素居中

1.单行竖直居中
> 给行内元素添加上下相同大小的`padding`值即可
> 设置元素的`line-height`等于父容器的高度，也可以竖直居中
> 使用弹性盒子`Flex`后，设置`align-items`为`center`
> 涉及到图片文字混排竖直居中使用vertical-align，可以使用伪元素加入以下代码

```css
container::after{
    content:'';
    display:inline;
    vertical-align:middle;
}
```

2.多行文字竖直居中
> 设置`padding`值仍然适用
> 若要使用`vertical-align`属性来垂直居中，可以将父容器设置为`table`，需要居中的元素`display`设置为`table-cell`，从而完成所需效果。
> 使用弹性盒子`Flex`后，设置`align-content`为`center`

3.水平居中
> 使用`text-align`设置为`center`。
> 使用弹性盒子`Flex`后，设置`justify-content`为`center`

### 块元素居中

1.竖直居中

> 使用定位，若是浮动的元素需要一个多余的元素来包裹要居中的元素，需要设置`position:relative`，而后设置`top:50%;`子元素设置相反方向50%，如`top:-50%;`。
> 使用定位，若是绝对定位的元素，在宽度已知时，先绝对定位`top:50%`，利用margin为负值的特性向左移动自身宽度的一半。若宽度未知，可使用`transform:translateY(-50%)`实现。通用scss的mixin如下：

```CSS
@mixin abs_h_center($width){
    position: absolute;
    width: $width;
    left: 50%;
    margin-left: -($width/2);
}
@mixin abs_v_center($height){
    position: absolute;
    height: $height;
    top: 50%;
    margin-top: -($height/2);
}
/*使用的例子*/
.content{
    @include abs_v_center(200px);
}
```

2.水平居中

> 弹性盒子的设置方法同行内元素，不在赘述
> 对于宽度已定的块元素直接使用`margin:0 auto`属性水平居中
> 若宽度不确定，应该使用`table`来完成布局，也设置`margin:0 auto`
> 使用定位，若是浮动的元素，需要设置`position:relative`，而后设置`left:50%;`子元素设置相反方向50%，如`left:-50%;`。
> 使用定位，若是绝对定位的元素，在宽度已知时，先绝对定位`left:50%`，利用margin为负值的特性向左移动自身宽度的一半。若宽度未知，可使用`transform:translateX(-50%)`实现。

### 水平竖直同时居中的大招

1.直接上代码（兼容IE8+）

```CSS
.parent{
    width:200px; height:200px; border:1px solid red; position: relative;
}
.child{
    background:#000; position:absolute; width:100px; height:100px;
    left:0; right:0; top:0; bottom:0; margin:auto;
}
```

这里如果不定义元素的宽和高的话，那么他的宽就会由left,right的值来决定，高会由top,bottom的值来决定，所以必须要设置元素的高和宽。同时如果改变left，right , top , bottom的值还能让元素向某个方向偏移，大家可以自己去尝试。

2.HACK的全兼容全居中代码
IE8+、火狐谷歌等现在浏览器中可以用display:table-cell来进行居中，而font-size的方法则适用于IE6和IE7，结合这两者的代码如下：

```CSS
.parent{
    *font-size:175.4px; /*fontsize的值为元素高度除以1.14所得结果，适合IE6、7*/
    height:200px; width:200px; text-align: center;/*标准浏览器的方法*/
    display: table-cell; vertical-align:middle;
}
.child{
    display:inline-block;
    *zoom:1; *display:inline; /*在ie6和7中实现inline-block的方法*/
    *vertical-align:middle; /*使用font-size方法必须*/
    font-size:12px; /*改回正常的font-size数值*/
    width:50px; height:50px; background:#00f;
}
```

注意`vertical-align:middle`写在父元素中才行。