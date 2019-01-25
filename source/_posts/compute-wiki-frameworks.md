---
title: Machine Learning Frameworks
date: 2019-01-25 10:25:39
categories:
  - Compute
tags:
  - Framework
  - Wiki
---

Introduce machine learning frameworks.

<!--more-->

# Tensorflow

* Developed by Google
* Learning tensorflow
  - [TensorFlow and deep learning, without a PhD](https://codelabs.developers.google.com/codelabs/cloud-tensorflow-mnist/#0)

* About Tensor
  - [What is a Tensor](https://www.guru99.com/tensor-tensorflow.html)
  > A "tensor" is like a matrix but with an arbitrary number of dimensions. A 1-dimensional tensor is a vector. A 2-dimensions tensor is a matrix. And then you can have tensors with 3, 4, 5 or more dimensions.


# Keras

* the Keras framework now has both TensorFlow and Theano as backends.
* Keras is a particularly easy to use deep learning framework.
* Now, any model previously written in Keras can now be run on top of TensorFlow.

# Theano

* Theano was originally developed as a symbolic math processor at the University of Montreal, for the purpose of performing symbolic differentiation or integration, on complicated non-linear functions.


# Caffe

* was developed at the Berkeley Vision and Learning Center (BVLC)
* useful for performing image analysis(CNN) and regional analysis(RCNN)


# Torch

* was originally developed at NYU, and is based upon the scripting language Lua, which was designed to be portable, fast, extensible, and easy to use in development.
* Facebook
* Torch is probably the easiest to get up and running, if you are using an Ubuntu platform. If you are using CentOS, workarounds are possible, but require careful modification of OS packages.

# Neon

* Adding to the growing trend of proprietary machine learning software being turned into open source projects, Nervana Systems, now an Intel subsidiary, has open sourced Neon
* deep learning framework has been ranked as the fastest framework
* Python, similar as Keras

# MIOpen

* Deep Learning acceleration on top of ROCm, CNN FFT GEMM support OpenCL and HIP

# IBM Deep Learning Platform
* IBM's platform is the only deep learning platform/framework examined here which does not consist of single framework. Instead it is a whole platform ? from OS to programming frameworks ? including Spark, and SystemML 
* `Apache Spark` is a prime component of IBM's deep learning platform, is designed for cluster computing and contains MLlib, a distributed machine learning framework.

# Reference
* [Deep Learning Frameworks](https://www.microway.com/hpc-tech-tips/deep-learning-frameworks-survey-tensorflow-torch-theano-caffe-neon-ibm-machine-learning-stack/)
