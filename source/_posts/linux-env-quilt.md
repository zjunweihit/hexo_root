---
title: 玩转quilt
date: 2016-07-02 21:25:47
categories: linux-env
tags:
  - quilt
---

Create the patch freely

<!--more-->

# quilt settings #
It is a reference for quilt settings:
```
# Example /etc/quilt.quiltrc

# Options passed to GNU diff when generating patches
QUILT_DIFF_OPTS="--show-c-function"
# Options passed to GNU patch when applying patches
#QUILT_PATCH_OPTS="--ignore-whitespace"
# Options passed to diffstat when generating patch statistics
QUILT_DIFFSTAT_OPTS="-f0"

# Options to pass to commands (QUILT_${COMMAND}_ARGS)
#QUILT_PUSH_ARGS="--color=auto"
QUILT_PUSH_ARGS=""
#QUILT_DIFF_ARGS="--no-timestamps --color=auto"
QUILT_DIFF_ARGS="--no-timestamps"
QUILT_REFRESH_ARGS="--no-timestamps --backup --diffstat --strip-trailing-whitespace -p ab"

# The directory in which patches are found (defaults to "patches").
#QUILT_PATCHES=patches

# Prefix all patch names with the relative path to the patch?
QUILT_PATCHES_PREFIX=yes

# Use a specific editor for quilt (defaults to the value of $EDITOR before
# sourcing this configuration file, or vi if $EDITOR wasn't set).
EDITOR=vim
```

# quilt operations for daily work #
1. 删除与patch相关的文件.
  * 删除前状态
```
$ quilt files
related_file_a.c
related_file_b.c
```
  * 删除(quilt remove)
```
$ quilt remove related_file_b.c
File related_file_b.c removed from patch patches/top.patch
```
  * 删除后状态
```
$ quilt files
related_file_a.c
```
  * 若要删除指定patch的某个相关文件
```
$ quilt remove -P <patch_name> <related_file_name>
```
1. 查看&编辑patch header信息
  * 默认查看top patch的header
```
$ quilt header
```
  * 也可以指定patch
```
$ quilt header patches/XX/YY/ZZ.patch
```
  * 编辑patch header
```
$ quilt header -e
```
  * 同理也可以指定patch
```
$ quilt header -e patches/XX/YY/ZZ.patch
```
1. 查看patch与相关文件的关系
  * 查看patch与哪些文件相关
```
$ quilt files
or
$ quilt files <patch_name>
```
  * 查看文件与哪些patch相关
```
$ quilt patches <file_name>
```
1. combine patches
  * combine的patches(a.patch, b.patch, c.patch)在serial文件中要是连续的.如果不是,要reposition
```
in serial file
XX/1.patch
XX/a.patch <--- from
XX/b.patch
XX/c.patch <--- to
XX/d.patch
```
  * 需要combine的patches要已经quilt pu, 即quilt to在c.patch(如果在c.patch之后比较远的patch,会有warning提示,最近有patch改动过,相关文件)
  * combine
```
$ quilt diff --combine XX/a.patch -P XX/c.patch > patches/XX/a_b_c.patch
```
  * unapply patches
```
$ quilt po XX/1.patch
```
  * replace a.patch, b.patch, c.patch with combined patch in serial file
```
XX/1.patch
#XX/a.patch
#XX/b.patch
#XX/c.patch
XX/a_b_c.patch
XX/d.patch
```
  * apply combined patch
```
$ quilt pu -a
```
  * git status (no source code change)
1. Import a patch
```
$ quilt import ../a.patch -P fix/a.patch
Importing patch ../a.patch (stored as patches/fix/a.patch)
```

