---
title: ROCm Docker Setup
date: 2019-01-03 10:18:11
categories:
  - Compute
tags:
  - ROCm
---

How to setup ROCm docker(Ubuntu 18.04).

<!-- more -->

# Install ROCm
[ROCm Installation Guide](http://zjunweihit.github.io/2018/12/25/compute-rocm-installation/)

# Install docker

* [How to use docker](http://zjunweihit.github.io/2017/08/08/linux-env-docker/)
* Check storage driver, "overlay2" is recommended by ROCm kernel.
```
docker info
```
  - If not, start docker with `--storage-driver=<name>` to enable it as [select storage driver](https://docs.docker.com/storage/storagedriver/select-storage-driver/)

# Get ROCm image
* ROCm
```
docker pull rocm/rocm-terminal
```
* Caffe
```
sudo docker pull rocm/hipcaffe
```
* TensorFlow
```
sudo docker pull rocm/tensorflow
```

# Run a docker
```
alias drun='sudo docker run -it -v $HOME:/data --privileged --device=/dev/kfd'

drun rocm/rocm-terminal
drun rocm/hipcaffe
drun rocm/tensorflow
```

# Verify a docker
* rocminfo to find out a GPU
```
/opt/rocm/bin/rocminfo
```

* run a tensorflow hello example
```
import tensorflow as tf
hello = tf.constant('Hello, TensorFlow')
sess = tf.Session()
# show gpu support if anyG
sess.run(hello)
```

# Note
* rock-dkms in native OS should work with thunk in docker

# Reference
* [Preparing a machine to run with ROCm and docker](https://github.com/RadeonOpenCompute/ROCm-docker/blob/master/quick-start.md)
