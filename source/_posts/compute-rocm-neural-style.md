---
title: TensorFlow Example> Neural Style
date: 2018-12-25 12:40:57
categories:
  - Compute
tags:
  - TensorFlow
  - ROCm
---

Run an example of neural style by TensorFlow powered by ROCm.

<!--more-->
# Environment
* Ubuntu 18.04
* tensorflow 1.12.0
* ROCm 2.0.89

# Install TensorFlow ROCm

* [TensorFlow powered by ROCm](https://zjunweihit.github.io/2018/12/25/compute-env-tensorflow/)

# Install Nvidia CUDA driver
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

# Install CUDA Toolkit
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
* Set environment variables
```
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64 ${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}
```

# Install cuDNN (Optional)
* Download cuDNN v7.0.5 (Dec 5, 2017), for CUDA 9.0
  - [cuDNN Archive](https://developer.nvidia.com/rdp/cudnn-archive)
  - cuDNN v7.0.5 Runtime Library for Ubuntu16.04 (Deb)
  - cuDNN v7.0.5 Developer Library for Ubuntu16.04 (Deb)
  - cuDNN v7.0.5 Code Samples and User Guide for Ubuntu16.04 (Deb)
```
sudo dpkg -i *.deb
```
* Install dependent libs for ministCUDNN
```
sudo apt-get install libfreeimage3 libfreeimage-dev
```

# Install TensorFlow CUDA (Optional)
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

# Install libs for neural style
* Get the neural repository
```
git clone https://github.com/anishathalye/neural-style.git
```
* Install libs by requirement file
```
pip3 install -r requirements.txt

cat requirements.txt

numpy
Pillow  # provides PIL
scipy
tensorflow-gpu >= 1.0  # installs tensorflow with GPU support, should still work even without GPU
                       # installed already
```
* Download vgg 19 mat data
  - [imagenet-vgg-verydeep-19.mat](http://www.vlfeat.org/matconvnet/models/imagenet-vgg-verydeep-19.mat)
* Run neural sytle
```
python3 ./neural_style.py --overwrite --network ~/vgg-19-mat/imagenet-vgg-verydeep-19.mat --content origin/night.jpeg --styles examples/1-style.jpg --output output/night.jpg
```

# Issues
> Import tensorflow
```
ImportError: libcublas.so.9.0: cannot open shared object file: No such file or directory
```
* Fix: install CUDA Toolkit 9.0

> Import keras
```
No module named 'tensorflow.python.framework')
```
* Fix: re-install any of tensorflow, as it's a common lib for all tensorflow backend.
```
pip3 install tensorflow
pip3 install tensorflow-gpu
pip3 install tensorflow-rocm
```

> error: import tensorflow
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

# Result
* Original picture about night
![Image](/image/compute-rocm-neural-style/night-org.jpg)

* Learn this picture's style
![Image](/image/compute-rocm-neural-style/1-style.jpg)

* Apply to original night and become a new night picture with above style
![Image](/image/compute-rocm-neural-style/night-output.jpg)


# Reference
* [neural-style](https://github.com/anishathalye/neural-style)
* [在Ubuntu 18.04安装Tensorflow](https://blog.csdn.net/B9Q8e64lO6mm/article/details/81117285)
