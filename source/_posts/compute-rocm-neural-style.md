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

# Install libs for neural style
* Get the neural repository
```
git clone https://github.com/anishathalye/neural-style.git
```
* Install libs by requirement file except `tensorflow-gpu`
```
pip3 install -r requirements.txt

cat requirements.txt

numpy
Pillow  # provides PIL
scipy
tensorflow-gpu >= 1.0  # installs tensorflow with GPU support, should still work even without GPU
                       # Not necessary for tensorflow-rocm
```
* Download vgg 19 mat data
  - [imagenet-vgg-verydeep-19.mat](http://www.vlfeat.org/matconvnet/models/imagenet-vgg-verydeep-19.mat)
* Run neural sytle
```
python3 ./neural_style.py --overwrite --network ~/vgg-19-mat/imagenet-vgg-verydeep-19.mat --content origin/night.jpeg --styles examples/1-style.jpg --output output/night.jpg
```

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
