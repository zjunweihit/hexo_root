---
title: How to setup Hexo
date: 2016-05-22 22:41:01
categories: linux-env
tags:
  - linux-env-misc
---

This document describes how to setup hexo on your system.

<!-- more -->

# Ubuntu 16.04

* Install nodejs
```
sudo apt-get install nodejs
```
* Install npm
```
sudo apt-get install npm
```
* Update the source for npm, appeding below to `~/.npmrc`
```
registry = https://registry.npm.taobao.org
```
* Install hexo, to show detail with `--verbose`
  * Run it with bash
```
sudo ln -sf /usr/bin/nodejs /usr/bin/node
sudo npm install --unsafe-perm -g hexo
```
* Test hexo
```
hexo --help
```
* Install hexo plugin
```
npm install hexo-deployer-git --save
```

# Mac

* Install nodejs and npm
  * get from [hexo--搭建](http://www.jianshu.com/p/a2023a601ceb)
* Update the source for npm, appeding below to `~/.npmrc`
```
registry = https://registry.npm.taobao.org
```
* Install hexo
```
sudo npm install -g hexo
```
* Test hexo
```
hexo --help
```
* Install hexo git plugin
```
cd folder where executes "hexo init"
npm install hexo-deployer-git --save
```

# How to use hexo

* Initialization
```
hexo init
```
* Create a new file
```
hexo n #new
```
* Generate files
```
hexo g #generate
```
* Start server to access locally, at `localhost:4000` or `<IP>:4000` at another PC
```
hexo s #server
```
* Deploy to the git repo
```
hexo d #deploy
```

## Reference
* [简洁轻便的博客平台: Hexo详解](http://www.tuicool.com/articles/ueI7naV)

# How to setup with github(zjunweihit as example)

* Initialize blog
```
mkdir blog
cd blog
hexo init
```
* Update files in root path
```
git clone https://github.com/zjunweihit/hexo_root.git
cp -a hexo_root/* ./
cp -a hexo_root/.gitignore ./
cp -a hexo_root/.git ./
```
* Update the theme
```
cd themes
git clone https://github.com/zjunweihit/jacman.git
```

# Reference
* [hexo--搭建](http://www.jianshu.com/p/a2023a601ceb)
* [使用GitHub和Hexo搭建免费静态Blog](http://www.jianshu.com/p/3c3991658f95)
* [hexo theme](https://hexo.io/themes/)
* [ubuntu15.04安装hexo](http://www.linuxdiyf.com/linux/18320.html)
