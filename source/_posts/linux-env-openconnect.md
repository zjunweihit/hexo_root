---
title: How to setup vpn with openconnect
date: 2016-11-24 21:44:03
categories: linux-env
tags:
  - linux-env-misc
---

This document describes how to setup hexo on your system.

<!-- more -->

# Mac
* Install requisite
```
brew cask install tuntap
brew install gcc lz4 gnutls pkg-config autoconf automake(including aclocal-1.15) stoken(including libstoken)
```
  - No need to install vpnc, which is used for cisco connection
* Install vpnc-script
  - Copy the contents from the page [vpnc-script](http://www.infradead.org/openconnect/vpnc-script.html) to the default file as below
```
vi /etc/vpnc/vpnc-script
sudo chmod +x /etc/vpnc/vpnc-script
```
* Import your stdid file
```
stoken import --file /path/to/yourname.stdid
```
* Install openconnect
```
wget ftp://ftp.infradead.org/pub/openconnect/openconnect-7.07.tar.gz
tar xvf openconnect-7.07.tar.gz
cd openconnect-7.07
./configure --with-stoken --disable-nls
make
make install
```
* Connect to the vpn server
```
sudo openconnect --token-mode=rsa -u <username> <vpn-server>
```
* Reference
  - [infradead.org](http://www.infradead.org/openconnect/index.html)
  - [macosx install openconnect with libstoken support](https://www.techietek.com/2016/01/19/macosx-install-openconnect-with-libstoken-support/)

