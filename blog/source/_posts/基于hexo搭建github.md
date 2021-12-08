---
title: 基于hexo搭建github
date: 2021-12-03 11:18:04
tags:
---
# 一、必备软件

## 1、安装Node.js
a、[下载Node.js](https://nodejs.org/dist)
b、确认安装
``` bash
$ node -v
$ npm -v
```
## 2、安装Git
a、[下载Git](https://github.com/git-for-windows/git/releases/)
b、确认安装
``` bash
$ git --version
```

## 3、安装hexo
a、打开cmd命令行(win+r 输入cmd回车)
``` bash
$ npm install hexo-cli -g
```
b、确认安装
``` bash
$ hexo -v
```
## 4、安装markdown工具
[下载Atom](https://atom.io/)

# 二、常用命令
## hexo配置相关命令
新建文件夹，然后右键
``` bash
$ git bash here
$ npm install hexo --save
$ hexo init
$ npm install
$ hexo g
$ hexo s
```
## git命令
``` bash
$ git init
$ git remote add origin https://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@github.com/upon88/upon88.github.io.git
$ git push origin master
```

# 三、windows系统上配置ssh key 的生成
a、key的生成
``` bash
$ git config --global user.name "yourname"
$ git config --global user.email "youremail"
$ git config user.name
$ git config user.email
$ ssh-keygen -t rsa -C "youremail"
```
b、测试是否成功
``` bash
$ ssh -T git@github.com
```

# 四、在当前文件夹配置github仓
根_config.yml中配置自己的远程仓库地址
``` bash
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  repo: https://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@github.com/upon88/upon88.github.io.git  ##这里是需要配置为自己的token、用户名及仓名。
  branch: main
```
# 五、主题
主题切换，直接在github搜索就可以找到，再根据说明步骤一一操作。

# 六、注意事项
a、在github中新建仓的名字一定要与用户名一致；
b、切换新文件夹或者切换电脑时，一定要重新去生成token。
