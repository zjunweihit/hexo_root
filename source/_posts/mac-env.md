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

# shortcut
* screenshot
  * fullscreen
```
command + shift + 3
```
  * screenshot for self-defined area
```
command + shift + 3
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

