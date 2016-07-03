---
title: minicom的多彩世界
date: 2016-07-02 21:45:06
categories: linux-env
tags:
  - linux-env-misc
---

Set up minicom to look into the debug machine

<!--more-->

# setup minicom #
1. Entry root user
```
    $ su -
```
1. Set minicom
```
    # minicom -s
    minicom: WARNING: configuration file not found, using defaults
    Device /dev/modem access failed: No such file or directory.
    (it means need configuration file for minicom)
    ………………………………[configuration]……………………………………
    ……………………   Filename and paths          …………………………
    ……………………   File transfer and protocols …………………………
    ……………………   Serial port setup           …………………………
    ……………………   Modem and dialing           …………………………
    ……………………   Screen and keyboard         …………………………
    ……………………   Save setup as df1           …………………………
    ……………………   Save setup as……           …………………………
    ……………………   Exit                        …………………………
    ……………………   Exit from  minicom          …………………………
    ……………………………………………………………………………………
```
1. Modify serial port setup as follow:
```
      A —  Serial Device  :    /dev/ttyS0
      B —  lockfile Location :  /var/lock
      C —  Calling Program :
      D —  Callout Program:
      E —   Bps/par/Bits   : 115200 8N1
      F —   Hardware Flow Control : NO
      G —   Software Flow Control : NO
       Change  with setting?
```
1. Set Modem and dialing as follow:
```
      delete A — initing string ……：
             B — Reset string ……：
             K — Hang-up string ……
```
1. Select Save as df1, it will be saved as default configuration.
1. Restart from minicom
1. Notice:
  * sometime even the setting is right, you cannot open the minicom with error as follow:
```
  Device /dev/ttys0 access failed : permission denied
```
  * Maybe it is reason that need to add your group(junwei-zhang) to the uucp group(CentOS root group name, other system 'dialout')
```
  $ sudo usermod -a -G uucp junwei-zhang
  -a --append append the user to the specify group with -G
  or
  $ sudo gpasswd -a junwei-zhang uucp
```
  * Maybe it also does not work, you can add junwei-zhang to tty device
```
$ gpasswd -a junwei-zhang tty
or
$ sudo usermod -a -G tty junwei-zhang
or
$ sudo vim /etc/group
append junwei-zhang to tty, as follow:
tty:x:5:junwei-zhang
```
  * /dev/ttyS0 permission denied
```
pax> ls -al /dev/ttyS0
crw-rw---- 1 root dialout 4, 64 Mar 23 21:00 /dev/ttyS0
                  ^^^^^^^
pax> echo xyzzy >/dev/ttyS0
bash: /dev/ttyS0: Permission denied

pax> id -Gn
pax cdrom floppy audio dip video plugdev netdev bluetooth scanner

pax> sudo adduser pax dialout
[sudo] password for pax: 
Adding user pax to group dialout
Done.

<< Restart done here to ensure new group picked up >>

pax> id -Gn
pax dialout cdrom floppy audio dip video plugdev netdev bluetooth scanner
```

# useful minicom wrapper
* useful minicom wrapper for store minicom log automatically.
  * refer to repo/minicom_wrapper.sh
  * set below in ~/.bashrc
```
alias minicom='/usr/local/bin/minicom_wrapper.sh'
```

# minicom sysrq
* 在minicom中,触发SysRq有时候可以帮助我们进行调试.
* 查看当前所有线程栈使用情况
```
ctrl+a f t
```
* 查看帮助
```
ctrl+a f h
```

