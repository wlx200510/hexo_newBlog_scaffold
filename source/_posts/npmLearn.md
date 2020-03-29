---
title: 学会发布自己的npm包
date: 2017-12-26 09:20:55
categories:
- 学习
tags:
- 工作
---
最近学习了下发布一个npm工具包的流程，记录下来，方便大家参考学习
<!-- more -->
<!-- excerpt -->
## 准备工作
- 在[npm官网](https://www.npmjs.com/)注册账号，要记住用户名/邮箱/密码 三个信息，后面需要用到
- 需要发布的项目应该有相应的git地址 包括但不限于`github`
- 最好在项目中使用gulp来把跑测试用例和打包的过程进行自动化，方便发布

## 操作流程
1. 在修改完成后，首先本地提交，并执行单元测试(`npm run unit`)
2. 如果测试通过，则可以打包出压缩后的成品代码(`npm run build`)
3. 此时进入预发布流程，可以先通过`npm whoami`来确认登录信息，如果当前未登录，则需要`npm login`来输入用户名/密码/邮箱来登录
4. 确定处于登录状态后，`npm version from-git`把本地的`npm`包版本号更新到最新
5. 使用`npm version patch`来增加新一期的版本号，实质是打了一个本地的`tag`
6. 成功后表明完成了本地所需的预发布流程，在发布之前先通过`git commit -am 'version info' && git push && git push --tags`同步到远程
7. 执行`npm publish`将`npm`最新版本的包进行发布。

## 同步最新版本
需要在用到此npm包的项目中的package.json中更改此包的最新版本号，然后执行`npm i`来更新包的代码 之后就可以在最新包的基础上进行开发
