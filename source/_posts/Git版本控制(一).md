---
title:  Git版本控制(一)
tags: 
- Git
categories: 
- Git
date: 2019-8-18
---
# Git版本控制(一)
**为什么要写这个系列？**

作为一个IT从业者，无论是开发还是运维，熟练使用版本控制应该算是一项基本技能。而Git又是目前最流行的一款版本控制工具，所以对Git进行深入且系统的学习是非常有必要的。据我日常使用经验和观察来看，可能大部分人使用Git应该还停留在`pull`、`push`、`add`、`commit`等一些基础操作层面。（包括我自己- -）我之前琐碎的整理过一些Git的笔记，但是不够系统，这次想写几篇博客系统的整理和学习下Git的一些高级用法。方便自己日后回看的同时希望能够帮到有同样需求的人。原理和概念性的东西我就不多介绍了，很多大佬写的比我通俗易懂，我还是多写写实际使用相关的内容。
## 为什么要使用版本控制？

VCS出现以前，研发人员经常需要手动控制版本演变，比如说建立一个文件共享服务器，在服务器上手动创建版本目录并拷贝相关文件来区分不同的版本。这样做带来的问题是：容易造成公共文件的覆盖,并且在进行多人合作项目时，各个成员之间沟通成本很高，代码集成效率很低。
## 版本控制的类型
(1) 集中式版本控制  
典型代表：SVN  
特点： 有集中的版本控制服务器、具备版本控制和分支管理能力、本地客户端不具备完整的版本库，需要时刻和服务器相连。  

(2) 分布式版本控制      
典型代表: Git （据说是Linux之父Linus度假的时候随手写的,大佬就是大佬）  
特点: 服务端和客户端都具有完整的版本库、客户端脱离服务端也可以进行版本管理
## 基础操作
### 安装Git  
可以参考[Git官方文档](https://git-scm.com/book/zh/v2)，官方文档有多种语言可以选择，包括中文。文档中介绍了Linux、Mac OS、Windows三种常用系统的详细安装方法。
### 最小配置
配置user信息
```bash
$ git config --globle user.name "your_name"
$ git config --globle user.email "your_email"
```
**配置user信息的好处**：一是可以看到每次的变更提交作者是谁，二是在Code review等场景下，可以Gitlab/Github等VCS系统可以实时通过邮箱将相关信息发送至变更提交者。  

**Config的作用域**

大多数教程只会告诉你`git config --globle`这个操作，这个配置也是工作中最常用的配置，但是config一共有三个作用域可以配置，也可以了解下。分别是：
```bash
$ git config --local  # 只对某个仓库有效
$ git config --global # 对当前用户所有仓库有效,
$ git config --system # 对系统所有登录的用户有效

优先级： local > global > system
```
**显示不同config的配置**
```bash
$ git config --list --local  
$ git config --list --global 
$ git config --list --system
```
### 建Git仓库
创建Git仓库一般有一下两种场景：  
(1) 对已有的项目进行版本控制
```
$ cd 项目代码所在的文件夹
$ git init
$ git remote add remote remote-address # 关联远端仓库
```
(2) 新建的项目直接使用版本控制
```
$ cd 一个文件夹
$ git init project_name # 这一步的操作会在当前路径下创建和项目名称同名的文件夹
$ cd project_name
$ git remote add remote remote-address # 关联远端仓库
```
### 文件管理
(1) 向仓库中添加文件
``` 
git add filename                     # 添加单个文件到本地暂存区
git commit -m"fix bug"               # 将缓存区内容添加到仓库
git commit --amend  "fix file bug"   # 修改最近一次commit提交的注释
git push                             # 推送到远端仓库
```
(2) 文件快速重命名
```
git mv oldfilename  newfilename
```
(3) 删除操作  
- 要从 Git 中移除某个文件，就必须要从已跟踪文件清单中移除，然后提交
```
git rm filename
```
- 如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项 -f
```
git rm -f filename
```
- 如果把文件从暂存区域移除，但仍然希望保留在当前工作目录中，换句话说，仅是从跟踪清单中删除，使用 --cached 选项即可
```
git rm --cached filename
```
(4) 比较操作
```
git diff --cached     # 比较暂存区和HEAD指向的内容的差异
git diff              # 比较工作区和暂存区的所有差异
git diff -- filename  # 比较工作区和暂存区指定文件的差异
```
(5) 恢复操作
```
git reset HEAD               # 让暂存区恢复成和HEAD一样
git reset HEAD  -- filename  # 让暂存区指定文件恢复成和HEAD一样
git checkout -- filename     # 工作区内容恢复成暂存区           git reset --hard  commit id  # 消除最近几次commit提交，恢复到指定commit(HEAD会变，暂存区和工作区都恢复至指定commit的内容，慎用！！！！)        
```
(6) 屏蔽不需要Git管理的文件  
通过`.gitignore`文件进行屏蔽，该文件支持文件名和*通配符。
### 版本历史查看
```
git log  --online  # 简介输出版本历史
git log -n2        # 查看最近两次的commit
git log  --online -n4 # 简洁输出最近4次的commit
git log -all       # 查看所有分支的历史，git log 默认只输出当前分支
git log --graph    # 简易图像化查看

以上各个参数可以互相组合使用
```
## 分支操作
```
git checkout -b branchname  # 创建新分支并切换到新分支
git checkout branchname     # 切换到指定分支
git branch  -d branchname   # 删除指定分支
git branch  -D branchname   # 强制删除指定分支
git branch  -av             # 列出当前仓库的所有分支
```

