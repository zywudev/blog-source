---
title: Git 命令备忘
date: 2017-03-15 10:17:30
tag: Git
categories: Git
---
对 Git 常用命令归类总结，方便查阅。
## Git 配置
```java
git config --global user.name "zywudev" # 全局用户名

git config --global user.email "zywu.dev@gmail.com" # 全局邮箱

git config user.name "zywudev" # 某一个项目使用特地的用户名 

git config user.email "zywu.dev@gmail.com" # 某一个项目使用特地的邮箱

git config --global alias.co checkout # 别名

git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%
d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative" # 日志

git config --global color.ui true # git 着色

git config --global core.quotepath false # 设置显示中文文件名
```
## Git 常用命令
```java
git # 查看 git 是否安装成功

git init # 初始化仓库

git status # 仓库状态

git add <file> # 将文件添加到暂存区

git add . # 将所有修改过的文件添加到暂存区

git rm --cached  # 删除缓存，但不删除文件

git rm <file> # 删除文件

git commit -m “first commit” # 提交

git log # 可以查看所有产生的 commit 记录

git tag v1.0 # 给当前代码打标签

git tag # 查看历史 tag 记录

git checkout v1.0 # 切换到 v1.0 的代码状态

git diff <$id1> <$id2> # 比较两次提交之间的差异

git diff <branch1>..<branch2> # 在两个分支之间比较

git diff --staged # 比较暂存区和版本库差异

git checkout -<file>   # 抛弃工作区修改

git checkout . # 抛弃工作区修改

git stash # 当前分支暂存

git stash list # 列所有stash

git stash apply # 恢复暂存的内容

git stash drop # 删除暂存区（删除一条）

git stash clear # 清空暂存区

```
## 分支管理
```java
git branch # 查看分支情况

git branch a # 新建 a 分支

git checkout a # 切换到 a 分支

git checkout -b a # 新建 a 分支 自动切换到 a 分支

git merge a # 将 a 分支合并到当前分支

git branch -d a # 删除 a 分支，a 分支已被合并

git branch -D a # 删除 a 分支，a 分支未被合并

git branch -r # 查看远程分支列表

git push origin:<branch> # 删除远程分支

git checkout a origin/a # 本地没有 a 分支的情况下，将远程 a 分支迁到本地

git checkout -b a origin/a # 把远程分支迁到本地顺便切换到该分支：
```
## 远程管理
```java
git push origin master # 把本地代码推到远程 master 分支

git pull origin master # 把远程 master 代码更新到本地

git clone git@github.com:zywudev/weather.git # 远程 test 项目 clone 到本地

git remote add origin git@github.com:zywudev/test.git # 本地已用项目与远程test项目关联，提交到远程test项目

git remote -v # 查看当前项目有哪些远程仓库

git remote show origin # 查看远程服务器仓库状态
```
