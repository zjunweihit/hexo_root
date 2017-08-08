---
title: How to build Mesa for amdgpu
date: 2017-08-08 15:41:38
categories:
tags:
  - Graphics
  - ubuntu
---

Build mesa for amdgpu in Ubuntu16.04.3

<!--more-->

# LLVM
```
cmake ../ -DLLVM_TARGETS_TO_BUILD="X86;AMDGPU" -DBUILD_SHARED_LIBS="ON"

make -j4 #If it's failed at the end, reduce the job number to allocate less memory
sudo make install
```
# libdrm
* Install dependent packages
```
apt install xutils-dev libtool libpciaccess-dev
```
* Build
```
./autogen.sh
make -j8
sudo make install
```
# Mesa
* Install dependent packages
```
apt install bison flex python-mako libpthread-stubs0-dev x11proto-gl-dev x11proto-dri2-dev x11proto-dri2-dev x11proto-present-dev libdrm-dev libdrm-amdgpu1 libdrm2 xcb libxcb-dri3-0 libxcb-dri3-dev libxcb-sync libxcb-present0 libxcb-present-dev libxcb-sync1 libxcb-sync-dev libxshmfence1 libxshmfence-dev libx11-dev libxext-dev libxdamage-dev libxfixes-dev libx11-xcb-dev libxcb-glx0-dev libxcb-dri2-0-dev libudev-dev libexpat1-dev libomxil-bellagio-dev libelf-dev automake zlib1g-dev
```
* Build
```
./configure --with-gallium-drivers=radeonsi --with-dri-drivers= --with-egl-platforms=drm,x11 --disable-xvmc --disable-vdpau --enable-texture-float
make -j8
sudo make install
```
