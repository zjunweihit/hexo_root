---
title: Python how to build a package and publish
date: 2019-10-29 09:11:42
categories:
  - Programming
tags:
  - Python
---

Build a wheel package and publish.

<!--more-->

# Pacaking a project

* prepare the project
```
pkg_demo
├── LICENSE
├── pkgdemo
│   ├── __init__.py
│   └── demo.py
├── README.md
├── requirements.txt
└── setup.py
```
* prepare pakcaging tool
```
$ pip3 install setuptools
$ pip3 install twine
```
* build wheel
```
$ python3 setup.py bdist_wheel --universal

pkg_demo
├── build
│   ├── bdist.linux-x86_64
│   └── lib
│       └── pkgdemo
│           ├── demo.py
│           └── __init__.py
├── dist
│   └── pkgdemo-0.0.1-py2.py3-none-any.whl
├── LICENSE
├── pkgdemo
│   ├── demo.py
│   └── __init__.py
├── pkgdemo.egg-info
│   ├── dependency_links.txt
│   ├── PKG-INFO
│   ├── SOURCES.txt
│   └── top_level.txt
├── README.md
├── requirements.txt
└── setup.py
```
* upload the package to pypi
```
$ twine upload dist/zjw_pkgdemo-0.0.1-py2.py3-none-any.whl
```
* create a virtualenv and import the package
```
pip3 install zjw_pkgdemo
Collecting zjw_pkgdemo
  Downloading https://files.pythonhosted.org/packages/13/e6/d5a4446281f81361ce11cb6d1a56df97771cbef343cfce3554bd4a920eb5/zjw_pkgdemo-0.0.1-py2.py3-none-any.whl
Installing collected packages: zjw-pkgdemo
Successfully installed zjw-pkgdemo-0.0.1

>>> import pkgdemo.demo
>>> pkgdemo.demo.info()
This is a demo
```

* reference
  - https://packaging.python.org/tutorials/packaging-projects/
