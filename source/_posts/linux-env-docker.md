---
title: How to use docker
date: 2017-08-08 13:22:32
categories: linux-env
tags:
  - ubuntu
---

Docker

<!--more-->

# Install Docker #
* Docker has two editions: Community Edition(CE) and Enterprise Edition(EE)
* CE also has two update channels, `stable` and `edge`:
  * `Stable`: gives you reliable updates every quarter
  * `Edge`: gives you new features every month

## Install Docker for Ubuntu
* Reference
  * [Install Docker for Ubuntu](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)
* Remove old version
```
sudo apt-get remove docker docker-engine docker.io
```
* Install packages
```
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
```
* Add official GPG key:
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
  * Verify that the key fingerprint is 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88.
```
sudo apt-key fingerprint 0EBFCD88

pub   4096R/0EBFCD88 2017-02-22
      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
      uid                  Docker Release (CE deb) <docker@docker.com>
      sub   4096R/F273FCD8 2017-02-22
```
* setup stable repository for amd64
```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```
* Install docker CE
```
sudo apt-get update
sudo apt-get install docker-ce
```
* Verify docker by `hello-world`
```
sudo docker run hello-world
```

# Manage Docker as a non-root user #
* Reference:
  * [Post-installation steps for Linux](https://docs.docker.com/engine/installation/linux/linux-postinstall/)
* Create `docker` group
```
sudo groupadd docker
```
* Add you to the `docker` group
```
sudo usermod -aG docker $USER
```
* Log out and log back in again to evaluate your group membership
* Verify docker without `sudo`
```
docker run hello-world
```

# Create docker image(interact on ubuntu16.04) #
* Get the target image
```
docker pull ubuntu:16.04
```
* Run a container (if you haven't pulled the image, it will be downloaded now)
```
$ docker run -it ubuntu:16.04 /bin/bash

-i --interactive	enter interactive container
-t --tty		allocate a pseudo-TTY
```
* Modify the container
```
root@cb62d48c0687:/# echo "echo hello-world" > hello.sh
root@cb62d48c0687:/# chmod +x hello.sh 
root@cb62d48c0687:/# ./hello.sh 
hello-world

(DON'T exit now)
```
* Check the running container
```
$ docker ps -a | head -n2
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                        PORTS               NAMES
cb62d48c0687        ubuntu:16.04        "/bin/bash"              5 minutes ago       Up 5 minutes                                      silly_payne
```
* Commit the container as image
```
$ docker commit -m "hello-world" -a "zjunwei" cb62d48c0687 hello-u1604:v1
sha256:be522fe570cccbcd391dfdc24c15e3e28e5951df29489ce02a9aefcbaaf8a260
```
* Find the image
```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-u1604         v1                  be522fe570cc        2 minutes ago       120MB
```
* Run as interactive
```
$ docker run -ti hello-u1604:v1
```
* Run the image directly
```
$ docker run hello-u1604:v1 echo abc
abc
```

# Create docker image(by dockerfile) #
* Create `dockerfile` as below
```
FROM ubuntu:16.04
MAINTAINER zjunwei

ADD echo.sh /
ENTRYPOINT ["/echo.sh"]
CMD ["This is test for docker hello"]
```
* Prepare echo.sh
```
#!/bin/bash
echo "hello from zjunwei"
```
* Build the image name:tag
```
$ docker build . -t zjunwei/hello:v1

Sending build context to Docker daemon  123.6MB
Step 1/5 : FROM ubuntu:16.04
 ---> 14f60031763d
Step 2/5 : MAINTAINER zjunwei
 ---> Using cache
 ---> 143d818b2089
Step 3/5 : ADD echo.sh /
 ---> Using cache
 ---> ae6348251dee
Step 4/5 : ENTRYPOINT /echo.sh
 ---> Using cache
 ---> f54009ef8c37
Step 5/5 : CMD This is test for docker hello
 ---> Using cache
 ---> 522f17f278d7
Successfully built 522f17f278d7
Successfully tagged zjunwei/hello:v1
```
* Find the image
```
$ docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
zjunwei/hello       v1                  522f17f278d7        6 minutes ago       120MB
```
* Run the image (Not support interactive way)
```
$ docker run zjunwei/hello:v1

hello from zjunwei
```

# TODO
* docer with xserver
  * [GUI app in container does not connect to X server in host](https://forums.docker.com/t/gui-app-in-container-does-not-connect-to-x-server-in-host/34702)
  * [docker with NV driver](http://gernotklingler.com/blog/docker-replaced-virtual-machines-chroots/)

# Reference #
* [What's docker](https://www.docker.com/what-docker)
* [docker home](https://www.docker.com)
* [docker docs](https://docs.docker.com)
* [10张图带你深入理解Docker容器和镜像](http://dockone.io/article/783)
* [Docker: Create Image from Container](http://blog.csdn.net/wxqee/article/details/52081866)
* [如何创建自己的docker image并上传到DockerHub上](http://www.cnblogs.com/shihuc/p/5152949.html)
* [Docker入门教程：15个Docker进阶命令](http://cloud.51cto.com/art/201412/462910.htm)
* [01-Docker新手入门初见篇](https://segmentfault.com/a/1190000007554125)

