---
title: git总结
date: 2022-06-13 21:14:59
tags:
  - git
  - 总结
---

Git 是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。Git 与常用的版本控制工具 CVS, Subversion 等不同，它采用了分布式版本库的方式，不必服务器端软件支持。

Git 一般只添加数据, 你执行的 Git 操作，几乎只往 Git 数据库中增加数据。 很难让Git 执行任何不可逆操作，或者让它以任何方式清除数据。 同别的 VCS 一样，未提交更新时有可能丢失或弄乱修改的内容;但是一旦你提交快照到 Git 中，就难以再丢失数据，特别是如果你定期的推送数据库到其它仓库的话。

<!--more-->

![1595228461@2ffee7bd1f90f29273dadb9cb5b06440.jpg](/Users/zeki/Documents/Blog/source/_posts/git总结/1595228461@2ffee7bd1f90f29273dadb9cb5b06440.jpg.png)

- Workspace：工作区

- Index/Stage：暂存区，也叫索引

- Repository：仓库区（或本地仓库），也存储库

- Remote：远程仓库



这使得我们使用 Git 成为一个安心愉悦的过程，因为我们深知可以尽情做各种尝试，而没有把事情弄糟的危险。 更深度探讨 Git 如何保存数据及恢复丢失数据的话题，请参考撤消操作。



1. 历史操作记录

   ```bash
   git log  # 查看历史操作记录
   git log --oneline  # 简洁版本的历史操作记录
   git log --graph  # 拓扑图选项
   git log --reverse  # 逆向显示日志
   git log --auther=auther  # 查看auther的提交
   # 指定日期，可以执行几个选项：--since 和 --before，也可以用 --until 和 --after。
   ```

   

2. 查看当前仓库状态

   ```bash
   git status  # 查看当前仓库状态
   ```

3. 推送

   ```bash
   git add *  # 添加至暂存区
   git commit -m "commit message"  # 提交
   git push origin master  # 推送至远程仓库master分支
   ```

   

4. 更新与拉取

   ```bash
   git clone http://xxx.com/xxx.git  # clone远程仓库
   git pull  # 获取新更改
   ```

5. 暂存区保留临时更改

   ```bash
   git stash  # 本地更改暂存 
   git stash list  # 列出本地暂存数据
   git stash pop  # 恢复
   ```

6. 移动

   对于仓库的文件做移动

   ```bash
   git mv xxx yyy  # xxx移动/重命名到yyy, git文件前显示R表示已被重命名
   git commit -a -m "rename xxx to yyy"  # -a标志，这使git commit自动检测修改的文件
   ```

7. 删除

   ```bash
   git rm xxx  # 删除xxx文件
   git commit -a -m "remove file xxx"
   ```

todo: http://www.bjpowernode.com/tutorial_git/1775.html

