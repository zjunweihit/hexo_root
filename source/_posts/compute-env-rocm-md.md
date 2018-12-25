---
title: ROCm Installation Guide
date: 2018-12-25 10:26:43
categories:
  - Compute
tags:
  - ROCm
---

How to install ROCm

<!--more-->

# Ubuntu 18.04

* Add apt repo
```
wget -qO - http://repo.radeon.com/rocm/apt/debian/rocm.gpg.key | sudo apt-key add -
echo 'deb [arch=amd64] http://repo.radeon.com/rocm/apt/debian/ xenial main' | sudo tee /etc/apt/sources.list.d/rocm.list
```
* Install ROCm packages
```
sudo apt update
sudo apt install rocm-dkms
```
* Grant the permission
```
sudo usermod -a -G video $LOGNAME
```
* Verify the driver
```
/opt/rocm/bin/rocminfo
/opt/rocm/opencl/bin/x86_64/clinfo
```
* More ROCm libs, if any
```
sudo apt install rocm-libs miopen-hip cxlactivitylogger
```
* Remove the ROCm
```
sudo apt autoremove rocm-dkms
```

# Reference

* [ROCm Installation Guide](https://rocm-documentation.readthedocs.io/en/latest/Installation_Guide/Installation-Guide.html#installing-from-amd-rocm-repositories)
