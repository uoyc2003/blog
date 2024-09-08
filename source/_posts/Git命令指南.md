---
title: Git命令指南
tags:
  - Git
  - 命令指南
categories:
  - 编程笔记
headimg: 'https://pic.zeng.cyou/wide/202409042025523.webp'
abbrlink: 25456
date: 2024-09-08 11:08:31
---

## 常用操作

|                 命令                 |                     描述                     |
| :----------------------------------: | :------------------------------------------: |
|              git init                |      在当前目录下初始化一个新的Git仓库       |
|         git clone <远程仓库>         |        从远程仓库克隆一个新的本地仓库        |
|           git add <文件名>           |          将工作区的修改添加到暂存区          |
|       git commit -m "<提交说明>"      | 将暂存区的内容提交到本地仓库，并附上提交说明  |
|             git status               |           显示工作区和暂存区的状态           |
|              git log                 |            查看仓库的提交历史记录            |
|             git diff                 | 显示工作区与暂存区的差异或两个提交之间的差异 |
|            git branch                |                 列出本地分支                 |
|          git branch <分支名>         |                 创建一个新分支               |
|         git branch -d <分支名>       |   删除指定的本地分支（仅删除已经被合并的分支）   |
|         git branch -D <分支名>       |      强制删除指定的本地分支（即使未被合并）      |
|         git checkout <分支名>        |                切换到指定分支                |
|           git merge <分支名>         |        将指定分支的代码合并到当前分支        |
|             git pull                 |        拉取远程仓库最新代码到本地分支        |
|             git push                 |        将本地分支的代码推送到远程仓库        |
|       git push <远程仓库> <分支>     |        将本地分支的代码推送到远程仓库        |
|            git remote                |               显示远程仓库列表               |
|   git remote add <名称> <远程仓库>   |            添加一个远程仓库到当前仓库        |
|          git remote -v               |             显示所有远程仓库及其URL          |
| git remote set-url origin <远程仓库> |                修改远程仓库的URL             |
|     git remote remove <仓库名称>     |                删除指定的远程仓库            |
|              git tag                 |                  列出仓库的标签              |
|  git tag -a <标签名> -m "<标签说明>" |                创建带有说明的标签            |
|          git tag -d <标签名>         |                  删除指定的标签              |
|             git stash                | 将当前工作区的未提交修改保存到栈中，并清理工作区 |
|           git stash pop              |       应用最近一次stash的内容并从栈中移除    |
|         git rebase <分支名>          |       将当前分支的提交重新应用到指定分支上   |
|       git cherry-pick <提交ID>       |            将指定的提交应用到当前分支        |
