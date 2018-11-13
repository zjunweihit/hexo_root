---
title: gtest
date: 2018-11-12 22:02:13
categories: linux-env
tagso:
  - gtest
---

Start to enable unit test for C++.

<!-- more -->

Verified on ubuntu 16.04 and 18.04.

# Install gtest

* install gtest source package
```
sudo apt install libgtest-dev
```

* Build gtest lib
```
cd /usr/src/gtest
sudo mkdir build
cd build
sudo cmake ../
sudo make -j4
```

* move the libs( to /user/local/bin
```
/usr/src/gtest/build# ls libgtest*
libgtest.a  libgtest_main.a

sudo cp -a libgtest*.a /usr/local/lib/
```

# Integrate a test case in gtest

* Prepare functions to test
  - func.cpp
```
#include "func.h"

int add(int a, int b)
{
    return a + b;
}
```
  - func.h
```
#ifndef _FUNC_H
#define _FUNC_H

int add(int a, int b);

#endif
```
* Create a test case
  - func_test.cpp
```
#include "gtest/gtest.h"
#include "func.h"

TEST(AddTest, AddTestCase)
{
    ASSERT_EQ(2, add(1, 1));
    ASSERT_EQ(3, add(2, 1));
}
```
* Create the main function for all test cases
  - main.cpp
```
#include "gtest/gtest.h"
#include <iostream>
using namespace std;

int main(int argc,char* argv[])
{
    testing::GTEST_FLAG(output) = "xml:"; //若要生成xml结果文件
    testing::InitGoogleTest(&argc,argv); //初始化
    RUN_ALL_TESTS();                     //跑单元测
    return 0;
}
```
* Build with CMakeLists.txt, depending on `libgtest`, `libpthread`
```
cmake_minimum_required(VERSION 2.8)

add_executable(main
    main.cpp
    func.cpp
    func_test.cpp
)
target_link_libraries(main gtest pthread)
```
* Run test and result:
```
./main
[==========] Running 1 test from 1 test case.
[----------] Global test environment set-up.
[----------] 1 test from AddTest
[ RUN      ] AddTest.AddTestCase
[       OK ] AddTest.AddTestCase (0 ms)
[----------] 1 test from AddTest (0 ms total)

[----------] Global test environment tear-down
[==========] 1 test from 1 test case ran. (0 ms total)
[  PASSED  ] 1 test.
```
* xml repot
```
$ ./main --gtest_output=xml

$ ls *.xml
test_detail.xml

$ cat test_detail.xml
<?xml version="1.0" encoding="UTF-8"?>
<testsuites tests="1" failures="0" disabled="0" errors="0" timestamp="2018-11-12T22:45:07" time="0" name="AllTests">
  <testsuite name="AddTest" tests="1" failures="0" disabled="0" errors="0" time="0">
    <testcase name="AddTestCase" status="run" time="0" classname="AddTest" />
  </testsuite>
</testsuites>

```

# Reference

* https://www.cnblogs.com/Jessica-jie/p/6704388.html
* https://www.cnblogs.com/hcu5555/archive/2015/04/30/4468847.html
* https://www.cnblogs.com/coderzh/archive/2009/04/06/1426755.html
