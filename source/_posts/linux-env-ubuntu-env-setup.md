---
title: Ubuntu系统环境那些事
date: 2016-06-29 22:30:00
categories: linux-env
tags:
  - ubuntu
---

居家旅行必备的设置，以下内容都是基于Ubuntu12.04系统的.
(Updated for Ubuntu14.04)

<!-- more -->

# Install Ubuntu

## Ubuntu 12.04 installer in USB mass storage device
* Download the Ubuntu 12.04 image(*.iso)
* Prepare a USB mass storage device and connect to the PC(assuming "sdb")
* Create installer in the USB device
```
$ su -
# fdisk -l
# dd bs=4M if=*.iso of=/dev/sdb
```
* Boot the PC and seclet boot from USB mass storage device
* I have tried to create one partition on the USB mass storage device, then create the installer in the new partition, other partitions are used for original purpose.
* USB memory cannot be done successfully, maybe USB hard disk can do this.
```
# fdisk /dev/sdb
  o: create empty partition table
  n: create new partition(sdb1 sdb2)

# dd bs=4M if=*.iso of=/dev/sdb
or 
# dd bs=4M if=*.iso of=/dev/sdb1  --> In this way, the partition cannot be found when booting PC, even if /dev/sdb1 is toggled as boot partition by fdisk(a: option)

# mkfs.vfat /dev/sdb2  --> Partition 2 can be used for windows.

I have created a Ubuntu13.04 installer in partition 1 of a USB hard disk successfully before. But unfortunately I missed the log...
```

## Ubuntu 12.04 硬件安装
(似乎依赖于windows系统，不推荐，虽然我目前就的就是这种方式)
(2016/07/02, 已经不用这种方法了)
* Usually there are 2 ways to setup by hardware.
* wubi安装(这可不是五笔啊)
  - get wubi.exe from ubuntu*.iso
  - wubi.exe --force-wubi
  - 因为12.04里，windows内安装的选项被禁用了。
* non-wubi
  * refer to
    - [setup with easybcd_1](http://wenku.baidu.com/view/bad8fe01e87101f69e3195a6.html)
    - [setup with esaybcd_2](http://wenku.baidu.com/view/ef1e8b210722192e4536f615.html)

# proxy settings

如果使用proxy来wget代码,那么必须要设置正确的proxy环境变量.

* 临时使用
``` bash
# export ftp_proxy=http://junwei-zhang:junwei123@improxy.mit.edu:10080
# export http_proxy=http://junwei-zhang:junwei123@improxy.mit.edu:10080
# export https_proxy=http://junwei-zhang:junwei123@improxy.mit.edu:10080
```
* 永久使用:将以上内容,写到bashrc中.
  * For all users
```
/etc/bash.bashrc
```
  * For the local user
```
~/.bashrc
```
* 查看是否写入到环境变量中
```
# env | grep proxy
ftp_proxy=http://junwei-zhang:junwei123@improxy.mit.edu:10080
http_proxy=http://junwei-zhang:junwei123@improxy.mit.edu:10080
https_proxy=http://junwei-zhang:junwei123@improxy.mit.edu:10080
```
* 如果apt-get是经过代理工作的,需要编缉/etc/apt/apt.conf
```
Acquire::http::Proxy "http://username:passwd@proxy_addr:port";
如:
Acquire::http::Proxy "http://junwei-zhang:junwei123@iproxy.mit.edu:10080";
Acquire::ftp::Proxy "http://junwei-zhang:junwei123@iproxy.mit.edu:10080";
```
* 如果是firefox需要proxy
  * 在网络设置里,输入proxy的IP及端口号

# 确保apt-get工作正常
```
apt-get update
apt-get upgrade
```
如果apt-get是经过代理工作的,需要编缉/etc/apt/apt.conf
```
Acquire::http::Proxy "http://username:passwd@proxy_addr:port";
如:
Acquire::http::Proxy "http://junwei-zhang:junwei123@iproxy.mit.edu:10080";
Acquire::ftp::Proxy "http://junwei-zhang:junwei123@iproxy.mit.edu:10080";
```
Ubuntu12.04, it may be got below error:
```
W: Failed to fetch http://mirrors.sohu.com/ubuntu/dists/precise/universe/i18n/Index  No Hash entry in Release file /var/lib/apt/lists/partial/mirrors.sohu.com_ubuntu_dists_precise_universe_i18n_Index

W: Failed to fetch bzip2:/var/lib/apt/lists/partial/mirrors.sohu.com_ubuntu_dists_precise-updates_main_binary-i386_Packages  Hash Sum mismatch

W: Failed to fetch bzip2:/var/lib/apt/lists/partial/extras.ubuntu.com_ubuntu_dists_precise_main_binary-i386_Packages  Hash Sum mismatch

E: Some index files failed to download. They have been ignored, or old ones used instead.
```
将/var/lib/apt/lists/partial/下的所有文件删除，但是这可以解决大部分问题，个别源不能解决，如"/var/lib/apt/lists/partial/extras.ubuntu.com_ubuntu_dists_precise_main_binary-i386_Packages"，无耐进入/var/lib/apt/lists/将对应的文件删除

但是在ubuntu14.04下，似乎不能这样解决。试过以下方法：
```
$ sudo apt-get clean
$ sudo rm /var/lib/apt/* -rf
$ sudo apt-get udpate --fix-missing
```
(其实也不只是ubuntu14.04这样，这个时候ubuntu12.04也有同样的问题）
```
software&update：donwload from， other， set best server，自动为我选择了ali云的镜像（http://mirrors.aliyun.com）

$ sudo apt-get clean
$ sudo rm /var/lib/apt/* -rf
$ sudo apt-get udpate --fix-missing
... <snip>
Get:60 http://mirrors.aliyun.com trusty-security/restricted Translation-en [14 B]
Get:61 http://mirrors.aliyun.com trusty-security/universe Translation-en [24.4 kB]
Ign http://mirrors.aliyun.com trusty/main Translation-en_US
Ign http://mirrors.aliyun.com trusty/multiverse Translation-en_US
Ign http://mirrors.aliyun.com trusty/restricted Translation-en_US
Ign http://mirrors.aliyun.com trusty/universe Translation-en_US 
Fetched 24.6 MB in 20s (1,182 kB/s)
W: Failed to fetch bzip2:/var/lib/apt/lists/partial/extras.ubuntu.com_ubuntu_dists_trusty_main_source_Sources  Hash Sum mismatch

W: Failed to fetch bzip2:/var/lib/apt/lists/partial/extras.ubuntu.com_ubuntu_dists_trusty_main_binary-i386_Packages  Hash Sum mismatch

E: Some index files failed to download. They have been ignored, or old ones used instead.

$ sudo vi /etc/apt/sources.list
  ## This software is not part of Ubuntu, but is offered by third-party
  ## developers who want to ship their latest software.
-  deb http://extras.ubuntu.com/ubuntu trusty main
+ #deb http://extras.ubuntu.com/ubuntu trusty main
-  deb-src http://extras.ubuntu.com/ubuntu trusty main
+ #deb-src http://extras.ubuntu.com/ubuntu trusty main

$ sudo apt-get udpate --fix-missing
Success!
```
使用apt-get下载*.deb包
```
apt-get download bluez
```
使用apt-get下载*.tar.gz源代码
```
apt-get source bluez
```

# firefox 同步 #
* [firefox机器间同步](http://support.mozilla.org/zh-CN/kb/%E5%A6%82%E4%BD%95%E5%9C%A8%E7%94%B5%E8%84%91%E4%B9%8B%E9%97%B4%E5%90%8C%E6%AD%A5Firefox%E7%9A%84%E8%AE%BE%E7%BD%AE)
* plugin with firefox(V11) in Ubuntu12.04 is incompatible, which because the version is too low.
  * To udpate 1:
  * Click "Help"->"Firefox Help"->"Firefox Free Download"
  * In the web page it will tell you it is not the latest version. You can update in this page.
  * It seemingly needs to restart the firefox or system.
  * To update 2:
  * sudo apt-get remove firefox
  * sudo apt-get install firefox

# Adobe flash support #
* Open Ubuntu software center(/usr/bin/software-center), search "flash"
* Open [adobe flash download](http://get.adobe.com/cn/flashplayer/)
* Select "APT, 适用于Uubuntu10.04+"
* Pop the window and choose "/usr/bin/software-center"
* Software-center could download and install flash plugin automatically

# 关闭ubuntu Tab键自动补全的声音 #
```
$ sudo vi /etc/inputrc
- # set bell-style none
+ set bell-style none
```

# 安装git #
```
$ sudo apt-get install git
```

# 安装quilt #
```
sudo apt-get install quilt
```

# 安装ssh #
```
$ sudo apt-get install ssh
```
* Ubuntu14.04
```
sudo apt-get install openssh-server

if you want to update the port number
sudo vi /etc/ssh/sshd_config
sudo service ssh restart
```

## ssh后 perl: warning: Setting locale failed ##
有时在chroot后到一个新的rootfs中，make uImage会在编译过程中出现以下warning。
```
  GEN     lib/oid_registry_data.c
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
        LANGUAGE = (unset),
        LC_ALL = (unset),
        LC_PAPER = "zh_CN.UTF-8",
        LC_ADDRESS = "zh_CN.UTF-8",
        LC_MONETARY = "zh_CN.UTF-8",
        LC_NUMERIC = "C",
        LC_TELEPHONE = "zh_CN.UTF-8",
        LC_IDENTIFICATION = "zh_CN.UTF-8",
        LC_COLLATE = "C",
        LC_MEASUREMENT = "zh_CN.UTF-8",
        LC_TIME = "zh_CN.UTF-8",
        LC_NAME = "zh_CN.UTF-8",
        LANG = "en_US.UTF-8"
perl: warning: Falling back to the standard locale ("C").
Compiling 49 OIDs
  CC      lib/oid_registry.o
```
这是因为local setting is not correct。可以在login user的bashrc中添加一些设置。如，你是以root用户登录到该系统，那就在/root/.bashrc中添加以下settings。
```
export LANGUAGE=en_US.UTF-8
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
locale-gen en_US.UTF-8
```

Referrence:
  * [locale-variables-have-no-effect-in-remote-shell-perl-warning-setting-locale-f](http://askubuntu.com/questions/144235/locale-variables-have-no-effect-in-remote-shell-perl-warning-setting-locale-f)

# 设置默认shell为bash #
(Ubuntu12.04默认shell为dash,即/bin/sh --> /bin/dash)
```
$ sudo dpkg-reconfigure dash
--> No
```

# rpm 包安装 #
```
$ apt-get install rpm
```

# 64-bit 系统对32-bit的兼容 #
```
$ sudo apt-get install ia32-libs
```

# 安装五笔 #
```
sudo apt-get install ibus-table-wubi
```
    * 如果没有ibus就先安装ibus
    * ibus开机启动
```
language support --> keyboard input method system选为ibus即可
```
* Ubuntu14.04, ibus不如以前那样好用了。因为，Ctrl+Space不能切换当前输入法与英文。所以转向Fcitx，国人写的输入法框架，更好的支持中文，支持一下！！
```
$ killall ibus-daemon
$ sudo apt-get install fcitx fcitx-table fcitx-table-wubi
```
* Remove the ibus, not needed now
```
$ sudo apt-get purge ibus
```
* But unfortunately, the ubuntu desktop is removed together(bad actions...), which causes the system setting is not available. Re-install the ubuntu desktop
```
$ sudo apt-get install ubuntu-desktop
```
* However, the `input method switch` does NOT work yet. The hotkey is used by others(`Text Entry` in `System Setting`).
  * Open `System Setting` and set the `Switch to next source` by other hotkey.
* Yeap, it's work now!

# 设置背景色 #
```
色度,饱和度,亮度
85,90,205
rgb
204 232 207
#CCE8CF
```

# vim 安装 #
* ubuntu12.04默认不能安装vim,这会导致vim相关的配置文件都无效.可以用以下方法安装vim.
```
The following packages have unmet dependencies:
vim : Depends: vim-common (= 2:7.3.429-2ubuntu2) but 2:7.3.429-2ubuntu2.1 is to be installed
E: Unable to correct problems, you have held broken packages.
```
* 卸载vim-common
```
$ sudo apt-get remove vim-common
```
* 再安装vim
```
$ sudo apt-get install vim
```

# 恢复光标关闭前的位置 #
* 增加以下内容在.vimrc
```
if has("autocmd")
    " When editing a file, always jump to the last cursor position
    autocmd BufReadPost *
    \ if line("'\"") > 0 && line ("'\"") <= line("$") |
    \   exe "normal! g'\"" |
    \ endif
endif
```
* uncomment以下内容在/etc/vim/vimrc
```
" Uncomment the following to have Vim jump to the last position when
" reopening a file
if has("autocmd")
  au BufReadPost * if line("'\"") > 1 && line("'\"") <= line("$") | exe "normal! g'\"" | endif
endif
```
* 如果以上两个方法都没有没好使，有可能是.viminfo没有设置正确的权限
```
$ ll ~/.viminfo
-rw------- 1 zjunwei zjunwei 4431 Nov  3 21:15 /home/zjunwei/.viminfo
 ^^          ^^^^^^^
```

# 目录結构 #
* vimwiki is got from internet
```
$ mkdir -p ~/.vim
$ cd ~/.vim
git clone https://github.com/zjunweihit/vimwiki
```
* others are got from vimwiki
```
$ cd vimwiki
$ cp -r automatically doc ftplugin plugin syntax ../
zjunwei@ubuntu:~$ ls .vim/
autoload  doc  ftplugin  plugin  syntax  vimwiki
```

# 创建root及其密码 #
```
zjunwei@ubuntu:~$ sudo passwd root
[sudo] password for zjunwei:
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
zjunwei@ubuntu:~$ su -
Password:
root@ubuntu:~#
```

# 设置默认编辑器 #
* 终端输入:
```
$ echo export EDITOR=/usr/bin/vim >> ~/.bashrc
```
* 使用系统管理工具update-alternatives设置
```
$ sudo update-alternatives --config editor
There are 3 choices for the alternative editor (providing /usr/bin/editor).

  Selection    Path                Priority   Status
  ------------------------------------------------------------
  * 0            /bin/nano            40        auto mode
    1            /bin/ed             -100       manual mode
    2            /bin/nano            40        manual mode
    3            /usr/bin/vim.basic   30        manual mode

  Press enter to keep the current choice[*], or type selection number: 3
  update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/editor (editor) in manual mode.
```
* 有可能会遇到两个vim,分别如下:
```
  Selection    Path                Priority   Status
  ------------------------------------------------------------
  * 0            /bin/nano            40        auto mode
    1            /bin/ed             -100       manual mode
    2            /bin/nano            40        manual mode
    3            /usr/bin/vim.basic   30        manual mode
    4            /usr/bin/vim.tiny    10        manual mode

vim.basic是一个完整版的vim，但没有图像界面
vim.tiny是一个vim的缩减版
```

# fix: override cpp setting #

Ubuntu 18.04, vim default setting has overrided ~/.vim/ftplugin/cpp.vim.
```
$ ls /usr/share/vim/vim80/ftplugin/cpp.vim
```

comment the last line:
```
"runtime! ftplugin/c.vim ftplugin/c_*.vim ftplugin/c/*.vim
```

# Add user to a group #
How to add a user to a group
* add user to "sudo" group
```
$ sudo adduser `whoami` sudo
```
* You can test which you belong to with the id -Gn command.
```
$ id -Gn
junwei-zhang adm disk dialout fuse admin
```
* After adding yourself to a certain group, you'll notice that the group is not immediately visible in the output of that command. If you do not want to restart the graphical session ("re-login"), perform a direct login:(or reboot the system)
```
$ su - $USER
```
* Check it again
```
$ id -Gn
junwei-zhang adm disk dialout sudo fuse admin
```

# Keep the env for sudo #
如何保留"sudo command"时环境变量?
* Modify the /etc/sudoer
```
$ sudo visudo

+ sudo env_keep += "http_proxy https_proxy ftp_proxy"
                    ^^^^^^^^^^
                    Add the env variant you want
```

# sudo echo #
* This won't work:
```
    sudo echo "Text I want to write" > /path/to/file
```
* If you want to write some text to a file that needs a root privileges, you do it this way:
```
echo "Text I want to write" | sudo tee /path/to/file > /dev/null
```
  * or (updated after reading discussion below):
```
sudo sh -c 'echo "Text I want to write" > /path/to/file'
```
* If you just want to append some text, you do it this way:
```
echo "Text I want to write" | sudo tee -a /path/to/file > /dev/null
```
  * or (updated after reading discussion below):
```
    sudo sh -c 'echo "Text I want to write" >> /path/to/file'
```

# sudo不需要输入密码 #
```
(1)
$ su -

(2)
#chmod 660 /etc/sudoer

(3)
modify it as follow:(using sudo without password)

%sudo ALL=NOPASSWD: ALL

(4)
#chmod 440 /etc/sudoer

(5) add user to sudo group
$sudo adduser `whoami` sudo
```
```
$ su -
$ sudoedit /etc/sudoer
+ %sudo ALL=NOPASSWD: ALL
$ exit
```
```
$ su -
$ sudoedit /etc/sudoer

# User privilege specification
root    ALL=(ALL:ALL) ALL
+ username ALL=NOPASSWD: ALL

# Allow members of group sudo to execute any command
- %sudo>-ALL=(ALL:ALL) ALL
+ #%sudo>-ALL=(ALL:ALL) ALL
$ exit
```

# Eclipse setup #
* Download JDK
```
$ sudo apt-get install openjdk-7-jdk

$ java <Tab>
java     javac    javadoc  javah    javap 
$ java -version
java version "1.7.0_25"
OpenJDK Runtime Environment (IcedTea 2.3.10) (7u25-2.3.10-1ubuntu0.12.04.2)
OpenJDK Server VM (build 23.7-b01, mixed mode)
```
* Download Eclipse
  * [eclipse kepler sr1](http://www.eclipse.org/downloads/packages/eclipse-ide-cc-developers/keplersr1)
```
$ sudo tar xvf eclipse-cpp-kepler-SR1-linux-gtk.tar.gz -C /opt/
```
  * Run eclipse
```
$ /opt/eclipse/eclipse &
```

# Wine for windows program #
- Open "Ubuntu Software Center"
- search "wine"
- install "wine Windows Program loader"
- Download the Windows software *.exe
- Click right button on it
- select "open with wine Windows Program loader"

# Boot Ubuntu from terminal #
(Ubuntu14.04: referring to http://www.htpcbeginner.com/force-ubuntu-boot-into-terminal/)
* Update the grub default settings
```
$ sudo vi /etc/default/grub

- GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
+ GRUB_CMDLINE_LINUX_DEFAULT="text"
```

# Boot Ubuntu from specific kernel #
* grep the menu entry
```
cat /boot/grub/grub.cfg | grep menuentry
```
* set default option in grub, entering the secondary option by ">"
```
grub-set-default 'Advanced options for Ubuntu>Ubuntu, with Linux 4.7.0-custom'
```
* update grub
```
vi /etc/default/grub

+ GRUB_DEFAULT=save

update-grub
```

# Set memmap in GRUB #
* `$` needs escaping symbol `\` both in default grub and grub.cfg
* default grub
```
$ sudo vi /etc/default/grub

- GRUB_CMDLINE_LINUX=""
+ GRUB_CMDLINE_LINUX="memmap=1G\\\$3G"

$ update-grub
the /boot/grub/grub.cfg will be updated accordingly
```
* grub.cfg, following will be appended to the command line
```
memmap=1G\$3G
```
* Verify after reboot
```
# dmesg | grep user
[    0.000000] user: [mem 0x00000000c0000000-0x00000000ffffffff] reserved
```
* referring to
  * [[PATCH] x86: e820: fix memmap kernel boot parameter](http://www.gossamer-threads.com/lists/linux/kernel/1774749)
  * [kernel-parameters.txt](https://www.kernel.org/doc/Documentation/kernel-parameters.txt)

# Uncomment to disable graphical terminal (grub-pc only)
```
- #GRUB_TERMINAL=console
+ GRUB_TERMINAL=console

$ sudo update-grub
```
* Reboot the system, you will login the terminal by default
* If you want to revert the changes, you can recover the /etc/default/grub and update the grub.
* If you only want to use the terminal, you can press "Ctrl+Alt+F1"~"Ctrl+Alt+F6". Then "Ctrl+Alt+F7" will take you back to the GUI desktop.

# Boot Ubuntu without the specific module installed automatically #
```
# echo "blacklist DRV_NAME" > /etc/modprobe.d/blacklist.conf

e.g.
# echo "blacklist amdgpu" > /etc/modprobe.d/blacklist.conf
```

# Networking setting
* Modify host name
  * forever
```
    $ sudo vi /etc/hostname
    name_new

    $ reboot
    or
    $ sudo hostname name_new
    $ hostname
    name_new

    $ sudo vi /etc/hosts
    127.0.1.1   name_new
```
* static IP
  * vi /etc/network/interfaces
```
# eth0 for static
auto eth0
iface eth0 inet static
address 192.168.0.131
netmask 255.255.255.0
gateway 192.168.0.1

# eth1 for dhcp
auto eth1
iface eth1 inet dhcp
```
  * restart network
```
$ sudo /etc/init.d/networking restart
or
$ sudo service networking restart
```
* set DNS
  * vi /etc/resolv.conf
    * 如果只修改这里,重启后会丢失DNS的信息
```
domain sdec.
search sdec.
nameserver 10.11.12.13
nameserver 192.168.0.2
```
  * vi /etc/resolvconf/resolv.conf.d/head
    * or /etc/resolvconf/resolv.conf.d/base (未测试)
    * or /etc/resolvconf/resolv.conf.d/tail (未测试)
```
domain sdec.
search sdec.
nameserver 10.11.12.13
nameserver 192.168.0.2
```
  * refresh resolvconf
```
$ sudo service resolvconf restart
```
* tftp
  * 安装tftp的客户端,服务器和xinetd
```
$ sudo apt-get install tftp-hpa tftpd-hpa xinetd
```
  * vi /etc/xinetd.conf(设置xinetd, 如果没有就创建)
```
# Simple configuration file for xinetd
#
# Some defaults, and include /etc/xinetd.d/

defaults
{

# Please note that you need a log_type line to be able to use log_on_success
# and log_on_failure. The default is the following :
# log_type = SYSLOG daemon info

}

includedir /etc/xinetd.d
```
  * vi /etc/xinetd.d/tftp(设置xinentd中的tftp控制,如果没有就创建)
```
service tftp
{
        socket_type             = dgram
        protocol                = udp
        wait                    = yes
        user                    = root
        server                  = /usr/sbin/in.tftpd
        server_args             = -s /tftpboot -c
        disable                 = no
        per_source              = 11
        cps                     = 100 2
        flags                   = IPv4
}
```
    * server_args配置tftp根目录路径
  * vi /etc/default/tftpd-hpa
```
* /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/tftpboot"
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="-l -c -s"
```
  * 如果需要,可以修改tftp根目录的权限
```
$ sudo chmod –R 777 /tftpboot
```
  * 重启服务
```
$ sudo /etc/init.d/xinetd reload
$ sudo /etc/init.d/xinetd restart
$ sudo service tftpd-hpa restart
```
  * 主机测试
```
# write some test info.
$ vi /tftpboot/a.txt
$ cd ~
$ tftp localhost
tftp> get a.txt
tftp> q
$ ls -l a*.txt
```
  * 常见问题
```
现象一：

tftp> get test.log

Transfer timed out.

原因：

tftpd服务没有启动

现象二：

tftp> get test.log

Error code 2: Only absolute filenames allowed

原因：

在/etc/xinetd.d/tftpd中设置的server_args为/etc/default/tftpd-hpa

cat /etc/default/tftpd-hpa

#Defaults for tftpd-hpa

RUN_DAEMON="no"

OPTIONS="-s /home/tftpd -c -p -U 077 -u tftpd"

设置的时候只要将server_args＝改为你自己设定的服务器文件夹就行了

现象三：

tftp> put ex070416.log

Error code 1: File not found

原因：

指定的文件不存在；或tftpd启动参数中没有指定-c选项，允许上传文件

现象其它:
再不好使就重新装一次^_^
```
* NFS
  * nfs server(通常板子的调试,只需要安装server即可)
    * setup nfs server
```
$ sudo apt-get install nfs-kernel-server nfs-common
```
    * 重启nfs server
```
$ sudo service nfs-kernel-server restart
```
    * 设置共享目录
```
$ sudo vi /etc/exports

/tftpboot       *(rw,sync,no_root_squash,no_all_squash)
$ sudo service nfs-kernel-server restart

for CentOS
$ sudo /usr/sbin/exportfs -a
```
  * nfs client
```
$ sudo apt-get install nfs-common
```
    * 查看nfs server上共享的目录
```
showmount -e 192.168.xx.xx
```
    * 创建共享挂载点，并执行挂载
```
$ mkdir /path-to-mount
$ mount -t nfs 192.168.xx.xx:/path-to-share /path-to-mount
```
    * 修改/etc/fstab文件
```
192.168.xx.xx:/data3/nfsroot/data3/nfsroot nfs defaults 0 0
```
    * 在共享目录下创建一个文件，看看另外一端能不能看到。

# Local networking setting RNDIS(Ubuntu) #
* Remote Network Device Interface Specification.
* 它是由微软制定的规范,Linux也兼容这个规范. 通过USB的Gadget,将device虚拟为一个网卡,以实现本地网络的连接.因此,IP必须为192.168.x.x的范围.
* Host PC
  * Linux(Ubuntu)需要使能下以Kconfig.
```
|-- Device Drivers  --->
    |-- [*] Network device support (CONFIG_NETDEVICES)  --->
        |-- USB Network Adapters  --->
            |-- <*> Multi-purpose USB Networking Framework (CONFIG_USB_USBNET)
```
    * 一般情况下,都是以 <M> 的形式发布的.所以,只要查看一下rndis_host.ko是否安装.
```
$ lsmod | grep eth
cdc_ether              13536  1 rndis_host
usbnet                 26212  3 rndis_host,cdc_subset,cdc_ether
```
    * 在我实验的环境里,并没有安装rndis_host.ko,但是也可以实现RNDIS功能.
  * Windows
    * 需要安装微软的Generic RNDIS驱动.
* Device: usually it is embedded device.
  * 需要使能以下Kconfig, 产生g_ether.ko的功能,并支持RNDIS. g_ether是USB Gadget ethernet的功能.
```
|-- Device Drivers  --->
    |-- [*] USB support (CONFIG_USB_SUPPORT)  --->
        |-- <*> USB Gadget Support (CONFIG_USB_GADGET)  --->
            `-- <*> USB Gadget Drivers  --->
                    |-- [*] RNDIS support (CONFIG_USB_ETH_RNDIS)
```
  * 如果同以上将g_ether.ko编入到内核的情况,可以在cmdline是设置ip,在系统启动过程中,会找网络.
```
setenv bootargs  root=/dev/mmcblk0p7 rootwait console=ttymxc1,115200 rw nosmp ip=192.168.1.2
```
  * 如果是以<M>的方法,那么在系统起来后,安装g_ether.ko,然后,手动设置IP
```
ifconfig usb0 192.168.1.2
```
* 待device系统起来,并且带有RNDIS的g_ether功能工作时,host(Ubuntu)会自动连接该设备.这时,我们要disconncet它.因为,它总是在不停地连接,使用过程中可能会自动断开,影响正常使用.断开后,Ubuntu将不会再自动连接,这时,我们要手动设置host端的ip.
```
ifconfig usb0 192.168.1.3 up
```
* 至此,我们可以稳定地使用host与device之间的网络.
* 注意:
  - 如果,host同时还在另一个local ip段内,如:192.168.0.123, 那么设置ip时,不要与它处于同一个ip段,否则,不能成功连接.

# Networking setting (CentOS) #
以下内容适用于CentOS等Redhat的Linux系统.
*  CentOS basic setings
  * modify host name
    * temporary & now
```
    $ hostname
    name_org
    $ sudo hostname name_new
    $ hostname
    name_new
```
    * forever & reboot
```
    $ sudo vi /etc/sysconfig/network
    HOSTNAME=name_new
    $ sudo vi /etc/hosts
    127.0.0.1   localhost.localdomain   name_new
```
  * DNS: /etc/resolv.conf
```
nameserver 8.8.8.8 #google域名服务器
nameserver 8.8.4.4 #google域名服务器
```
  * check DNS server status
```
  $ nslookup www.google.com
```
  * gateway: /etc/sysconfig/network
```
NETWORKING=yes(表示系统是否使用网络，一般设置为yes。如果设为no，则不能使用网络，而且很多系统服务程序将无法启动)
HOSTNAME=centos(设置本机的主机名，这里设置的主机名要和/etc/hosts中设置的主机名对应)
GATEWAY=192.168.1.1(设置本机连接的网关的IP地址。例如，网关为10.0.0.2)
```
  * restart network
```
$ sudo service network restart
```
  * modify IP: /etc/sysconfig/network-scripts/ifcfg-eth0
```
DEVICE=eth0 #描述网卡对应的设备别名，例如ifcfg-eth0的文件中它为eth0
BOOTPROTO=static #设置网卡获得ip地址的方式，可能的选项为static，dhcp或bootp，分别对应静态指定的 ip地址，通过dhcp协议获得的ip地址，通过bootp协议获得的ip地址
BROADCAST=192.168.0.255 #对应的子网广播地址
HWADDR=00:07:E9:05:E8:B4 #对应的网卡物理地址
IPADDR=192.168.1.17 #如果设置网卡获得 ip地址的方式为静态指定，此字段就指定了网卡对应的ip地址
IPV6INIT=no
IPV6_AUTOCONF=no
NETMASK=255.255.255.0 #网卡对应的网络掩码
NETWORK=192.168.1.0 #网卡对应的网络地址
ONBOOT=yes #系统启动时是否设置此网络接口，设置为yes时，系统启动时激活此设备
```
  * firewall setting:
```
sudo service iptables save
sudo service iptables stop
sudo chkconfig iptables off
```
* 和网络相关的设置主要如下:
```
/etc/host.conf         配置域名服务客户端的控制文件
/etc/hosts             完成主机名映射为IP地址的功能
/etc/resolv.conf       域名服务客户端的配置文件,用于指定域名服务器的位置
/etc/sysconfig/network 包含了主机最基本的网络信息,用于系统启动.
/etc/sysconfig/network-script/    系统启动时初始化网络的一些信息
/etc/xinetd.conf       定义了由超级进程xinetd启动的网络服务
/etc/networks          完成域名与网络地址的映射
/etc/protocols         设定了主机使用的协议以及各个协议的协议号
/etc/services          设定主机的不同端口的网络服务
```
* /etc/host.conf文件的默认信息如下：
```
multi on           #允许主机拥有多个IP地址
order hosts,bind   #主机名解析顺序，即本地解析,DNS域名解析的顺序
```
  * 这个文件一般不需要我们修改，默认的解析顺序是本地解析,DNS服务器解析，也就是说在本系统里对于一个主机名首先进行本地解析，如果本地解析没有，然后进行DNS服务器解析。
* /etc/hosts文件默认的内容大概如下：
```
127.0.0.1   zjunwei localhost.localdomain localhost
::1         localhost6.localdomain6 localhost6
```
  * 第一行是ipv4, 第二行是ipv6,不用的话,注释掉
  * 如果ping localhost的时候,显示zjunwei.因为,后面的是它的别名.
  * 这个文件如果被人恶意修改了,是很危险的.比如,淘宝的网站域名解析到了其它的钓鱼网站上.
* /etc/resolv.conf, 指定域名解析的DNS服务器IP等信息
```
nameserver    指定DNS服务器的IP地址
domain        定义本地域名信息
search        定义域名的搜索列表
sortlist      对gethostbyname返回的地址进行排序

一般使用nameserver就够用了
```
  * 一般来说，推荐设置2个DNS服务器，比如我们用google的免费DNS服务器，那么该文件的设置内容如下：
```
nameserver 8.8.8.8
nameserver 8.8.4.4
```
* /etc/sysconfig/network
```
NETWORKING=yes
NETWORKING_IPV6=no
HOSTNAME=butbueatiful
GATEWAY=192.168.0.1

----
NETWORK          设置网络是否有效，yes有效，no无效
NETWORKING_IPV6  设置ipv6网络是否有效，yes有效，no无效
HOSTNAME         设置服务器的主机名，最好和/etc/hosts里设置一样，否则在使用一些程序的时候会有问题。
GATEWAY          指定默认网关IP
```
* ifcfg-ethX, 设置对应网口的IP等信息, 比如第一个网口, 那么就是/etc/sysconfig/network-scripts/ifcfg-eth0，配置例子：
```
DEVICE="eth0"
BOOTPROTO="static"
BROADCAST="192.168.0.255"
HWADDR="00:16:36:1B:BB:74"
IPADDR="192.168.1.110"
NETMASK="255.255.255.0"
ONBOOT="yes"

----
DEVICE="eth0"
BOOTPROTO="dhcp"
HWADDR="00:0C:29:8E:13:7E"
NM_CONTROLLED="no"
ONBOOT="yes"

----
DEVICE        设备名，不要自己乱改，和文件ifcfg-ethX里的ethX要一致
BROADCAST     广播地址
HWADDR        物理地址，这个你不要乱改
IPADDR        IP地址
NETMASK       子网掩码
ONBOOT        启动或者重启网络时，是否启动该设备，yes是启动，no是不启动
BOOTPROTO     开机协议，最常见的三个参数如下：
              static(静态IP)
              none(不指定,设置固定ip的情况，这个也行，但是如果要设定多网口绑定bond的时候，必须设成none）
              dhcp(动态获得IP相关信息）

DEVICE="eth0"              设备名
NM_CONTROLLED="yes"        设备是否被NetworkManager管理
ONBOOT="no"                开机是否启动
HWADDR="00:0C:29:59:E2:D3" 硬件地址（MAC地址）
TYPE=Ethernet              类型
BOOTPROTO=none             启动协议{none|dhcp}
IPADDR=192.168.0.1         IP地址
PREFIX=24                  子网掩码
GATEWAY=192.168.0.254      默认网关
DNS1=202.106.0.20          主DNS
DOMAIN=202.106.46.151      辅助DNS
UUID=5fb06bd0-0bb0-7ffb-45f1-d6edd65f3e03    设备UUID编号
```
* route-ethX,比如第一个网口eth0的路由信息,那么就是/etc/sysconfig/network-scripts/route-eth0：
```
比如我们现在有这样一个需求，通过eth0去网络172.17.27.0/24不走默认路由，需要走192.168.0.254，那么我们第一反应，肯定是用route命令追加路由信息：
[root@butbueatiful ~]# route add -net 172.17.27.0 netmask 255.255.255.0 gw 192.168.0.254 dev eth0

可是，你没意识到的是，这样只是动态追加的而已，重启网络后，路由信息就消失了，所以需要设置静态路由，这时候就要设置/etc/sysconfig/network-scripts/route-eth0文件了，如果没有该文件，你就新建一个：
[root@butbueatiful ~]# vi /etc/sysconfig/network-scripts/route-eth0
#追加
172.17.27.0/24via 192.168.0.254

这下即使重启网络，重启系统，该路由也会自动加载，当然了，如果你没有这样的需要，那么这个文件就没必要创建和配置了。
```
* 常用修改文件及命令
```
/etc/sysconfig/network-scripts/ifcfg-设备名（通常为ifcfg-eth0)
/etc/sysconfig/network
/etc/resolv.conf文件：设置DNS服务器

以上种种这些方式可以同时在5.0与6.0系统中实现，但6.0系统后官方文档中描述说：ifconfig与route是非常陈旧的命令，取而代之的是ip命令。
```
  * 老命令
```
ifconfig    接口  选项|地址

# ifconfig  eth0  up          # 开启eth0网卡
# ifconfig  eth0  down        # 关闭eth0网卡
# ifconfig  eth0  -arp        # 关闭eth0网卡arp协议
# ifconfig  eth0  promisc     # 开启eth0网卡的混合模式
# ifconfig  eth0  mtu 1400    # 设置eth0网卡的最大传输单元为1400
# ifconfig  eth0  192.168.0.2/24    # 设置eth0网卡IP地址
# ifconfig  eth0  192.168.0.2  netmask 255.255.255.0    # 功能同上

网关设置：

route  add [-net|-host] target [netmask] gw
route  del [-net|-host] target [netmask] gw

# route add  -net 192.168.3.0/24  gw  192.168.0.254    # 设置到192.168.3.0网段的网关为192.168.0.254
# route add  -net 192.168.3.0 netmask 255.255.255.0  gw  192.168.0.254    # 功能同上
# route add  -host 192.168.4.4  gw  192.168.0.254    # 设置到192.168.4.4主机的网关为192.168.0.254
#
# route del  -net 192.168.3.0/24                        # 删除192.168.3.0网段的网关信息
# route del  -host 192.168.4.4                        # 删除192.168.4.4主机的网关信息
# route add default gw  192.168.0.254                # 设置默认网关为192.168.0.254
# route del default gw  192.168.0.254                # 删除默认网关为192.168.0.254
```
  * 新命令
```
ip  [选项]  操作对象{link|addr|route...}

# ip link show                  # 显示网络接口信息
# ip link set eth0 upi          # 开启网卡
# ip link set eth0 down         # 关闭网卡
# ip link set eth0 promisc on   # 开启网卡的混合模式
# ip link set eth0 promisc offi # 关闭网卡的混个模式
# ip link set eth0 txqueuelen 1200    # 设置网卡队列长度
# ip link set eth0 mtu 1400     # 设置网卡最大传输单元
# ip addr show                  # 显示网卡IP信息
# ip addr add 192.168.0.1/24 dev eth0 # 设置eth0网卡IP地址192.168.0.1
# ip addr del 192.168.0.1/24 dev eth0 # 删除eth0网卡IP地址

# ip route list                 # 查看路由信息
# ip route add 192.168.4.0/24  via  192.168.0.254 dev eth0 # 设置192.168.4.0网段的网关为192.168.0.254,数据走eth0接口
# ip route add default via  192.168.0.254  dev eth0    # 设置默认网关为192.168.0.254
# ip route del 192.168.4.0/24    # 删除192.168.4.0网段的网关
# ip route del default    # 删除默认路由
```
* 安全设置
  * /etc/resolv.conf和/etc/hosts被人篡改了的话, 会很危险, 那我们在设置好着2个文件后, 做一下处理, 让这2个文件默认不能直接修改, 即使root也不行, 执行如下命令：
```
# chattr +i /etc/{resolv.conf,hosts}
```
  * 如果想修改时
```
# chattr -i /etc/{resolv.conf,hosts}
```
* 网络排除思路
```
检查本机网络协议是否正确:  # ping -c 3 127.0.0.1
检查本机网卡链路是否正确:  # ping -c 3 192.168.0.1(本机IP地址)
检查网关是否正确:          # ping -c 3 192.168.0.254(网关IP地址)
检查外部连通性:            # ping -c 3 www.google.com.hk
```

# cifs mounting #
* mount cifs as usual nfs for kernel < 3.8, but the newer Ubuntu 16.04 adopts different way
```
The default in mainline kernel versions prior to v3.8 was sec=ntlm. In v3.8, the default was changed to sec=ntlmssp.

$ sudo mount.cifs ubuntu-shared/ /mnt -o user=admin,sec=ntlm
```
> Reference
  - [mount error(13): Permission denied](https://askubuntu.com/questions/337128/mount-error13-permission-denied)

# ldconfig #
* check lib loading order
```
ldconfig -p
```
* after install lib in system, load the lib in cache file
```
sudo ldconfig
```
* config setting, loaded by order(number, character)
```
/etc/ld.so.conf.d
```
# locate #
Update locate database, which will be updated daily by cron(man updatedb)
```
sudo updatedb
```
# update-alternative #
* setup link for gcc/g++,
  - `--install <link> <name> <path> <priority>` priority is higher if the value is bigger.
```
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 5
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 10

sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-5 5
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-7 10
```

* config different gcc/g++
```
sudo update-alternatives --config gcc
There are 2 choices for the alternative gcc (providing /usr/bin/gcc).

  Selection    Path            Priority   Status
------------------------------------------------------------
* 0            /usr/bin/gcc-7   10        auto mode
  1            /usr/bin/gcc-5   5         manual mode
  2            /usr/bin/gcc-7   10        manual mode

Press <enter> to keep the current choice[*], or type selection number: 
```
