---
title: 类vue-cli中webpack模板的多页+elementUI模板
date: 2018-01-11 10:46:20
categories:
- 学习
tags:
- 学习
- Vue
---
在网上找vue-cli的多页模板 然后在引入各种组件库总是有问题，并且各个配置文件说明都不同，于是参照官方`vue-cli`来做了同配置的多页面脚手架出来。
<!-- more -->
<!-- excerpt -->

> 本脚手架并且适当简化了一些功能（删除了测试库） 并引入了外部mock和axios两个常用库可以灵活配置。 这里写一下说明文件和心得体会

## 配置功能

最基本的功能为`webpack3+Vue2`的基础上引入了外部组件库`elementUI` 其实也可以灵活修改为别的，`css`的支持仅引入了`less`和`sass`，相信这两者用的人也是最多的。还有一点是针对多页面也引入了`vue-router`, 也就是说这个多页面仓库也可以当单页面来搞起。
加入的`axios`库是本地业务所需，这个可以在生成脚手架时不选择，但这个作为Vue的推荐库，建议尽量用这个，坑比较少。

1. [Vue2](https://github.com/vuejs/vue)
2. [Webpack3](https://github.com/webpack/webpack)
3. [ElementUI](https://github.com/ElemeFE/element)
4. [Eslint](https://github.com/eslint/eslint)([eslint-config-vue](https://github.com/vuejs/eslint-config-vue) default)
5. [Postcss](https://github.com/postcss/postcss)([autoprefixer](https://github.com/postcss/autoprefixer) default)
6. [Less](http://lesscss.org/)
7. [Sass](https://github.com/webpack-contrib/sass-loader)
8. [VueRouter](https://github.com/vuejs/vue-router)
9. [mock.js](https://github.com/nuysoft/Mock)
10. [axios](https://github.com/axios/axios)

## 使用方法
相信看到这篇文章的人对`vue-cli`的使用比较熟练了，有需要补课的小伙伴戳[这里](https://github.com/vuejs/vue-cli)

``` bash
$ npm install -g vue-cli
$ vue init wlx200510/vue-multiple-pages-cli new_project
$ cd new_project
$ npm install
```
开发流程：
```bash
# serve with hot reload at localhost:8060
$ npm run dev
```
打包流程
```bash
$ npm run build
$ node server.js #listen 2333 port
```

## 文件架构

```bash
├── build
│   ├── build.js # build entry
│   ├── utils.js # tool funcs
│   ├── webpack.base.conf.js
│   ├── webpack.dev.conf.js
│   └── webpack.prod.conf.js
│
├── config
│   ├── index.js  # config index settings
│   ├── dev.env.js # dev env
│   └── prod.env.js # prod build env
│
├── src  # main folder
│   ├── assets  # common assets folder
│   │   ├── img
│   │   │   └── logo.png
│   │   ├── js
│   │   └── css
│   ├── components # common components folder
│   │   └── modal.vue
│   └── pages  # pages
│       ├── user  # user part (folder name can be customized)
│       │   ├── login # login.html (folder name can be customized)
│       │   │   ├── app.js   # entry js (file name can't be customized unless you change the webpack.config.js)
│       │   │   ├── app.vue  # login vue (file name can be customized)
│       │   │   └── app.html # template html (file name can't be customized unless you change the webpack.config.js)
│       │   └── index # index.html
│       │       ├── app.js
│       │       ├── app.html
│       │       └── app.vue
│       └── customer # customer part (folder name can be customized)
│           └── home # home.html
│               ├── app.html
│               ├── app.js
│               ├── app.vue
│               ├── mock
│               │   └── index.js # mock.js to mock API
│               ├── router
│               │   └── index.js # vue-router use example
│               └── selfComponents
│                   └── notFound.vue # components example with vue-router
├── LICENSE
├── .babelrc          # babel config (es2015 default)
├── .eslintrc.js      # eslint config (eslint-config-vue default)
├── server.js         # port 2333
├── package.json
├── postcss.config.js # postcss (autoprefixer default)
├── webpack.config.js
└── README.md
```

## 具体细节

本仓库的[具体地址](https://github.com/wlx200510/vue-multiple-pages-cli)
多页面入口的设置是参照`element-starter`来做的，特点是文件目录结构一定是要遵循上述规定，具体参考`github`中的`README`文档
项目的配置细节大部分都在config目录下，熟悉`vue-cli/webpack`模板的应该都很容易看懂，因为只多了一项`openPage`其余基本相同

## 编写模板体会

1. 通过双大括号来处理文本的渲染。
2. 编写`meta.js`用于用户生成项目前的交互和提示。
3. `webpack`生成两份分别用于开发环境和打包环境的架构设计很合理。
4. 配置文件单独列出，所有的配置与具体的`webpack.conf`文件解耦。
5. 最难处理的莫过于文件路径问题，这个需要多次尝试，有耐心才行。
