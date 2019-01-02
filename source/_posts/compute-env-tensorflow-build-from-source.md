---
title: TensorFlow Build From Source
date: 2019-01-02 13:27:57
categories:
  - Compute
tags:
  - TensorFlow
  - ROCm
---

How to build TensorFlow ROCm. (Verified in Ubuntu 18.04)

<!--more-->

# Software in Ubuntu 18.04
* Kernel
```
uname -r
4.15.0-43-generic
```

* gcc
```
gcc --version
gcc (Ubuntu 7.3.0-27ubuntu1~18.04) 7.3.0
Copyright (C) 2017 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

* python3
```
python3 --version
Python 3.6.7
```

# Install Bazel

Bazel is a free software tool that allows for the automation of building and testing of software. The company Google uses the build tool Blaze internally and released and open-sourced part of the Blaze tool as Bazel, named as an anagram of Blaze.

## Install bazel online
* Install JDK 8
```
sudo apt-get install openjdk-8-jdk
```

* Add Bazel source list
```
echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list

curl https://bazel.build/bazel-release.pub.gpg | sudo apt-key add -
```

* Install Bazel
```
sudo apt update && sudo apt install bazel

# upgrade only
sudo apt install --only-upgrade bazel
```

## Install bazel by binary

* Pre-install required packages
```
sudo apt-get install pkg-config zip g++ zlib1g-dev unzip python
```

* Download 0.19.2, due to tensorflow-upstream.git [downgrading bazel version from 0.20.0 to 0.19.2 to workaround issue #24124](https://github.com/ROCmSoftwarePlatform/tensorflow-upstream/commit/d9ee32ca7db3a6cb0897d26d3cd9bd766e88fe8e#diff-d815c171a670ccb45b850c82a9b92d0c)
  - [bazel release page](https://github.com/bazelbuild/bazel/releases)
  - bazel-0.19.2-installer-linux-x86_64.sh

* Install by script
```
chmod +x bazel-0.19.2-installer-linux-x86_64.sh
./bazel-0.19.2-installer-linux-x86_64.sh --user
```
  - `--user` install Bazel to `$HOME/bin` folder and export the path in `.bashrc`

# Download the TensorFlow source
* tensorflow
```
git clone https://github.com/tensorflow/tensorflow.git
```
* tensorflow ROCm upstream
```
git clone https://github.com/ROCmSoftwarePlatform/tensorflow-upstream.git
```

# Build TensorFlow ROCm
* Configuration
```
$ yes "" | TF_NEED_ROCM=1 PYTHON_BIN_PATH=/usr/bin/python3 ./configure

WARNING: --batch mode is deprecated. Please instead explicitly shut down your Bazel server using the command "bazel shutdown".                                                                              You have bazel 0.19.2 installed.
Found possible Python library paths:
  /usr/local/lib/python3.6/dist-packages
  /usr/lib/python3/dist-packages
Please input the desired Python library path to use.  Default is [/usr/local/lib/python3.6/dist-packages]
Do you wish to build TensorFlow with XLA JIT support? [Y/n]: XLA JIT support will be enabled for TensorFlow.

Do you wish to build TensorFlow with OpenCL SYCL support? [y/N]: No OpenCL SYCL support will be enabled for TensorFlow.

Do you wish to build TensorFlow with CUDA support? [y/N]: No CUDA support will be enabled for TensorFlow.

Do you wish to download a fresh release of clang? (Experimental) [y/N]: Clang will not be downloaded.

Do you wish to build TensorFlow with MPI support? [y/N]: No MPI support will be enabled for TensorFlow.

Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native -Wno-sign-compare]:

Would you like to interactively configure ./WORKSPACE for Android builds? [y/N]: Not configuring the WORKSPACE for Android builds.

Preconfigured Bazel build configs. You can use any of the below by adding "--config=<>" to your build command. See .bazelrc for more details.
        --config=mkl            # Build with MKL support.
        --config=monolithic     # Config for mostly static monolithic build.
        --config=gdr            # Build with GDR support.
        --config=verbs          # Build with libverbs support.
        --config=ngraph         # Build with Intel nGraph support.
        --config=dynamic_kernels        # (Experimental) Build kernels into separate shared objects.

Preconfigured Bazel build configs to DISABLE default on features:
        --config=noaws          # Disable AWS S3 filesystem support.
        --config=nogcp          # Disable GCP support.
        --config=nohdfs         # Disable HDFS support.
        --config=noignite       # Disable Apacha Ignite support.
        --config=nokafka        # Disable Apache Kafka support.
        --config=nonccl         # Disable NVIDIA NCCL support.
Configuration finished
```

* Build tensorflow with ROCm
```
bazel build --config=opt --config=rocm //tensorflow/tools/pip_package:build_pip_package
```
  - If the memory is constrained, build it as below: RAM (in MB), CPU (in cores) and available I/O (1.0 being average workstation)
```
bazel build --local_resources 4096,4.0,1.0 -j 4 --config=opt --config=rocm //tensorflow/tools/pip_package:build_pip_package
```

* Generate the tensorflow package
```
bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
2019年 01月 02日 星期三 12:10:26 CST : === Preparing sources in dir: /tmp/tmp.TfGUlmequ5
~/repo/tensorflow-upstream ~/repo/tensorflow-upstream
~/repo/tensorflow-upstream
2019年 01月 02日 星期三 12:10:43 CST : === Building wheel
warning: no files found matching '*.pyd' under directory '*'
warning: no files found matching '*.pd' under directory '*'
warning: no files found matching '*.dll' under directory '*'
warning: no files found matching '*.lib' under directory '*'
warning: no files found matching '*.h' under directory 'tensorflow/include/tensorflow'
warning: no files found matching '*' under directory 'tensorflow/include/Eigen'
warning: no files found matching '*.h' under directory 'tensorflow/include/google'
warning: no files found matching '*' under directory 'tensorflow/include/third_party'
warning: no files found matching '*' under directory 'tensorflow/include/unsupported'
2019年 01月 02日 星期三 12:11:02 CST : === Output wheel file is in: /tmp/tensorflow_pkg
```
* Install the wheel package with ROCm
```
pip3 install  /tmp/tensorflow_pkg/tensorflow-1.12.0-cp36-cp36m-linux_x86_64.whl

Processing /tmp/tensorflow_pkg/tensorflow-1.12.0-cp36-cp36m-linux_x86_64.whl
Collecting tensorboard<1.13.0,>=1.12.0 (from tensorflow==1.12.0)
  Using cached https://files.pythonhosted.org/packages/bd/7e/528c868bb8a0542c8a5686ff3a08502d2691bd50499c6e55f8989fa8e5a0/tensorboard-1.12.1-py3-none-any.whl
Collecting grpcio>=1.8.6 (from tensorflow==1.12.0)
  Using cached https://files.pythonhosted.org/packages/3b/bb/701d879849c938028c09fdb5405dbde7c86644bbbb90098094002db23ded/grpcio-1.17.1-cp36-cp36m-manylinux1_x86_64.whl
Collecting numpy>=1.13.3 (from tensorflow==1.12.0)
  Using cached https://files.pythonhosted.org/packages/ff/7f/9d804d2348471c67a7d8b5f84f9bc59fd1cefa148986f2b74552f8573555/numpy-1.15.4-cp36-cp36m-manylinux1_x86_64.whl
Collecting tensorflow-estimator>=1.10.0 (from tensorflow==1.12.0)
  Using cached https://files.pythonhosted.org/packages/e7/5a/bace04b34b4a3214ed105899a27f3e94f0468b64bce46125ff42f495b1fc/tensorflow_estimator-1.10.12-py2.py3-none-any.whl
Collecting six>=1.10.0 (from tensorflow==1.12.0)
  Using cached https://files.pythonhosted.org/packages/73/fb/00a976f728d0d1fecfe898238ce23f502a721c0ac0ecfedb80e0d88c64e9/six-1.12.0-py2.py3-none-any.whl
Collecting gast>=0.2.0 (from tensorflow==1.12.0)
Collecting wheel>=0.26 (from tensorflow==1.12.0)
  Using cached https://files.pythonhosted.org/packages/ff/47/1dfa4795e24fd6f93d5d58602dd716c3f101cfd5a77cd9acbe519b44a0a9/wheel-0.32.3-py2.py3-none-any.whl
Collecting protobuf>=3.6.1 (from tensorflow==1.12.0)
  Using cached https://files.pythonhosted.org/packages/c2/f9/28787754923612ca9bfdffc588daa05580ed70698add063a5629d1a4209d/protobuf-3.6.1-cp36-cp36m-manylinux1_x86_64.whl
Collecting absl-py>=0.1.6 (from tensorflow==1.12.0)
Collecting keras-applications>=1.0.6 (from tensorflow==1.12.0)
  Using cached https://files.pythonhosted.org/packages/3f/c4/2ff40221029f7098d58f8d7fb99b97e8100f3293f9856f0fb5834bef100b/Keras_Applications-1.0.6-py2.py3-none-any.whl
Collecting astor>=0.6.0 (from tensorflow==1.12.0)
  Using cached https://files.pythonhosted.org/packages/35/6b/11530768cac581a12952a2aad00e1526b89d242d0b9f59534ef6e6a1752f/astor-0.7.1-py2.py3-none-any.whl
Collecting keras-preprocessing>=1.0.5 (from tensorflow==1.12.0)
  Using cached https://files.pythonhosted.org/packages/fc/94/74e0fa783d3fc07e41715973435dd051ca89c550881b3454233c39c73e69/Keras_Preprocessing-1.0.5-py2.py3-none-any.whl
Collecting termcolor>=1.1.0 (from tensorflow==1.12.0)
Collecting werkzeug>=0.11.10 (from tensorboard<1.13.0,>=1.12.0->tensorflow==1.12.0)
  Using cached https://files.pythonhosted.org/packages/20/c4/12e3e56473e52375aa29c4764e70d1b8f3efa6682bef8d0aae04fe335243/Werkzeug-0.14.1-py2.py3-none-any.whl
Collecting markdown>=2.6.8 (from tensorboard<1.13.0,>=1.12.0->tensorflow==1.12.0)
  Using cached https://files.pythonhosted.org/packages/7a/6b/5600647404ba15545ec37d2f7f58844d690baf2f81f3a60b862e48f29287/Markdown-3.0.1-py2.py3-none-any.whl
Collecting mock>=2.0.0 (from tensorflow-estimator>=1.10.0->tensorflow==1.12.0)
  Using cached https://files.pythonhosted.org/packages/e6/35/f187bdf23be87092bd0f1200d43d23076cee4d0dec109f195173fd3ebc79/mock-2.0.0-py2.py3-none-any.whl
Collecting setuptools (from protobuf>=3.6.1->tensorflow==1.12.0)
  Using cached https://files.pythonhosted.org/packages/37/06/754589caf971b0d2d48f151c2586f62902d93dc908e2fd9b9b9f6aa3c9dd/setuptools-40.6.3-py2.py3-none-any.whl
Collecting h5py (from keras-applications>=1.0.6->tensorflow==1.12.0)
  Using cached https://files.pythonhosted.org/packages/30/99/d7d4fbf2d02bb30fb76179911a250074b55b852d34e98dd452a9f394ac06/h5py-2.9.0-cp36-cp36m-manylinux1_x86_64.whl
Collecting pbr>=0.11 (from mock>=2.0.0->tensorflow-estimator>=1.10.0->tensorflow==1.12.0)
  Using cached https://files.pythonhosted.org/packages/f3/04/fddc1c2dd75b256eda4d360024692231a2c19a0c61ad7f4a162407c1ab58/pbr-5.1.1-py2.py3-none-any.whl
Installing collected packages: werkzeug, six, grpcio, setuptools, protobuf, wheel, numpy, markdown, tensorboard, pbr, mock, absl-py, tensorflow-estimator, gast, h5py, keras-applications, astor, keras-preprocessing, termcolor, tensorflow
Successfully installed absl-py-0.6.1 astor-0.7.1 gast-0.2.0 grpcio-1.17.1 h5py-2.9.0 keras-applications-1.0.6 keras-preprocessing-1.0.5 markdown-3.0.1 mock-2.0.0 numpy-1.15.4 pbr-5.1.1 protobuf-3.6.1 setuptools-40.6.3 six-1.12.0 tensorboard-1.12.1 tensorflow-1.12.0 tensorflow-estimator-1.10.12 termcolor-1.1.0 werkzeug-0.14.1 wheel-0.32.3
```

# Verify TensorFlow by hello exmaple
* tensorflow-upstream(develop-upstream)
```
# go out of buidling folder
$ cd ~
$ python3
Python 3.6.7 (default, Oct 22 2018, 11:32:17)
[GCC 8.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import tensorflow as tf
2019-01-02 14:21:18.320305: E tensorflow/stream_executor/rocm/rocm_blas.cc:2335] Unable to register rocBLAS factory: Attempting to register factory for plugin rocBLAS when one has already been registered
2019-01-02 14:21:18.320333: E tensorflow/stream_executor/rocm/rocm_dnn.cc:5125] Unable to register MIOpen factory: Attempting to register factory for plugin MIOpen when one has already been registered
>>> hello = tf.constant('Hello, TensorFlow')
>>> sess = tf.Session()
2019-01-02 14:21:29.515332: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1532] Found device 0 with properties:
name: Vega [Radeon RX Vega]
AMDGPU ISA: gfx900
memoryClockRate (GHz) 1.59
pciBusID 0000:03:00.0
Total memory: 7.98GiB
Free memory: 7.73GiB
2019-01-02 14:21:29.515387: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1643] Adding visible gpu devices: 0
2019-01-02 14:21:29.515418: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1054] Device interconnect StreamExecutor with strength 1 edge matrix:
2019-01-02 14:21:29.515426: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1060]      0
2019-01-02 14:21:29.515433: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1073] 0:   N
2019-01-02 14:21:29.515488: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1190] Created TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:0 with 7524 MB memory) -> physical GPU (device: 0, name: Vega [Radeon RX Vega], pci bus id: 0000:03:00.0)
>>> sess.run(hello)
b'Hello, TensorFlow'
```
* tensorflow(master), enabling CPU only(not sure the reason)
```
python3
Python 3.6.7 (default, Oct 22 2018, 11:32:17)
[GCC 8.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import tensorflow as tf
>>> hello = tf.constant('Hello, TensorFlow')
>>> sess.run(hello)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'sess' is not defined
>>> sess = tf.Session()
2019-01-02 14:36:01.000696: I tensorflow/core/platform/profile_utils/cpu_utils.cc:94] CPU Frequency: 3408000000 Hz
2019-01-02 14:36:01.001841: I tensorflow/compiler/xla/service/service.cc:161] XLA service 0x7f39586d2330 executing computations on platform Host. Devices:
2019-01-02 14:36:01.001915: I tensorflow/compiler/xla/service/service.cc:168]   StreamExecutor device (0): <undefined>, <undefined>
>>> sess.run(hello)
b'Hello, TensorFlow'
```

# Reference
* [Installing Bazel on Ubuntu](https://docs.bazel.build/versions/master/install-ubuntu.html)
* [TensorFlow ROCm build readme](https://github.com/ROCmSoftwarePlatform/tensorflow-upstream/blob/develop-upstream/rocm_docs/tensorflow-build-from-source.md)
