---
title: js线程机制的介绍和练习
date: 2017-12-02 13:59:04
top: true
categories:
- 学习
tags:
- 工作
- javascript
- promise
---
通过几个小题目来深入了解一下js代码的运行机制 扩展至promise的学习
<!-- more -->
> 给定的几行代码，我们需要知道其输出内容和顺序。JavaScript是一门单线程语言，但有其独特的线程机制

热身代码：
```javascript
setTimeout(function(){
    console.log('定时器开始啦')
});
new Promise(function(resolve){
    console.log('马上执行for循环啦');
    for(var i = 0; i < 10000; i++){
        i == 99 && resolve();
    }
}).then(function(){
    console.log('执行then函数啦')
});
console.log('代码执行结束');
```
#JavaScript事件循环

- 同步任务 -> 主线程 -> 任务顺序执行完毕
- 异步任务 -> Event Table -> EventQueue(并入主执行线程)

同步和异步任务分别进入不同的执行"场所"，同步的进入主线程，异步的进入`Event Table`并注册函数。
当指定的事情完成时，`Event Table`会将这个函数移入`Event Queue`。
主线程内的任务执行完毕为空，会去`Event Queue`读取对应的函数，进入主线程执行。
上述过程会不断重复，也就是常说的`Event Loop`(事件循环)。

*JS引擎存在monitoring process进程，会持续不断的检查主线程执行栈是否为空，一旦为空，就会去Event Queue那里检查是否有等待被调用的函数。*

#定时器线程

1. `setTimeout`这个函数，是经过指定时间后，把要执行的任务加入到Event Queue中，又因为是单线程任务要一个一个执行，如果前面的任务需要的时间太久，那么只能等着
2. `setTimeout(fn,0)`的含义是，指定某个任务在主线程最早可得的空闲时间执行，只要主线程执行栈内的同步任务全部执行完成，栈为空就马上执行。
3. `setInterval`会每隔指定的时间将注册的函数置入Event Queue，如果前面的任务耗时太久，那么同样需要等待。
4. 对于`setInterval(fn,ms)`来说，我们已经知道不是每过ms秒会执行一次fn，而是每过ms秒，会有fn进入Event Queue。

*一旦setInterval的回调函数fn执行时间超过了延迟时间ms，那么就完全看不出来有时间间隔了*

# Promise与process.nextTick(callback)

process.nextTick 指在node.js里面，事件循环的下一次循环中调用callback

除了广义的同步和异步任务，更精细的定义为：
> macro-task(宏任务)：包括整体代码script，setTimeout，setInterval
> micro-task(微任务)：Promise，process.nextTick
接下来的主要介绍这两个任务的概念和线程表现:

1. 这两种类型的任务会进入与之对应的EventQueue
2. 事件循环的顺序，决定JS代码的执行顺序
3. 先是进入整体代码的宏任务，开始事件循环，然后紧接着执行当前宏任务的微任务
4. 执行完当前宏任务的微任务后 进入EventQueue里面的下一个宏任务

![js两种任务机制](http://opm3cm6nh.bkt.clouddn.com/loop.jpg)

# 代码练习
我们来分析一段较复杂的代码，看看你是否真的掌握了JS的执行机制
```javascript
console.log('1');
setTimeout(function() {
    console.log('2');
    process.nextTick(function() {
        console.log('3');
    })
    new Promise(function(resolve) {
        console.log('4');
        resolve();
    }).then(function() {
        console.log('5')
    })
})
process.nextTick(function() {
    console.log('6');
})
new Promise(function(resolve) {
    console.log('7');
    resolve();
}).then(function() {
    console.log('8')
})
setTimeout(function() {
    console.log('9');
    process.nextTick(function() {
        console.log('10');
    })
    new Promise(function(resolve) {
        console.log('11');
        resolve();
    }).then(function() {
        console.log('12')
    })
})
```
。
。
。
。
。
。

标准答案：1 7 6 8 2 4 3 5 9 11 10 12

总结：
- JavaScript是一门单线程语言
- Event Loop是JavaScript的执行机制
- 针对Promise的知识，[这里](https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html)推荐一篇文章，非常值得一看