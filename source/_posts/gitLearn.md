title: 学习git常用及高阶命令
date: 2017-07-09 21:53:31
categories:
- 学习
tags:
- 工作
- git
---
git使用已经很久了，把常用的一些命令作为笔记总结如下
<!-- more -->
> 几个常用的git命令情景

- 复制代码仓库 `git clone --bare|--mirror|--non-bare /var/www/html/myrepo.git`
- 后悔药 覆盖最后一次修改 `git add . git commit --amend git push origin master -f`
- Git 放弃本地修改 `git checkout . && git clean -df`
- Git 销毁最后一次提交 `git reset --hard HEAD^ git push -f origin HEAD^:master`
- 打包嵌入版本号到文件 `git rev-parse HEAD > version.txt`
- 本地拉取远程git仓库 `git init && git remote add origin git@项目地址`
- PUSH前关联git的远程仓库 `git branch --set-upstream debug origin/debug`
- 建立已有关联关系的本地分支`git checkout --track  origin/dev-zhengqigit`

> 实用的高级Git命令(10条)

- 输出最后一次提交的改变 `git archive -o ../updated.zip HEAD $(git diff --name-only HEAD^)`
  *它会输出最近提交的修改类容到一个zip文件中。*
- 输出两个提交间的改变 `git archive -o ../latest.zip NEW_COMMIT_ID_HERE $(git diff --name-only OLD_COMMIT_ID_HERE NEW_COMMIT_ID_HERE)`
- 克隆 指定的远程分支 如果你渴望只克隆远程仓库的一个指定分支，而不是整个仓库分支，这对你帮助很大。
```sh
    git init
    git remote add -t BRANCH_NAME_HERE -f origin REMOTE_REPO_URL_PATH_HERE
    git checkout BRANCH_NAME_HERE
```
- 开始一个无历史的新分支 `git checkout --orphan NEW_BRANCH_NAME_HERE`
- 不想切换分支，但是又想从其它分支中获得你需要的文件 `git checkout BRANCH_NAME_HERE -- PATH_TO_FILE_IN_BRANCH_HERE`
- 同一branch协同工作，让git忽视某一指定文件的变动,防止merge覆盖 `git update-index --assume-unchanged PATH_TO_FILE_HERE`
- 检查提交的变动是否是release的一部分 `git name-rev --name-only COMMIT_HASH_HERE`
- 实用rebase替代merge完成推送
*但是在多团队成员共同工作于一条branch的情形中，常规的merge会导致log中出现多条消息，从而产生混淆。因此，您可以在pull的时候使用rebase，以此来减少无用的merge消息，从而保持历史记录的清晰。*
```sh
git pull --rebase
#将某条branch配置为总是使用rebase推送
git config branch.BRANCH_NAME_HERE.rebase true
```
- 检测 你的分支的改变是否为其它分支的一部分
*cherry命令让我们检测你的分支的改变是否出现在其它一些分支中。它通过+或者-符号来显示从当前分支与所给的分支之间的改变：是否合并了(merged)。.+ 指示没有出现在所给分支中，反之，- 就表示出现在了所给的分支中了*
```sh
git cherry -v OTHER_BRANCH_NAME_HERE
#例如: 检测master分支
git cherry -v master
```
- 检查提交的变动是否是release的一部分 `git name-rev --name-only COMMIT_HASH_HERE`

> 关于垃圾回收的 用来备忘

- Git 垃圾回收  `git gc --auto`
- Git 仓库占用空间  `$ du -hs .git/objects 45M .git/objects`
- 清理历史中的文件

```shell
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch ****/nohup.out' --prune-empty --tag-name-filter cat -- --all git filter-branch --index-filter 'git rm --cached --ignore-unmatch ****/nohup.out' HEAD git for-each-ref --format="%(refname)" refs/original/ | xargs -n 1 git update-ref -d
```