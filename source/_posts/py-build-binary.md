---
title: Python build a binary
date: 2019-11-09 14:31:44
categories:
  - Programming
tags:
  - Python
---

Build a binary from python source.

<!--more-->

# Build a python binary
* Install the tool
```
pip3 install pyinstaller
```
* simply way
```
pyinstaller -F ttViewer.py
```
* build with key
```
pyinstaller -F ttViewer.py --key 133
```

