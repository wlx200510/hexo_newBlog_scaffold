---
title: SHELL脚本的学习和制作
date: 2017-3-20 10:36:00
categories:
- 学习
tags:
- 总结
- 工作
---
因前端工程的自动化构建，在webpack打包后需要将index.html、dist和static三个文件上传到部署的linux环境中，之前一直是用windows通过tr上传到跳板机而后scp到发布机器上，近来部署了jenkins，可直接在jenkins部署的机器上进行shell脚本的触发自动化操作
<!-- more -->
### 本地的PACK构建shell脚本

首先是在在package.json中的scripts键值中加入release

`"release": "sh build/pack.sh"`

之后在项目目录下新建build文件，并在其中新建pack.sh文件，内容如下：

```bash
#!/bin/sh
if [ ! "$1"]; then
    echo "不可缺少打包版本号"
    exit 1
fi
# 上面的命令是确认传来了作为版本号的第一个参数
echo "编译中....."
npm run build
# "build": "cross-env NODE_ENV=production gulp",为gulp的命令
echo "压缩打包中....."
mkdir -p output
# -p指即便是有当前目录，建立也算成功
echo "$1" > output/version.txt
tar zcvf qmt.tgz dist/ index.html static/
mv qmt.tgz $1.tgz
mv $1.tgz output
# 打包两个文件夹的内容和index.html 并把打包后的文件移动到output中
echo "打包完毕"
echo "提交、添加tag, 并push到gitlab"
git add output/
git add src/components/_global/footer/index.vue
# index.vue更改了下角标的版本号，需要重新添加保存
git commit -a -v -m 'new release'
git tag v"$1"
git push -u origin release --tags
# 把新的tag推送到仓库
```
这就是执行了npm run release后执行的一系列命令。

## 使代码提交时能自动触发jenkins构建脚本

先在jenkins中建立一个任务 点击左上角新建  建立一个新的project(本次是直接复制的别人已建立好的配置)，取名为test(推送到测试环境) 在打开的配置页面中有如下几点需要注意

- 项目名称核对是否正确。
- 勾选参数化构建过程，将服务器的地址设置为变量，在脚本中可直接读到($TEST_SERVERS)
- 源码管理项，选择git，填写当前项目的git地址，Branch(refs/heads/release) Local subdirectory for repo下写跟后端约定的文件夹名称
- 构建触发器这里，选择Build when a change is push to GitLab 这里需要注意，要配置好项目的gitlab仓库的webhook，地址就是这个后面跟着的URL，同时下面的Target Branch选择release分支。
- 构建触发器选择后，一些触发选项根据需要来确定，然后到项目的gitlab页面，点击右上角的配置-webhook项，第一个对话框写入上面提到的URL，选择push和push tags两个一般就可以完成触发
- 在构建的commend对话框，写入触发脚本的shell命令如下, test.sh是接下来要写的shell脚本
```
cd $WORKSPACE/test
sh build/test.sh
```

## 写test.sh脚本

脚本的目标是基于jenkins所在的机器，运行此脚本，将打好的包传输到部署机器上，解压到所需的文件夹下，并把原压缩包删除。

```bash
#!/bin/sh
echo $TEST_SERVERS
# 打开目标文件目录
cd $WORKSPACE/test
# 读取当前包的版本号
VERSION=`cat ./output/version.txt`
echo "版本号: "$VERSION
scp output/qmt.tgz root@$TEST_SERVERS:/tmp && \
# 通过scp命令把打好的output文件夹下的包传输到目标机器的tmp文件夹下
ssh root@$TEST_SERVERS 
#连接到目标机器上，默认是在root下 接下来要执行的命令需要放在双引号中！
# 先打开最顶层的目录，在最顶层的目录进行接下来的操作
# tar命令的zxvf对应解压 -C可指定解压到的文件夹 rm为删除命令
"cd ../ && tar zxvf ./tmp/qmt.tgz -C ./tv/le/*-static && \
tar zxvf ./tmp/qmt.tgz -C ./tv/le/staging-*-static && \
rm -rf ./tmp/qmt.tgz"
```

## 学习发布用的upload脚本

upload脚本是将代码发布到两台机器上，因此$DEPLOY_SERVERS是一个两个IP地址用逗号分隔的字符串，先分隔为数组，而后进行循环部署处理。还有一点是通过读入的版本号来建立对应版本的文件夹，而后用软连接来指向，方便出问题后回滚版本。

```bash
#!/bin/sh
echo $DEPLOY_SERVERS
VERSION=`cat ../output/version.txt`
echo "版本号: "$VERSION
cd $WORKSPACE/lesports-fe-cms
# 下面这个是讲$DEPLOY_SERVERS用','分隔为数组并存入ADDR
IFS=',' read -ra ADDR <<< "$DEPLOY_SERVERS"
# 循环数组读取每一个变量
for i in "${ADDR[@]}"; do
    # 先通过scp把包上传到要部署的机器上
    scp output/qmt.tgz root@$i:/tmp
    ssh root@$i "mkdir -p /letv/leapps/tags && \
    cd /letv/leapps/tags && \
    mkdir -p qmt.$VERSION && \
    cd qmt.$VERSION && \
    tar zxvf /tmp/qmt.tgz -C . && \
    rm -f /tmp/qmt.tgz && \
    ls -rthl && \
    cd ../../ && \
    rm -fr qmt-cms-static && \
    ln -s tags/qmt.$VERSION qmt-cms-static
    "
done
# mkdir -p qmt.$VERSION 为建立带版本号的文件夹(在tags文件夹下)
# cd进入带版本号的文件夹,把包解压到当前文件夹下
# ls -rthl 为显示目录内容列表
# 回退到 /letv/leapps文件夹中 此处已有qmt-cms-static这个文件夹，ln -s tags/qmt.$VERSION qmt-cms-static 就是把这个文件夹软连接到最新的带tag的文件夹上。
```

## 优化脚本-取消版本号并增加错误提示
```bash
#!/bin/sh
if [ ! "$1" ]; then
    echo "版本号是必须的"
    exit 1
fi
# 用于当本地的tag已存在时的提示信息
curTag=`git tag | grep v$1`
if [ "$curTag" == "v$1" ]; then
    echo "版本号已存在"
    exit 1
fi
#
echo "DIST编译中....."
npm run dist
#
echo "压缩打包中......"
mkdir -p output
tar zcvf qmt.tgz dist/ index.html static/
mv qmt.tgz output
echo "打包完毕"
echo "提交、添加tag，并push到gitlab"
git add output/
git add src/components/_global/footer/index.vue
git commit -a -v -m "new release v$1"
git tag -a v"$1"
# 捕捉错误并提示出错信息
git push -u origin release || { echo "本地推送失败，请pull后再执行"; exit 1; }
git push origin v"$1" || { echo "tag push failed"; exit 1; }
```

*这些shell脚本的难度都不是很大，主要是练习，毕竟也不是专业的运维，否则最起码也要有定时执行一类的，不过工作需要的运维东西不算多，慢慢积累吧。*