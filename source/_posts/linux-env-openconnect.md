---
title: How to setup vpn with openconnect
date: 2016-11-24 21:44:03
categories: linux-env
tags:
  - linux-env-misc
---

This document describes how to setup openconnect on your system.

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

# Ubuntu
* Get the stdid file
* Install stoken (https://sourceforge.net/p/stoken/wiki/Home/)
  - Ubuntu16.04(apt-get)
  - Ubuntu14.04(the stoken at 14.04 is too old, so install it from ppa)
```
sudo -s
apt-get install python-software-properties
add-apt-repository ppa:cernekee/ppa
apt-get update
apt-get install stoken libstoken-dev
```
* Import previous token ID
```
stoken import --file mytoken.sdtid
```
* Input init password or PIN and new PIN
* Cache PIN in ~/.stokenrc
```
stoken setpin
```
* Show code with either
```
stoken tokencode
stoken-gui
stoken-gui --small
```
* Install VPN client
```
apt-get install network-manager-openconnect-gnome
```
* Configure VPN in NetworkManager
```
Gateway: sha-vpn.abc.com
Token Mode: RSA SecurID - read from ~/.stokenrc
```
