---
title: git push错误
date: 2021-12-28 14:55:59
tags:
---
# 错误一
## error log
``` bash
$ git push -u origin master
```
```
fatal: AggregateException encountered.
   ▒▒▒▒һ▒▒▒▒▒▒▒▒▒▒
To https://github.com/upon88/upon88.github.io.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'https://ghp_tcsNXkQEnviUzMhnt68H6MSA4QL9bL0E                                     HsFR@github.com/upon88/upon88.github.io.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
## 原因
``` bash
git init    初始化本地仓库
git remote add origin XXX     添加远程仓库地址
```
如果你在这之后就执行
``` bash
git add -A，
git commit -m " "
git push origin master
```
那么就会出现这个问题(被拒绝)，所以在remote add后不要着急git add，一定要git pull origin master；
因为在码云创建的仓库有ReadMe文件，而本地没有，造成本地和远程的不同步。
## 解
### 方法一
本地没有ReadMe文件，那么就在本地生成一个：
``` bash
git pull --rebase origin master     本地生成ReadMe文件
git push origin master
```
### 方法二
强制上传覆盖远程文件
``` bash
git push -f origin master
```
备注：这个命令在团队开发的时候最好不要用,否则可能会有生命危险
