---
title: KOA框架学习
date: 2018-02-11 18:37:04
categories:
- 学习
tags:
- 后端
---
根据两篇完整教程，来看koa中的一些基础知识，争取变成自己的技术栈
<!-- more -->
<!-- excerpt -->
koa框架特点: 优雅/简介/自由度高 所有功能都要通过插件实现，本身很轻
使用环境: Node版本为8.x 
koa搭建服务器的基本代码套路:
```js
const Koa = require('koa')
const app = new Koa()
// ctx是Koa提供的上下文对象(包括 HTTP 请求和 HTTP 回复)
const main = async (ctx, next) => {
    ctx.response.body = 'Hello World';
    await next()
}

app.use(main) // 通过use来启用中间件
app.listen(3000)
```


## Context对象API
*ctx对request和response的API有直接的引用方式,列表解释时直接用ctx来引用*

### 应用层API
`ctx.state` 推荐的命名空间，用于储存传递的信息
`ctx.app` 应用程序的实例引用
`ctx.cookies.get(name, [options])` 通过 `options` 获取 `cookie name`
`ctx.cookies.set(name, value, [option])` 向`cookie`内写入内容(option内容参照[cookies](https://github.com/jed/cookies)模块)
`ctx.throw([status], [msg], [properties])` 用于客户端响应的错误消息如：`ctx.throw(401, 'access_denied', { user: user });`更多:[http-errors](https://github.com/jshttp/http-errors)
`ctx.assert(value, [status], [msg], [properties])` 当value不存在是 抛出类似于throw的错误，参照[http-assert](https://github.com/jshttp/http-assert)


### HTTP请求
`ctx.req(request)`为核心(`ctx`)

`ctx.is()`:
非常类似 ctx.request.is(). 检查响应类型是否是所提供的类型之一。这对于创建操纵响应的中间件特别有用。
例如, 这是一个中间件，可以削减除流之外的所有HTML响应。

```js
const minify = require('html-minifier');
app.use(async (ctx, next) => {
  await next();
  if (!ctx.response.is('html')) return;
  let body = ctx.body;
  if (!body || body.pipe) return;
  if (Buffer.isBuffer(body)) body = body.toString();
  ctx.body = minify(body);
});
```


| API | 功能 |
| - | - |
| ctx.header(s) | 请求头内容 |
| ctx.method | 请求方法 |
| ctx.url(path) | 请求的路由地址 |
| ctx.origin | 请求来源 |
| ctx.query | GET请求的参数(对象形式) |
| ctx.querystring | GET请求参数的字符串 |
| ctx.host(hostname) | 请求的http服务器域名/IP |
| ctx.ip | 运行客户端机器的IP |
| ctx.protocol | HTTP协议 |
| ctx.socker | 请求的套接字 |
| ctx.is() | 检查响应类型是否是提供的类型 |
| <Accetp首部>ctx.accepts() | 服务器能返回的媒体类型 |
| <Accetp首部>ctx.acceptsEncodings() | 服务器能返回的编码方式 |
| <Accetp首部>ctx.acceptsCharsets() | 服务器能返回的字符集 |
| <Accetp首部>ctx.acceptsLanguages() | 服务器能返回的语言类型 |


### HTTP响应
`ctx.res(response)`为核心
*ctx.response.body = ctx.body* 
基本上所有的信息都可以设置
`ctx.res.flushHeaders()` 刷新任何设置的标头，并开始主体。

`ctx.body`可以赋值的内容：
- string 写入 Content-Type 默认为 text/html 或 text/plain, 同时默认字符集是 utf-8。Content-Length 字段也是如此
- Buffer 写入 Content-Type 默认为 application/octet-stream, 并且 Content-Length 字段也是如此。
- Stream 管道 Content-Type 默认为 application/octet-stream。
- Object || Array JSON-字符串化 Content-Type 默认为 application/json
- null 无内容响应

`ctx.set`API设计如下：
- 传入两个参数，分别为KEY和VALUE(`ctx.set('Cache-Control', 'no-cache');`)
- 传入一个简单对象，用于设置多个响应头内容

`ctx.type`设置的`Content-Type`一般指mime字符串或者文件扩展名
```js
ctx.type = 'text/plain; charset=utf-8';
ctx.type = 'image/png';
ctx.type = '.png';
ctx.type = 'png';
```

| API | 功能 |
| - | - |
| ctx.body | 发送给用户的内容 |
| ctx.status | 发送的状态码 |
| ctx.length | 响应的Content-Length |
| ctx.message | 将响应的状态消息设置为给定值(OK) |
| ctx.type | 响应内容的Content-Type |
| ctx.headerSent | 检查是否已经发送了一个响应头 |
| ctx.redirect() | 重定向请求地址(参数是地址) |
| ctx.lastModified | 最后一次被修改的日期时间(Data或日期字符串) |
| ctx.etag | 与返回实体相关的实体标记 |
| ctx.attachment([filename]) | 指定下载filename |
| ctx.set() | 设置响应头内容 |
| ctx.append() | 为响应体附加额外的值 |
| ctx.remove() | 删除响应头内特定字段 |
| ctx.get() | 获取响应头字段(不区分大小写) |

注：常用的需设置的响应首部

| 首部 | 描述 |
| - | - |
| Access-Control-Allow-Method | 跨域时允许使用的方法 |
| Access-Control-Allow-Origin | 跨域允许的请求来源host |
| Access-Control-Allow-Credentials | 跨域是否要传递cookie |
| Access-Control-Allow-Headers | 跨域时允许包含的响应头字段 |

## 必用的中间件

- [koa-router](github.com/alexmingoia/koa-router) 非常方便的路由解决方案 用法参照[这里](https://chenshenhai.github.io/koa2-note/note/route/koa-router.html)
- [koa-bodyparser](github.com/koajs/bodyparser) 用于解析POST方法传来的数据 用法参考[这里](https://chenshenhai.github.io/koa2-note/note/request/post-use-middleware.html)
- [koa-static](github.com/koajs/static) 方便安全地实现静态资源服务器 用法参考[这里](https://chenshenhai.github.io/koa2-note/note/static/middleware.html)
- [koa-views](github.com/queckezz/koa-views) 用于KOA的模板引擎 核心在于`ctx.render` [这里](https://chenshenhai.github.io/koa2-note/note/template/add.html)
- [koa-jsonp](github.com/kilianc/koa-jsonp) 使用JSONP的方式提供跨域接口 用法参照[这里](https://chenshenhai.github.io/koa2-note/note/jsonp/koa-jsonp.html)
- [busboy](github.com/mscdex/busboy) 用于实现服务器上传文件，要与node原生API相配合 用法参考[这里](https://chenshenhai.github.io/koa2-note/note/upload/pic-async.html)

## 测试框架

在后端写API接口，跟前端不同，测试是必须的一环。
建议全局安装mocha测试框架，会在当前目录找到test文件夹，并执行响应的测试工具
在项目里依赖安装chai断言库和supertest接口测试工具，这两者分别用于引入断言API和模拟响应的接口
`supertest`的API需要接口把应用的app暴露出来，然后传入`supertest`方法中

`mocha`工具说明：
- describe()描述的是一个测试套件
- 嵌套在describe()的it()是对接口进行自动化测试的测试用例
- 一个describe()可以包含多个it()

`./test/index.test.js`:
```js
const supertest = require('supertest')
const chai = require('chai')
const app = require('./../index') // 原文件有module.exports = app
const expect = chai.expect //引入断言库
const request = supertest( app.listen() ) //api测试工具的使用方法

// 测试套件/组
describe( '开始测试demo的GET请求', ( ) => {
  // 测试用例
  it('测试/getString.json请求', ( done ) => {
      request
        .get('/getString.json')
        .expect(200)
        .end(( err, res ) => {
            // 断言判断结果是否为object类型
            expect(res.body).to.be.an('object')
            expect(res.body.success).to.be.an('boolean')
            expect(res.body.data).to.be.an('string')
            done()
        })
  })
})
```
直接键入`mocha`来启动项目测试, [chai](http://chaijs.com/)断言库的[学习文档](http://www.html-js.com/article/1875)
*chai.expect使用来判断测试结果是否与预期一样*

## 优化koa2写的服务端API代码组织

> 根据已有的ES7特性 在async/await外 对于接口API的新的代码组织思路的探讨

看了下网上的资料 有以下几点值得考量
1. 接口层和逻辑层的解耦，代码组织清晰
2. 使用class来组织代码，对外暴露必要方法
3. 使用static关键字，避免实例化

### 接口层和逻辑层的解耦

具体思路是在`router.js`文件中只统一处理路由参数，相关处理逻辑都放在其他模块中作为方法暴露出来
代码的组织思路如下：

```javascript
/* /server/router.js */

const router = require('koa-router')()
const userctrl = require('../controller/userControl.js') //例子 用户控制器逻辑

router
    .post('/api/user/login',userctrl.login)         // 用户登录
    .post('/api/user/register',userctrl.register)   // 用户注册      
    .get('/api/user/logout',userctrl.logout)        // 用户退出      
    .put('/api/user/resetpwd',userctrl.resetpwd)    // 重置用户密码
    .delete('/api/user/deluser',resetpwd.deluser)   // 删除用户
```
这就是把第一层进来的逻辑进行拆分，router的写法可能不同的库不一样，仅供参考

### 用class来组织代码

`class`是ES6的新增关键字，可以考虑在写`api`时对方法做类级别的封装，从而更靠近传统语言的后端开发思路。
代码组织示例：

```javascript
/* /server/controller/userControl.js */
import mongoose from 'mongoose'
const UserModel = mongoose.model('User') // 使用的mongoose简化数据库对象操作

class UserController {
    async register(ctx) {
        //具体的注册逻辑 await
    }

    async logout(ctx) {
        // await
    }
}
// ...

export default new UserController()
```
使用`class`关键字来包裹方法，从而让控制器逻辑更清晰。

### 使用static关键字

ES6中，可用`static`来定义静态方法，静态方法并不需要实例化就可以访问，也就意味着，使用`static`，你不需要`new`，你可以减少内存的损耗。
所以上面的例子可以写成：

```javascript
class UserController {
    static async register(ctx) {
        //具体的注册逻辑 await
    }

    static async logout(ctx) {
        // await
    }
}
// ...

export default UserController
```

网上还有个思路很好，就是可以用`class`封装一些公共方法放到另外一个文件夹，然后可以有两种方法来使用，一种是上述类似的静态方法，另外一种是通过`extend`关键字来实现对父类方法的继承，公用方法根据其类型的不同划分为不同的`class`。

对API输出的思路可以参考这个[文章](https://www.jianshu.com/p/6b816c609669)，在格式化输出，url过滤，错误捕捉，日志打印都有诸多借鉴之处。

## 项目整体框架学习

　　目前涉及Node的开发有两种，一种是中间层开发(包括合并数据接口/服务端渲染) 一种是完整后端项目(包括数据库的调用和API接口)，原来的整体MVC架构已不是主流，虽然还有部分企业再用，但不利于快速迭代和开发，这里以这两种开发模式为例，简单谈一下框架搭建需要注意的点
　　如果是完整的后端项目，出API接口跟前端交互这种，其实其本质的框架应该跟传统后端语言对齐，比如JAVA和PYTHON等，需要注意的可扩展性和可容错性都是一脉相承的，这里推荐[这篇文章](https://www.cnblogs.com/hh54188/p/6371578.html),将后端的一些经典思想说的比较明白，其实Node到这一层，功夫不在语言本身，而在之外了。另外在github上有一篇针对后端架构设计比较好的[文章](https://github.com/focusaurus/express_code_structure)，用的是express框架，对代码的组织原则和协定写得非常到位，建议好好研究。
　　针对小型的中间层koa项目，我根据实际项目的运作经验和一些koa的思考，搭建了以下的代码组织架构，根据实际需要可灵活删减：

```sh
├── app.js          # 核心应用文件，这是整个后端应用的入口
├── bin             # 后端部署需要的shell脚本的存放位置
├── common          # 业务相关的公共模块
├── config          # 配置文件，可能有多种配置，如环境变量等
│   └── config.js
├── controller      # 逻辑控制器+处理接口路由的地方
│   ├── router1.js
│   └── router2.js
├── dispatch.js     # 主进程文件
├── lib             # 与业务无关的公共模块/方法
├── middleware      # koa中间件
├── node_modules    # 项目依赖
├── package.json    # 模块依赖表
├── test            # 测试脚本存放处
└── worker.js       # 工作进程文件
```

比较完整的项目还要引入数据库，也就是说这里要引入使用数据库的第三方中间件，针对数据库的架构目前思考尚不成熟，所以上面的架构没有设计数据库的操作，目前来想就是增加一个文件夹，里面放入针对所用数据库的一些ORM脚本和初始化建表的一些代码吧。

*PS：有些项目的middlerware是放在controller下面，作为控制器的一部分，也有其合理性。*