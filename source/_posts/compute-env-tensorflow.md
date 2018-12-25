---
title: TensorFlow powered by ROCm
date: 2018-12-25 10:33:56
categories:
  - Compute
tags:
  - TensorFlow
  - ROCm
---

How to enable TensorFlow powered by ROCm.
(Verified in Ubuntu 18.04)

<!--more-->
# software in Ubuntu 18.04
  - tensorflow 1.12.0
  - ROCm 2.0.89

# Install ROCm
  - [ROCm Installation Guide](https://zjunweihit.github.io/2018/12/25/compute-env-rocm-md/)
  - **includes ROCm libs**

# Python3 libs
Python3 3.6.5 is installed by default
```
sudo apt install python3-pip

The following NEW packages will be installed:
  dh-python libexpat1-dev libpython3-dev libpython3.6-dev python-pip-whl python3-dev python3-distutils python3-lib2to3 python3-pip python3-setuptools python3-wheel python3.6-dev
```

# tensorflow for CPU(optional)
```
pip3 install tensorflow

Installing collected packages: astor, six, numpy, keras-preprocessing, h5py, keras-applications, setuptools, protobuf, wheel, markdown, werkzeug, grpcio, tensorboard, gast, termcolor, absl-py, tensorflow
Successfully installed absl-py-0.6.1 astor-0.7.1 gast-0.2.0 grpcio-1.17.1 h5py-2.9.0 keras-applications-1.0.6 keras-preprocessing-1.0.5 markdown-3.0.1 numpy-1.15.4 protobuf-3.6.1 setuptools-40.6.3 six-1.12.0 tensorboard-1.12.1 tensorflow-1.12.0 termcolor-1.1.0 werkzeug-0.14.1 wheel-0.32.3
```

# tensorflow for ROCm
```
pip3 install tensorflow-rocm

Installing collected packages: wheel, six, setuptools, protobuf, markdown, grpcio, werkzeug, numpy, tensorboard, gast, keras-preprocessing, absl-py, astor, termcolor, h5py, keras-applications, tensorflow-rocm
Successfully installed absl-py-0.6.1 astor-0.7.1 gast-0.2.0 grpcio-1.17.1 h5py-2.9.0 keras-applications-1.0.6 keras-preprocessing-1.0.5 markdown-3.0.1 numpy-1.15.4 protobuf-3.6.1 setuptools-40.6.3 six-1.12.0 tensorboard-1.12.1 tensorflow-rocm-1.12.0 termcolor-1.1.0 werkzeug-0.14.1 wheel-0.32.3
```

# verify tensorflow by hello example
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
