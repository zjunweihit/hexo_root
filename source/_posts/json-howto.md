---
title: How to use Json
date: 2019-10-22 13:21:12
categories:
  - Programming
tags:
  - Json
  - C++
---
* json format and C++ support

<!--more-->

# Json Format

Data could be classified as
  - scalar: a variable
  - sequence: a list or array
  - mapping: key/value, hash or dictionary

Data expression
  - same kind of data could be seprated by ","
  - mapping uses ":"
  - array(list) uses "[]"
  - mapping set uses "{}"

Example
```
{
    "version": 0.2,
    "vc_kernel_args": [
        [0, "float"],
        [1, "float"],
        [2, "float"],
        [3, "int"],
        [4, "int"]
    ],
    "hsaco_kernel_obj": {
        "symbol"           : "vector_add",
        "dim"              : 1,
        "workgroup_size_x" : 16,
        "workgroup_size_y" : 1,
        "workgroup_size_z" : 1,
        "grid_size_x"      : 16,
        "grid_size_y"      : 1,
        "grid_size_z"      : 1
    }
}
```

# Json for Modern C++

* download release from page [nlohmann json release](https://github.com/nlohmann/json/releases)
* include the header
```
#include "json.hpp"
```
* Parse json file
```
#include <fstream>
#include "json.hpp"

using json = nlohmann::json;

std::ifstream f(jsonFile);
if (!f.is_open()) {
   std::cout << "Failed to open " << jsonFile << std::endl;
   return;
}

json j = json::parse(f);

std::cout << j["version"] << std::endl;
std::cout << j["vc_kernel_args"].size() << std::endl;

// dump the string
std::string str = j["vc_kernel_args"][0][1].dump(); // it has "", this is "float"
str.substr(1, str.size() - 2); // remove the "" at the beginning and end
```

# Reference
* http://www.ruanyifeng.com/blog/2009/05/data_types_and_json.html
