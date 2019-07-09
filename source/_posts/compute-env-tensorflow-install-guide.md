---
title: TensorFlow Installation Guide
date: 2018-12-25 10:33:56
categories:
  - Compute
tags:
  - TensorFlow
  - ROCm
---

How to install TensorFlow. (Verified in Ubuntu 18.04)

<!--more-->
# Software in Ubuntu 18.04
  - tensorflow 1.12.0
  - ROCm 2.0.89
  - CUDA Toolkit 9.0(required by tensorflow-gpu in 1.12)
  - Python3 3.6.5 is installed by default
```
sudo apt install python3-pip

The following NEW packages will be installed:
  dh-python libexpat1-dev libpython3-dev libpython3.6-dev python-pip-whl python3-dev python3-distutils python3-lib2to3 python3-pip python3-setuptools python3-wheel python3.6-dev
```

# TensorFlow for CPU
```
pip3 install tensorflow

Installing collected packages: astor, six, numpy, keras-preprocessing, h5py, keras-applications, setuptools, protobuf, wheel, markdown, werkzeug, grpcio, tensorboard, gast, termcolor, absl-py, tensorflow
Successfully installed absl-py-0.6.1 astor-0.7.1 gast-0.2.0 grpcio-1.17.1 h5py-2.9.0 keras-applications-1.0.6 keras-preprocessing-1.0.5 markdown-3.0.1 numpy-1.15.4 protobuf-3.6.1 setuptools-40.6.3 six-1.12.0 tensorboard-1.12.1 tensorflow-1.12.0 termcolor-1.1.0 werkzeug-0.14.1 wheel-0.32.3
```

# TensorFlow for ROCm
## Install ROCm
  - [ROCm Installation Guide](https://zjunweihit.github.io/2018/12/25/compute-rocm-installation/)
  - **includes ROCm libs**

## Install TensorFlow ROCm
* Install miopen
```
sudo apt install rocm-libs miopen-hip cxlactivitylogger
```
* Install tensorflow-rocm whl package from PyPI
```
pip3 install --user tensorflow-rocm

Installing collected packages: wheel, six, setuptools, protobuf, markdown, grpcio, werkzeug, numpy, tensorboard, gast, keras-preprocessing, absl-py, astor, termcolor, h5py, keras-applications, tensorflow-rocm
Successfully installed absl-py-0.6.1 astor-0.7.1 gast-0.2.0 grpcio-1.17.1 h5py-2.9.0 keras-applications-1.0.6 keras-preprocessing-1.0.5 markdown-3.0.1 numpy-1.15.4 protobuf-3.6.1 setuptools-40.6.3 six-1.12.0 tensorboard-1.12.1 tensorflow-rocm-1.12.0 termcolor-1.1.0 werkzeug-0.14.1 wheel-0.32.3
```

# TensorFlow for CUDA

## Install Nvidia CUDA driver
```
sudo apt install nvidia-cuda-dev


The following additional packages will be installed:
  libaccinj64-9.1 libcublas9.1 libcudart9.1 libcufft9.1 libcufftw9.1 libcuinj64-9.1 libcurand9.1 libcusolver9.1 libcusparse9.1 libnppc9.1 libnppial9.1 libnppicc9.1 libnppicom9.1 libnppidei9.1
  libnppif9.1 libnppig9.1 libnppim9.1 libnppist9.1 libnppisu9.1 libnppitc9.1 libnpps9.1 libnvblas9.1 libnvgraph9.1 libnvidia-compute-390 libnvrtc9.1 libnvtoolsext1 libnvvm3 libthrust-dev libvdpau-dev
Suggested packages:
  nvidia-cuda-toolkit libvdpau-doc
Recommended packages:
  libnvcuvid1
The following NEW packages will be installed:
  libaccinj64-9.1 libcublas9.1 libcudart9.1 libcufft9.1 libcufftw9.1 libcuinj64-9.1 libcurand9.1 libcusolver9.1 libcusparse9.1 libnppc9.1 libnppial9.1 libnppicc9.1 libnppicom9.1 libnppidei9.1
  libnppif9.1 libnppig9.1 libnppim9.1 libnppist9.1 libnppisu9.1 libnppitc9.1 libnpps9.1 libnvblas9.1 libnvgraph9.1 libnvidia-compute-390 libnvrtc9.1 libnvtoolsext1 libnvvm3 libthrust-dev libvdpau-dev
  nvidia-cuda-dev
```

## Install CUDA Toolkit
It includes libs required by `tensorflow-gpu`, e.g. libcublas. 
In tensorflow 1.12, it requires libcublas:9.0. By default in Ubuntu18.04 it has libcublas:9.1, not supported.

* Download the CUDA toolkit
  - [CUDA Toolkit 9.0 (Sept 2017)](https://developer.nvidia.com/cuda-toolkit-archive)
  - Linux > X86\_64 > Ubuntu > 16.04 > runfile > Base Installer(1.6GB)
* Install dependent libs
```
sudo apt-get install freeglut3-dev build-essential libx11-dev libxmu-dev libxi-dev libgl1-mesa-glx libglu1-mesa libglu1-mesa-dev
```
* Install cuda 9.0 toolkit
```
sudo chmod 777 cuda_9.0.176_384.81_linux.run
sudo ./cuda_9.0.176_384.81_linux.run -toolkit -samples -silent -override

(Installer must be in silent mode to use the toolkit, samples or driver command-line flags. Please also pass -silent to the installer.)

lrwxrwxrwx 1 root root 19 Dec 25 13:10 /usr/local/cuda -> /usr/local/cuda-9.0
```
  - Remove cuda 9.0 toolkit
```
sudo /usr/local/cuda/bin/uninstall_cuda_9.0.pl
```
* Set environment variables
```
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64 ${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}
```

## Install cuDNN (Optional)
* Download cuDNN v7.0.5 (Dec 5, 2017), for CUDA 9.0
  - [cuDNN Archive](https://developer.nvidia.com/rdp/cudnn-archive)
  - cuDNN v7.0.5 Runtime Library for Ubuntu16.04 (Deb)
  - cuDNN v7.0.5 Developer Library for Ubuntu16.04 (Deb)
  - cuDNN v7.0.5 Code Samples and User Guide for Ubuntu16.04 (Deb)
```
sudo dpkg -i .deb
```
* Install dependent libs for ministCUDNN
```
sudo apt-get install libfreeimage3 libfreeimage-dev
```

## Install TensorFlow CUDA
* Install tensorflow-gpu
```
pip3 install tensorflow-gpu

Installing collected packages: numpy, six, h5py, keras-applications, wheel, werkzeug, grpcio, setuptools, protobuf, markdown, tensorboard, astor, gast, absl-py, keras-preprocessing, termcolor, tensorflow-gpu
Successfully installed absl-py-0.6.1 astor-0.7.1 gast-0.2.0 grpcio-1.17.1 h5py-2.9.0 keras-applications-1.0.6 keras-preprocessing-1.0.5 markdown-3.0.1 numpy-1.15.4 protobuf-3.6.1 setuptools-40.6.3 six-1.12.0 tensorboard-1.12.1 tensorflow-gpu-1.12.0 termcolor-1.1.0 werkzeug-0.14.1 wheel-0.32.3
```
* Install libcupti-dev
```
sudo apt-get install libcupti-dev
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/extras/CUPTI/lib64
```

# Verify TensorFlow by hello example
```
import tensorflow as tf
hello = tf.constant('Hello, TensorFlow')
sess = tf.Session()
# show gpu support if anyG
sess.run(hello)
```
* CPU
```
$ python3
Python 3.6.7 (default, Oct 22 2018, 11:32:17) 
[GCC 8.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import tensorflow as tf
>>> hello = tf.constant('Hello, TensorFlow')
>>> sess = tf.Session()
2018-12-25 10:51:13.340471: I tensorflow/core/platform/cpu_feature_guard.cc:141] Your CPU supports instructions that this TensorFlow binary was not compiled to use: AVX2 FMA
>>> sess.run(hello)
b'Hello, TensorFlow'
>>>
```
* ROCm
```
$ python3
Python 3.6.7 (default, Oct 22 2018, 11:32:17) 
[GCC 8.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import tensorflow as tf
>>> hello = tf.constant('Hello, TensorFlow')
>>> sess = tf.Session()
2018-12-25 11:10:41.247769: I tensorflow/core/platform/cpu_feature_guard.cc:141] Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.1 SSE4.2 AVX AVX2 FMA
2018-12-25 11:10:41.249605: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1530] Found device 0 with properties: 
name: Vega [Radeon RX Vega]
AMDGPU ISA: gfx900
memoryClockRate (GHz) 1.59
pciBusID 0000:03:00.0
Total memory: 7.98GiB
Free memory: 7.73GiB
2018-12-25 11:10:41.249675: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1641] Adding visible gpu devices: 0
2018-12-25 11:10:41.249722: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1051] Device interconnect StreamExecutor with strength 1 edge matrix:
2018-12-25 11:10:41.249749: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1057]      0 
2018-12-25 11:10:41.249766: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1070] 0:   N 
2018-12-25 11:10:41.249865: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1189] Created TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:0 with 7524 MB memory) -> physical GPU (device: 0, name: Vega [Radeon RX Vega], pci bus id: 0000:03:00.0)
>>> sess.run(hello)
b'Hello, TensorFlow'
```
* CUDA(Not verified yet)

# Issues
> error: import tensorflow
```
ImportError: libCXLActivityLogger.so: cannot open shared object file: No such file or directory
ImportError: libhiprand.so.1: cannot open shared object file: No such file or directory
```
* fix: install ROCm libs
```
sudo apt install rocm-libs miopen-hip cxlactivitylogger

```
> Error: import tensorflow
```
ImportError: libcublas.so.9.0: cannot open shared object file: No such file or directory
```
* Fix: install CUDA Toolkit 9.0 when use CUDA stack

> Error: Import keras
```
No module named 'tensorflow.python.framework')
```
* Fix: re-install any of tensorflow, as it's a common lib for all tensorflow backend.
```
pip3 install tensorflow
pip3 install tensorflow-gpu
pip3 install tensorflow-rocm
```

> Error: import tensorflow
```
python3: Relink '/lib/x86_64-linux-gnu/libudev.so.1' with '/lib/x86_64-linux-gnu/librt.so.1' for IFUNC symbol 'clock_gettime'
Segmentation fault (core dumped)
exit python3
```
* Fix: install nvidia cuda driver, including nvidia-cuda-dev
```
sudo apt install python3-pycuda
```

> Which tensorflow is used? gpu or rocm

* install tensorflow-rocm again, switching to tensorflow-rocm by default
* if tensorflow-gpu is installed along with tensorflow-rocm, we need to install cuda toolkit and nvidia cuda driver, as tensorflow-gpu is called now.
