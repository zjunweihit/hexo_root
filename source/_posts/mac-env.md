---
title: Mac 环境那些事
date: 2016-07-02 12:59:34
tags:
  - linux-env-misc
---

This doc describes the MAC environment.

<!--more-->

* Modify the hostname
```
sudo scutil --set HostName MacBookPro
```
* Set the computer name
```
sudo scutil --set ComputerName MacBookPro
```
* Install homebrew [homebrew](https://brew.sh)
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

brew update
brew search "name"
brew install "name"
brew remove "name"
```

# shortcut
* screenshot
  * fullscreen
```
command + shift + 3
```
  * screenshot for self-defined area
```
command + shift + 4
```

# Jekyll setup
* Install Jekyll with gem
```
gem installl jekyll
```
  * Maybe you will met the issue, caused by unaccessable source of "https://rubygems.org/"
```
ERROR:  Could not find a valid gem 'jekyll' (>= 0), here is why:
          Unable to download data from https://rubygems.org/ - Errno::EPIPE: Broken pipe - SSL_connect (https://rubygems.org/latest_specs.4.8.gz)
```
  * You can try to use the alternative source of taobao
```
gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/
gem sources -l
*** CURRENT SOURCES ***

https://ruby.taobao.org/
```
* Build the html files in _site directory
```
jekyll build
```
* Start jekyll server for access locally
```
jekyll server
```
* Access the html file in browser
```
http://localhost:4000
```

## reference
* [Mac下Jekyll安装](http://www.jianshu.com/p/07064eb79740)
* [CocoaPods安装经历](http://blog.csdn.net/th_gsb/article/details/49178083)

# Upgrade to macOS sierra
* 好像新的macOS与老的系统兼容不太好，好多软件都要重新安装，而且安装方法也有些不太一样。
* `openconnect`，具体参考openconnect安装更新页面。
* `homebrew`
  * re-install from master branch(previously stable branch), in sierra, install brewhome from master branch by default.
  * remove old homebrew
    - ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"
  * install from master by default in macOS
    - /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
  * reference: [brew home page](https://brew.sh)
  * switch branch manually, referring to [macOS sierra brew权限问题](https://segmentfault.com/q/1010000007194095)
```
cd "$(brew —repo)" && git fetch && git reset —hard origin/master && brew update
sudo chown -R $(whoami) /usr/local  (if run into permission)
```
* `xcode`
  * 升级后很多软件不好用，e.g. git找不到路径, 编译openconnect找不到库
  * reset/install xcode [mac更新系统后Git不能用，提示missing xcrun](http://blog.csdn.net/qq_23089525/article/details/52789005)
```
xcode-select - -reset; xcode-select - -install
```
