---
title: CVS
date: 2016-07-02 21:37:40
categories: linux-env
tags:
  - cvs
---

Manage the code and file with CVS

<!--more-->

# CVS operations for daily work #
* 更新当前目录下的所有文件,如果有新添加的文件也自动创建.
```
$ cvs up -dP
help info:
   -P      Prune empty directories.
   -d      Build directories, like checkout does.
```
* cvs update help
```
Usage: cvs update [-APCdflRp] [-k kopt] [-r rev] [-D date] [-j rev]
    [-I ign] [-W spec] [files...]
        -A      Reset any sticky tags/date/kopts.
        -P      Prune empty directories.
        -C      Overwrite locally modified files with clean repository copies.
        -d      Build directories, like checkout does.
        -f      Force a head revision match if tag/date not found.
        -l      Local directory only, no recursion.
        -R      Process directories recursively.
        -p      Send updates to standard output (avoids stickiness).
        -k kopt Use RCS kopt -k option on checkout. (is sticky)
        -r rev  Update using specified revision/tag (is sticky).
        -D date Set date to update from (is sticky).
        -j rev  Merge in changes made between current revision and rev.
        -I ign  More files to ignore (! to reset).
        -W spec Wrappers specification line.
```

