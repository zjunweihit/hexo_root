---
title: How to setup v2ray in Linux
date: 2020-05-06 16:16:36
categories: linux-env
tags:
  - linux-env-misc
---

Setup v2ray in Linux.

<!-- more -->

# Ubuntu 18.04

* Install the [v2rayL](https://github.com/jiangxufeng/v2rayL)
  - installing online seems not working, even if you downloads it successfully
  - download the zip from https://www.lanzous.com/iaynbud
  - install from the local folder
```
$ ./install.sh
```
* Set the scription with a correct link
* Turn on one of connection
* Find the config file `/etc/v2rayL/config.json`
```
{
    "port": 1081,
    "protocol": "http",
    "settings": {
        "userLevel": 8
    },
    "tag": "http"
}
```
* (Options) find the port by `netstat`
```
$ sudo netstat -tunlp | grep v2
tcp6       0      0 :::1080                 :::*                    LISTEN      6029/v2ray
tcp6       0      0 :::12345                :::*                    LISTEN      6029/v2ray
tcp6       0      0 :::1081                 :::*                    LISTEN      6029/v2ray
udp6       0      0 :::12345                :::*                                6029/v2ray
udp6       0      0 :::1080                 :::*                                6029/v2ray
```
* Setup proxy for cmake, wget and other command in termials
```
$ export http_proxy=127.0.0.1:1081
$ export https_proxy=127.0.0.1:1081
```
* Start your work then :)
