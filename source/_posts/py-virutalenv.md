---
title: Python virtualenv
date: 2019-10-28 13:49:05
categories:
  - Programming
tags:
  - Python
---

Work in python virtual environment.

<!--more-->

# virtualenv

* install virtualenv
```
$ pip3 install virtualenv
```
* create virtualenv
```
$ virtualenv env1
$ ls env1 
bin  include  lib
```
* enter the env1
```
$ source env1/bin/activate
(env1) $
```
* do anything in the virtual env
  - install demo, which will be installed in `env1/lib/python3.6/site-packages/demo`
```
$ pip3 install demo
```
  - get the installed packages
```
$ pip freeze > requirements.txt
$ cat requirements.txt
pbr==5.4.3
six==1.12.0
stevedore==1.31.0
virtualenv==16.7.7
virtualenv-clone==0.5.3
virtualenvwrapper==4.8.4
zjunweidemo==0.0.1
```
* exit the env1
```
(env1) $ deactivate
$
```

# virtualenvwrapper
* install virtualenvwrapper
```
$ sudo apt install virtualenvwrapper
```
* enable virtualenvwrapper
```
$ source /usr/share/virtualenvwrapper/virtualenvwrapper.sh
```
* create virtual env
```
$ mkvirtualenv env2
```
* enter virtual env
```
$ workon env2
```
* exit virtualenvwrapper
```
deactivate
```
