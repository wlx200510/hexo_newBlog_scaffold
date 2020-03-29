---
title: Javascript一些优雅实现
date: 2017-10-22 21:34:55
categories:
- 学习
tags:
- javascript
- 总结
---
作为一个jser, 当对一门语言的使用到达一定量时，不免会疲倦，仅以此文来惊醒，其实它还有很多地方等待探索~
<!-- more -->
<!-- excerpt -->
## 优雅实现`sleep`的效果
在诸如`python/Java`中都有`sleep`函数，但`js`没有，那应该如何用最优雅的方法实现？
### 粗暴版本
```js
function sleep(time) {
    for (var start = +new Date; +new Date - start < time;) {}
}
```
这样就会把所有的执行任务`block`掉, 客户端还好，服务器就炸毛了。

### `Promise`版本
```js
function sleep(time) {
    return new Promise((resolve) => setTimeout(resolve, time))
}
const t1 = +new Date()
sleep(3000).then(()=>{
    const t2 = +new Date()
    console.log(t2 - t1)
})
```
这种方式不会阻塞，无负载问题，使用时代码量不多，但`Promise`内部的代码出错后需要层层判断后`catch`，还是有些麻烦。

### `Async/Await`版本的延迟函数
```js
function sleep(delay) {
  return new Promise(reslove => {
    setTimeout(reslove, delay)
  })
}

async function test() {
  const t1 = +new Date()
  await sleep(3000)
  const t2 = +new Date()
  console.log(t2 - t1)
}()
```
各个函数都是非嵌套结构，现在开始要拥抱这种写法，整体写法可参照：[四个改进点](https://juejin.im/post/59f9ce7a51882554f666220f)

### 用`C++`实现的绝对优雅
> 这里有 C++ 实现的模块：https://github.com/ErikDubbelboer/node-sleep

```js
const sleep = require("sleep")

const t1 = +new Date()
sleep.msleep(3000)
const t2 = +new Date()
console.log(t2 - t1)
```
*现在开始异步请求都要试着在Promise的基础上拥抱Async/Await*

## 获取时间戳的方法
1. 第一层级别：`var timestamp = new Date().getTime()`
2. 第二层级别：`var timestamp = (new Date()).valueOf()` 使用`valueOf`方法返回对象的原始值
3. 第三层级别：`var timestamp = +new Date()` 使用了隐式转换 可参考[来源博文](https://github.com/jawil/blog/issues/30) 讲的很详细

这个方面原文中对隐式转换讲解的比较透彻，作为js的基础应该适当回顾学习。原始值指的是 'Null','Undefined','String','Boolean','Number','Symbol' 6种基本数据类型之一，不能忘却。

## 数组去重的算法进阶

> 暂不考虑对象字面量，函数等引用类型的去重，也不考虑 NaN, undefined, null等特殊类型情况

### 普通版
只说下思路，这种复杂度为平方的不再上代码，整体就是对这个数组每个元素取出来放到另一个数组中，在放入之前检验这个新数组中是否已有这个值，防止重复注入

### 进阶版
```js
var a =  [1, 1, '1', '2', 1]
function unique(arr) {
    return arr.filter(function(ele,index,array){
        return array.indexOf(ele) === index
    })
}
console.log(unique(a)) // [1, 2, "1"]
```
利用函数式编程一对一去掉重复的数，写法相当优雅，但复杂度与基础版相当

### 超进阶版
```js
function unique(arr) {
    var obj = {}
    return arr.filter(function(item, index, array){
        return obj.hasOwnProperty(typeof item + item) ? 
        false : 
        (obj[typeof item + item] = true)
    })
}
```
之前最喜欢的一种去重方法，通过引入字典查找，将复杂度降低为`O(n)`，这个示例更加极端，把代码缩减到极致 不过易读性受影响

### 究极进化版
```js
const unique = a => [...new Set(a)]
```
`ES6`的`Set`数据结构与解构写法的完美结合，最优雅写法和可读性。

## 数组的转化与求和
在前端数组是一个最常用的数据结构，这是两个数组常用的操作，看一下如何优雅实现
### 数组转化
需求是把`argruments`对象(类数组)转换成数组:
- 兼容性良好的版本：`var arr = Array.prototype.slice.call(arguments);` 不再赘述
- ES6的新写法：`var arr = Array.from(arguments);`(有length属性就可以) `var args = [...arguments];`(解构写法)

第一种方法的原理来源博客讲得比较细，主要是探究`ArraySlice`的源码实现，摘抄如下：

> `slice.call`的作用原理就是，利用`call`，将`slice`的方法作用于 `arrayLike`，`slice`的两个参数为空，`slice`内部解析使得 `arguments.lengt`等于0的时候 相当于处理 `slice(0)` ： 即选择整个数组，`slice`方法内部没有强制判断必须是`Array`类型，`slice`返回的是新建的数组（使用循环取值）”，所以这样就实现了类数组到数组的转化，`call`这个神奇的方法、`slice`的处理缺一不可。

### 数组求和
直接上最优雅的迭代
```js
let arr = [1, 2, 3, 4, 5]
function sum(arr) {
return arr.reduce((a, b) => a + b)
}
sum(arr) //15
```

## 位运算黑科技总结
这个小节接触一下传说中的`ACM`常用技巧~
### 交换两个数字
1. 引入中间变量的方法(略去不写)
2. 使用纯数字运算的方法(特别大的数字不行)
```js
let a = 3,b = 4
a += b
b = a - b
a -= b
```
3. 解构赋值 `[a, b] = [b, a]`
4. 用异或实现
```js
let a = 3,b = 4
  a ^= b
  b ^= a
  a ^= b
console.log(a, b)
```
> 对于开始的两个赋值语句，`a=a^b，b=b^a`，相当于`b=b^(a^b)=a^b^b`，而`b^b`显然等于0。因此`b=a^0`，显然结果为`a`。
> 同理可以分析第三个赋值语句，`a=a^b=(a^b)^a=b`

优点：不存在引入中间变量，不存在整数溢出
缺点：前端对位操作这一块可能了解不深，不容易理解

### 数字取整
这是个比较常见的前端需求，普通的处理办法有`parseInt`和`Math.trunc`等
各种位运算黑科技系列(大数fail)：
1. `~~number`取整
```js
console.log(~~47.11)  // -> 47
console.log(~~1.9999) // -> 1
console.log(~~3)      // -> 3
console.log(~~[])     // -> 0
console.log(~~NaN)    // -> 0
console.log(~~null)   // -> 0
console.log(~~2147493647.123) // -> -2147473649 🙁
```
2. `number | 0`按位或取整
```js
console.log(20.15|0);          // -> 20
console.log((-20.15)|0);       // -> -20
console.log(3000000000.15|0);  // -> -1294967296 🙁
```
3. `number ^ 0`按位异或取整
```js
console.log(20.15^0);          // -> 20
console.log((-20.15)^0);       // -> -20
console.log(3000000000.15^0);  // -> -1294967296 🙁
```
4. `number << 0`左移取整
```js
console.log(20.15 < < 0);     // -> 20
console.log((-20.15) < < 0);  //-20
console.log(3000000000.15 << 0);  // -> -1294967296 🙁
```

## 压轴之数字格式化
> 需求是把 `1234567890` 转换为 `1,234,567,890` (加上千位分隔符)

### 普通版
```js
function formatNumber(str) {
  let arr = [], count = str.length;

  while (count >= 3) {
    arr.unshift(str.slice(count - 3, count))
    count -= 3
  }

  // 如果是不是3的倍数就另外追加到上去
  str.length % 3 && arr.unshift(str.slice(0, str.length % 3))
  return arr.toString()
}
```
这个版本就是按照流线思路下来，其实也不是很简单，思路比较普通，不过也需要注释

### 进阶版
```js
function formatNumber(str) {
  // ["0", "9", "8", "7", "6", "5", "4", "3", "2", "1"]
  return str.split("").reverse().reduce((prev, next, index) => {
    return ((index % 3) ? next : (next + ',')) + prev
  })
}
```
感觉真心是函数式编程玩到骨子里去了，真心没注释不好看懂，编程珠玑中有类似的思路提到，看完之后恍然大悟的感觉。

### 进阶正则版
```js
function formatNumber(str) {
  return str.replace(/\B(?=(\d{3})+(?!\d))/g, ',')
}
```
建议好好学习这个思路，正则真心是瑞士军刀，类似问题都会有这个方面的办法，就是找到快慢的问题。
### 终极进化版
```js
(123456789).toLocaleString('en-US')
```
没看到那篇博客是真不知道还有这个`API`……
```js
(123456789).toLocaleString('zh-hans-CN-u-nu-hanidec', {useGrouping: false})
(123456789).toLocaleString('zh-hans-CN-u-nu-hanidec', {useGrouping: true})
```
PS: 感觉是某牛总结的[前端工具箱](https://github.com/CodingMeUp/some_notes/tree/master/%E5%89%8D%E7%AB%AF) 可以分享一波