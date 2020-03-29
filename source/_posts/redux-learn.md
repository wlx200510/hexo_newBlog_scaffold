---
title: REACT框架学习心得
date: 2018-02-08 18:22:47
categories:
- 学习
tags:
- REACT
---
最近参考一个github上的高star量的`REACT`框架总结学习到了不少东东，现在简单在博客上做个小分享。
<!-- more -->
<!-- excerpt -->

> 先上一个`REACT`的框架源码介绍的[文章](https://medium.com/@ericchurchill/the-react-source-code-a-beginners-walkthrough-i-7240e86f3030)，辅助源码的解释，团队推荐。

## 框架概念和工程模板

　　这一部分的内容直接给个链接，[`github`地址](https://github.com/wlx200510/react-wlx)在此，我在原作者的基础上加了自己的一些内容，仓库的最新代码也进行了重构，包括`actionCreator`和`reducer`，欢迎提意见和`star`。这个教程可以说是把react的轮廓放到了大家面前，并且这个代码也可以`fork`一份直接用到自己的移动端项目上，`PC`端还需要把REM取消后使用。
　　这个`README`解决的是框架可用的问题，相信用这个demo来让新人入门也好，代码进阶也罢，都可以顺利完成，不过具体的REACT框架原理建议在项目跑起来后要有所钻研，但源码又不好立即入手，这就需要上面的框架源码介绍文章了，希望对源码有探究心的小伙伴参考一开始推荐的文章，也可以看[这一篇](https://bogdan-lyashenko.github.io/Under-the-hood-ReactJS/?utm_campaign=read_more&utm_medium=blog&utm_source=mybridge)的分章节介绍，更加详尽。

## redux学习心得

简单说一下在看整个教程和代码写法时着重的几个点：

1. `connect`函数的[`mapDispatchToProps`]参数传入一个对象，当对象中键属性是对象，实际是自动包含了dispatch调用，从而不需要显式调用。
2. 当引入了`redux-thunk`后，需要在createStore上加入`thunk`中间件，并且上一点中键的属性变成了函数，此时异步下的`dispatch`需要显式调用。
3. 为了减少样板代码，需要统一用数据检索的思想来完成`reducer`和`action`对象的生成逻辑，具体可参考github中的样例。
4. 前端的数据其实就是三类：Domain data(服务器给的展示数据)、App state(某个行为的标识数据，如正在请求数据之类)、UI state(UI状态)
5. `Store` 代表着应用核心，因此应该用域数据(`Domain data`)和应用状态数据(`App state`)定义 `State`，而不是用 UI 状态(`UI state`)
6. Reducer的重构介绍中一个核心概念需要理解，就是函数分解，在redux重构中又分为 工具函数/业务逻辑/高阶函数 三种拆分技巧

　　*建议仔细看下`redux`重构技巧，中文翻译版本[在此](http://cn.redux.js.org/docs/recipes/)，英文好的建议看原版。里面关于实现撤销重做的思路介绍，感觉非常不错。*

## 进一步探索

　　其实我这里想说的就是`react`的最佳实践的东西，确切来说就是组件拆分这一块，我感觉用react的很重要的进阶就是知道什么时候使用无状态组件，如何合理拆分组件，其实比函数分解都难，尤其在实际业务中还会有越拆越麻烦的现象发生，另外一点就是要稳准地找到高阶组件的切入点，解决开发中冗余代码和逻辑的痛点，必要的时候还要跟产品沟通，来整合通用逻辑，方便增量开发和维护。
　　另一方面还要探索`react`各种库的使用和实现，毕竟作为工程师，实现需求是最重要的，包括但不限于各种UI库的引入，比如最新的iceworks的的代码生成的学习，这都是提高`REACT`水平的良好机会，在此奉上`iceworks`的[地址](https://github.com/alibaba/ice)，望大家一起进步。