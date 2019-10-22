---
title: C++ string operation
date: 2019-10-22 10:45:39
categories:
  - Programing
tags:
  - C++
---

String operation.

<!--more-->

# Find a key word

* str.`find()` <[cpp_ref](http://www.cplusplus.com/reference/string/string/find/)>
  - string index starts from 0
```
std::string str("12345678901234567890");

std::size_t found = str.find("3");
if (found!=std::string::npos)
  std::cout << "first '3' found at: " << found << '\n';

std::size_t found2 = str.find("3", found + 1);
if (found2!=std::string::npos)
  std::cout << "second '3' found at: " << found2 << '\n';

----
first '3' found at: 2
second '3' found at: 12
```
  - `find_first_of()` and `find_last_of()`
```
found = str.find_first_of("3");
if (found!=std::string::npos)
  std::cout << "first '3' found at: " << found << '\n';
found = str.find_last_of("3");
if (found!=std::string::npos)
  std::cout << "last '3' found at: " << found << '\n';

----
first '3' found at: 2
last '3' found at: 12
```
* str.rfind(): find the key from right to left, at a specific position [cpp_ref](http://www.cplusplus.com/reference/string/string/rfind/)
```
std::string str("12345678901234567890");

std::size_t found = str.rfind("3");
if (found!=std::string::npos)
  std::cout << "last '3' found at: " << found << '\n';

std::size_t found2 = str.rfind("3", found - 1);
if (found2!=std::string::npos)
  std::cout << "second right '3' found at: " << found2 << '\n';

found = str.rfind("3", 2);
if (found!=std::string::npos)
  std::cout << "first '3' found at: " << found << '\n';

----
last '3' found at: 12
second right '3' found at: 2
first '3' found at: 2
```

# Get a part from a string

* str.substr(pos, len) [cpp_ref](http://www.cplusplus.com/reference/string/string/substr/)
```
std::string str("12345678901234567890");

str = str.substr(3, 2);
std::cout << str << '\n';

----
45
```


