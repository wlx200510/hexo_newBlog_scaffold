title: 接触到的git的三个重要知识点
date: 2017-08-16 23:03:14
categories:
- git
tags:
- 总结
- 工作
---
通过剖析工作中常用的场景来探讨如何更灵活地使用git来完成任务~
<!-- more -->
## git配置多个SSH-Key

1. 生成第一个需要的SSH-Key(比如公司用的)

    `$ ssh-keygen -t rsa -C "emailname@company.com" -f ~/.ssh/id_rsa`
    这样就会在对应目录下生成`id_rsa`和`id_rsa.pub`私钥和公钥, 公钥里面的内容需要粘贴到公司服务器的ssh-key配置中

2. 生成一个github用的SSH-Key(第二个key)

    `$ ssh-keygen -t rsa -C "yourname@your.com" -f ~/.ssh/github_rsa`
    还在同一个路径里面，把公钥的内容粘贴到github服务器的SSH-key配置中

3. 在本机上添加两个私钥

    `$ ssh-add ~/.ssh/id_rsa`
    `$ ssh-add ~/.ssh/github_rsa`
    如果执行时提示"Could not open a connection to your authentication agent",则执行以下命令
    `ssh-agent bash` 再运行`ssh-add`命令

4. 修改(增加)配置文件
    在`~/.ssh`目录下新建`config`文件： `touch config`
    添加内容如下：(hostName是git地址最开头的那块：git@git.n.mifan.com:flash/flash-wap.git 就是git.n.mifan.com)
        ```bash
        # gitlab(根据不同公司配置来修改, 实质是对不同的git地址 指定不同的私钥)
        Host gitlab.com
            HostName gitlab.com
            PreferredAuthentications publickey
            IdentityFile ~/.ssh/id_rsa
        # github
        Host github.com
            HostName github.com
            PreferredAuthentications publickey
            IdentityFile ~/.ssh/github_rsa
        ```

5. 确认目录结构并测试
    `$ ssh -T git@github.com`
    输出You've successfully authenticated, but GitHub does not provide shell access字样 表示配置成功

## git使用cherry-pick

>主要的功能是提交过程的重演，从而可以灵活地调整commit的历程

1. 一种应用场景是在A分支的提交发现应该提交到B分支，在B分支上cherry-pick，A分支上git reset --hard 分支hash

    用法: `git cherry-pick <commit id>` 对已经存在的commit进行二次apply
    先`git log`，然后`git checkout old_b`, 在这个旧分支上进行`git cherry-pick 309u5j0438u0948v090948v5903w`(log中的hash值 可多个)

2. 要把dev-3.0分支上的某些更改移到dev-2.x的版本上, 产品开发的灵活上线需求

    首先新建一个要在其上应用cherry-pick的分支  `git checkout -b release-2.1 release-2.0`,
    将dev-3.0分支上的commit在release-2.1分支上重演 `git cherry-pick dev-3.0分支上的某些commit-hash`
    例如：
    ```bash
    git cherry-pick  
    20c2f506d789bb9f041050dc2c1e954fa3fb6910 
    2633961a16b0dda7b767b9264662223a2874dfa9 
    5d5929eafd1b03fd4e7b6aa15a6c571fbcb3ceb4
    ```
    多个commit-hash使用空格分割，commit-hash最好按照提交时间先后排列，最先提交的放在最前面

但要注意这个特性不要乱用 大部分可能使rebase使用情况不要用cherry-pick 会简单问题复杂化

## git链接远程仓库及取消

查看当前的远程仓库的命令: `git remote -v`

```bash
# 添加
git remote add origin git@url
# 删除
git remote rm origin
# 修改
git remote origin set-url git@url
```

## git指定文件更新为特定分支的版本

`git checkout` 指定分支 指定文件

```bash
git checkout master src/index/index.mix
```