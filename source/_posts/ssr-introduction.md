title: 使用SSR来模拟后端mock数据
date: 2017-03-11 10:10:08
categories:
- 工具
tags:
- Javascript
- 后端
---
将一个目录设置成一个静态服务器，ssr相当于是搭建了一个 Mock Server ，构建假数据，然后把这些假数据存到 JSON 文件上，Mock Server 可以响应请求或者生成页面，当然也可以顺便生成 API 文档。
<!-- more -->
### 特点
- 可强制跨域访问（ajax不报错，方便调试）
- 可启动多个服务，端口冲突可自动顺延
- 可设定指定端口号

### 使用方法

*全局安装:*
`npm install -g ssr`

*命令示例:*
`$ssr   默认端口为1987  访问地址为=> http://localhost:1987`
`$ssr -p 2016   端口设置为2016  访问地址为=> http://localhost:2016`
`$ssr -cp 2017  端口设置为2017  设置为可跨域访问`

*在项目中的使用指南*
1. 建立一个用于mock数据的文件夹mock1  `touch mock1`
2. 在mock1中放入mock数据的file文件，内容是json数据  `cd mock1 && touch file.json`
3. 在mock1中用vscode打开json文件，并粘入mock数据  `code file.json`
4. 在mock1文件夹中启动ssr  `ssr -cp 2017`
5. 在项目中就可以使用ajax对file中的内容进行请求(前提是项目也是运行在一个本地服务器上)

*ajax请求示例代码*

```
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>测试ssr的性能</title>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <script src="http://apps.bdimg.com/libs/jquery/2.1.1/jquery.min.js"></script>
    </head>
    <body>
    </body>
    <script>
        $.ajax({
            type: 'POST',
            datyType: 'json',
            data: { 'name': 'karl', 'old': 10 },
            url: 'http://127.0.0.1:2017/file.json',
            success: function(data) {
                console.log('success:', data);
            },
            error: function(error) {
                console.log('error', error);
            }
        })
    </script>
</html>
```

### 加上逻辑的MOCK数据

新建一个proxy.config.js
此文件内容如下：
```
module.exports = {
    'GET /users': {{name: 'kenny wang'}, {name: 'karl}},  // 对应请求是http://localhost:2017/users method='GET'
    'GET /user1': {name: 'karl api'},
    'POST /users': {name: '???text'},
    'POST /users/2: "22222223",
    'POST /users': function(data,url) {
        // data 为接受传递来的数据
        // url 请求的地址
        // 需接受的参数  - form-data   - x-www-form-urlencoded  - raw
        if (data.name === 'jslite') {
            return {name: 'kkkarl'}
        } else {
            return {name: 'yyy'}
        }
    }
};
```

此文件的使用方法：

`$ ssr --proxy proxy.config.js -cp 2017`

这样只要约定好数据格式, 后端端口不会影响前端调试, Mock Server 可以相应请求

> PS: 前提是开发页面的web环境必须是有服务器的~否则没法跨域发送ajax的。
