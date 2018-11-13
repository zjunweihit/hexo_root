---
title: CMake
date: 2018-11-12 16:57:55
categories: linux-env
tags:
  - cmake
---

Go to make build easier.

<!-- more -->

# Build document
* CMakeLists.txt
  - like Makefile
* add_executable(`<name>` source1 [source2 ...])
  - Adds an executable target called `<name>` to be built from the source files listed in the command invocation.
  - The `<name>` corresponds to the logical target name and must be globally unique within a project.
```
* c++11 support
  - add_compile_options() will add the options to gcc and g++
  - set(CMAKE_CXX_FLAGS) works for g++ only
```
if(CMAKE_COMPILER_IS_GNUCXX)
    # add_compile_options(-std=c++11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
    message(STATUS "optional:-std=c++11")
endif(CMAKE_COMPILER_IS_GNUCXX)
```

# single source file
add_executable(stest single_list.cpp)

# multiple source files
add_executable(main
    create1.cpp
    create2.cpp
    create3.cpp
)
```

# Build a single file
* a single source file is going to be built into an executable binary
```
cmake_minimum_required(VERSION 2.8)

add_executable(stest single_list.cpp)
```

# Build multiple files in sub directory
* serveral files in different sub direcotries, build them all in the root directory, and also could build each in a sub directory.
```
.
├── CMakeLists.txt
├── dlist
│   ├── CMakeLists.txt
│   └── double_list.cpp
└── slist
    ├── CMakeLists.txt
    └── single_list.cpp
```
* slist CMakeLists.txt
```
cmake_minimum_required(VERSION 2.8)

add_executable(stest single_list.cpp)
```
* dlist CMakeLists.txt
```
cmake_minimum_required(VERSION 2.8)

add_executable(dtest double_list.cpp)
```
* folder root
```
cmake_minimum_required(VERSION 2.8)

add_subdirectory(slist)
add_subdirectory(dlist)

```

# Reference
* [cmake-tutorial](https://cmake.org/cmake-tutorial/)
  - [cmake-tutorial ZN in CSDN](https://blog.csdn.net/ajianyingxiaoqinghan/article/details/70229799)
* [cmake-buildsystem(7)](https://cmake.org/cmake/help/v3.0/manual/cmake-buildsystem.7.html)
* [cmake doc](https://cmake.org/documentation/)
