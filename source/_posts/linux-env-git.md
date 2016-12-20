---
title: 玩转git
date: 2016-07-02 20:37:32
categories: linux-env
tags:
  - SCM
---

Let's play with git ^^

<!--more-->

# git config #
* git config的操作
  - --global: 对所有git repo, 否则只会影响当前git repo的.git/config文件.
  - --unset: 删除alias.XX的设置
```
# git config --unset alias.XX YY_ZZ_UU
```
* git config user.name "your name"
* git config user.email yourname@email_server
* git config core.editor vim
  - 默认情况下git会使用默认编辑器,如果想指定与系统不同的编辑器.
  - 就会在.git/config中增加如下内容.(当然,手动改的话也是可以的)
```
[core]
	editor = vim
```
* git config core.paper "less -N"
  - use "less" with number
```
man less

       -N or --LINE-NUMBERS
              Causes  a  line  number to be displayed at the beginning of each
              line in the display.
```
* git config color.diff true
  - set color for diff
* git config alias.XX YY_ZZ_UU
  - git XX == git YY_ZZ_UU
```
git config --global alias.ci commit
git config --global alias.cia 'commit --amend'
git config --global alias.rs reset
git config --global alias.rh 'reset --hard'
git config --global alias.rp 'reset HEAD^'
git config --global alias.st status
git config --global alias.br branch
git config --global alias.co checkout
git config --global alias.lo 'log --oneline'
git config --global alias.cl 'clean -df'
git config --global alias.pr 'pull --rebase'
git config --global alias.rbc 'rebase --continue'
git config --global alias.rb rebase
```
* git config core.autocrlf true
```
If core.autocrlf is set to true, that means that any time you add a file to the git repo that git thinks is a text file, it will turn all CRLF line endings to just LF before it stores it in the commit. Whenever you git checkout something, all text files automatically will have their LF line endings converted to CRLF endings. This allows development of a project across platforms that use different line-ending styles without commits being very noisy because each editor changes the line ending style as the line ending style is always consistently LF.

The side-effect of this convenient conversion, and this is what the warning you're seeing is about, is that if a text file you authored originally had LF endings instead of CRLF, it will be stored with LF as usual, but when checked out later it will have CRLF endings. For normal text files this is usually just fine. The warning is a "for your information" in this case, but in case git incorrectly assesses a binary file to be a text file, it is an important warning because git would then be corrupting your binary file.

If core.autocrlf is set to false, no line-ending conversion is ever performed, so text files are checked in as-is. This usually works ok, as long as all your developers are either on Linux or all on Windows. But in my experience I still tend to get text files with mixed line endings that end up causing problems.

My personal preference is to leave the setting turned ON, as a Windows developer.

See http://kernel.org/pub/software/scm/git/docs/git-config.html for updated info that includes the "input" value.
```

# 创建你的git repo #
* 使用git创建仓库主要有两种方法.
  * git init
  * git inti --bare
    - Usually we also set followings
```
git config receive.denyNonFastForwards true
git config core.sharedrepository 1
```
* git init:创建work tree的snapshot在服务器端,在分机git clone到本地后,git push会出现以下错误.
```
zjunwei@ubuntu:~/test/test2$ git push
Counting objects: 5, done.
Writing objects: 100% (3/3), 268 bytes, done.
Total 3 (delta 0), reused 0 (delta 0)
remote: error: refusing to update checked out branch: refs/heads/master
remote: error: By default, updating the current branch in a non-bare repository
remote: error: is denied, because it will make the index and work tree inconsistent
remote: error: with what you pushed, and will require 'git reset --hard' to match
remote: error: the work tree to HEAD.
remote: error:
remote: error: You can set 'receive.denyCurrentBranch' configuration variable to
remote: error: 'ignore' or 'warn' in the remote repository to allow pushing into
remote: error: its current branch; however, this is not recommended unless you
remote: error: arranged to update its work tree to match what you pushed in some
remote: error: other way.
remote: error:
remote: error: To squelch this message and still keep the default behaviour, set
remote: error: 'receive.denyCurrentBranch' configuration variable to 'refuse'.
To ssh://zjunwei-pc/~/test2
 ! [remote rejected] master -> master (branch is currently checked out)
 error: failed to push some refs to 'ssh://zjunwei-pc/~/test2'
```
  * 这是由于git默认拒绝了push操作,需要修改服务器仓库的.git/config添加如下代码：
```
[receive]
	denyCurrentBranch = ignore
```
  * 但是这样的话,服务器的git log会更新,而实际的代码没有更新.需要做如下操作:
```
$ git reset --hard
```
* git init --bare:创建空仓库,不含有work tree.可以在分机或服务器clone work tree.
  * 服务器:
```
$ mkdir gitrepo
$ cd gitrepo
$ git init --bare
```
  * 分机:
```
$ git clone ssh://$SERVERIP/$GITREPO_PATH/gitrepo
... Added new files ...
$ git add -A
$ git commit .
$ git push
junwei@ubuntu:~/test/test$ git push
No refs in common and none specified; doing nothing.
Perhaps you should specify a branch such as 'master'.
fatal: The remote end hung up unexpectedly
error: failed to push some refs to 'ssh://zjunwei-pc/~/test'
```
    * 这是由于push到一个空仓库所置,git不知道要提交哪个分支(我们指定为master).
    * 这时需要用如下命令(只有第一次需要,以后可以正常使用push):
```
$ git push origin master
```
    * 也许push的时候,会遇到以下error
```
zjunwei@ubuntu:~/gitroot/test$ git push origin master
zjunwei@zjunwei-pc's password:
Counting objects: 3, done.
Writing objects: 100% (3/3), 218 bytes, done.
Total 3 (delta 0), reused 0 (delta 0)
error: insufficient permission for adding an object to repository database ./objects

fatal: failed to write object
error: unpack failed: unpack-objects abnormal exit
To ssh://zjunwei-pc/gitroot/test
 ! [remote rejected] master -> master (n/a (unpacker error))
 error: failed to push some refs to 'ssh://zjunwei-pc/gitroot/test'
```
    * 说明其他用户没有相应的权限,尤其是git repo设置在了root path, such as "/gitroot"
```
# cd /gitroot/test.git
# chmod 777 -R *
```
# 创建开发branch #
* Create new branch for development.
```
$ git branch dev
$ git checkout dev
or
$ git checkout -b dev
```
* Develop in dev branch: git add, git commit
* If review it, get the "dev" branch to another PC "tmp" branch
```
In another PC the same repo:
$ git pull ssh://{YOUR_IP}/{PATH}/{repo} dev:tmp
It needs to know the passwd {YOUR_IP}
```
* After review, get the reviewed code from another PC "tmp" branch
```
In your PC the same repo:
$ git checkout -b tmp
$ git pull ssh://{REVIEWER_IP}/{PATH}/{repo} tmp:tmp
It needs to know the passwd {REVIEWER_IP}
```
* Pick the "dev" branch modification to master brach
```
$ git cherry-pick <commit-id>
commit-id is the "dev" branch commit it. modify the commit log as you wish.
```
  - If pick multi-commit:
```
from id1 to id2
$ git cherry-pick <commit-id-1>..<commit-id-2>
from dev HEAD~1 to HEAD
$ git cherry-pick dev~1 dev
```
  - If not commit for each commit.
```
$ git cherry-pick -n <commit-id>
```
  - check dev branch log
```
$ git log -b dev
```
* After development, you could remove the dev branch
```
$ git branch -d dev
```
# git patch operations #
* Create patches: git format-patch
```
$ git format-patch <commit-id> -o <dir> -n1
<commit-id>: 从哪个commit开始
-o:          输出到哪个文件
-n1:         创建几个patch,每个为一个commit
             也可以用origin,表示把所有本地未提交的commit都做成patch
```
* Apply patches:
  - git apply: 将patch的改动,apply到源代码上.
```
$ git apply 0001-XXX.patch
```
  - git am: 除了git apply相应的操作外,自动commit, 并自动填写commit log.
```
$ git am 0001-XXX.patch
```
  - Apply several patches.
    1. git am patches
      - 如果所有的patch都可以直接打上,那么任务完成,并且都已经提交.可以查看git log.
      - 会按顺序打patch
    1. If it rejects, you need to apply the patch manually.
```
$ git apply --reject 0001-XXX.patch
$ vi file.c file.c.rej <fix the rejects>
$ git add -A
```
    1. git am --resolved
      - am会自动将当前patch(0001-XXX.patch)打上,并继续打下一个patch/commit.
      - 如果又有patch出现reject,还需要再次手动fix the rejects.
      - 如果想跳过当前patch
```
$ git am --skip
```
      - 如果要退出当前am操作.
```
$ git am --abort
```
    1. 如果所有patch打完,可以查看am中是否还有patch in series
```
$ git am --continue
Resolve operation not in progress, we are not resuming.
```

# git operations for daily work #
1. git rebase
  * 假设有以下commit,你想要改commit a的内容.
```
  commit c
  commit b
  commit a <-- modify this commit
  commit 1 <-- git rebase here
```
  * 需要回退到commit 1(commit a前一个commit)
```
$ git rebase -i <commit id of commit 1>
```
  * 这时会显示如下内容,把pick-->edit即表示要编辑缉的commit.
```
pick  commit c
pick  commit b
pick  commit a  <-- modify "pick" as "edit"
```
  * 如果是想从重做以上commit(a)的内容.可以reset它的操作.
```
$ git reset HEAD^
```
  * 然后重新提交
```
$ git commit .
```
  * push 所有其它的commit(在commit a之后的)
```
$ git rebase --continue
```
1. git pull
  * set upstream by default
```
git pull <remote> <branch>
```
  * If you wish to set tracking information for this branch you can do so with:
```
git branch --set-upstream-to=<remote>/<branch> <local_branch>
```
1. fix merge when git pull
  * 有时候,当运行git pull的时候,会出现merge failure的信息.这时就需要我们手动来merge这些操作.通常,是关于patches/serial文件的.
  * 查看哪些文件需要更新或合并.
```
$ git status
```
  * merge manually.
  * 添加改动的文件.
```
$ git add -A
```
  * 提交改动的文件到本地(commit)
```
$ git commit
注意:不是"git commit ."
```
  * 提交merge及以前的改动到服务器.
```
$ git push
```
1. 查看文件更动的记录(latest信息)
```
$ git annotate <file_name>
```
1. 查看commit的tag信息, commit属于哪个kernel分支
```
$ git describe --contain
```
  * --contain 指下一个tag,即包含当前commit的下一个最近的tag
1. git checkout
  * 回到之前某个版本
```
$ git checkout <commit id>
```
  * 恢复到最新版本
```
$ git checkout <branch name>
```
1. git diff
  * git add之后,直接使用git diff看不到add中的内容.
  * 可以使用如下命令,查看add的内容.
```
$ git diff --cached
```
  * 如果,还有新的改动,没有git add,那么
```
git diff            --> 查看新的改动,并没有add
git diff --cached   --> 查看add的内容
```
1. git remote
  * nserv is the server name(ip<192.168.1.1> or domain name<nserv.cn>)
  * 查看远程仓库：
```
$ git remote -v
origin  git+ssh://nserv/export/git/linux-3.4.git (fetch)
origin  git+ssh://nserv/export/git/linux-3.4.git (push)
```
  * 添加远程仓库：
```
$ git remote add [name] [url]
$ git remote add new git+ssh://nserv/export/git/BRANCH_new.git
```
  * 删除远程仓库：
```
$ git remote rm [name]
$ git remote rm new
```
  * 取远程origin仓库的master分支到本地的new分支上
```
$ git pull origin master:new
           ^^^^^^
指的是git remote -v所列出来的远程仓库
```
  * 提交本地new分支到远程origin仓库的master分支上
```
$ git push origin new:master
```
  * 应用场景：将某个分支(含主分支)提交到远程仓库（以某分支创建远程仓库）。
```
将本地分支master提交到远程新建仓库
---- remote(nserv) ----
# cd /export/git/BRANCH_NEW
# git init --bare

---- local ----
$ git remote add new git+ssh://nserv/export/git/BRANCH_NEW.git
$ git remote -v
new     git+ssh://nserv/export/git/BRANCH_new.git (fetch)
new     git+ssh://nserv/export/git/BRANCH_new.git (push)
origin  git+ssh://nserv/export/git/linux-3.4.git (fetch)
origin  git+ssh://nserv/export/git/linux-3.4.git (push)

提交本地dev分支到远程new的master分支上
$ git push new dev:master

如果左边的分支名为空，则删除远程master分支
$ git push origin :master
```
  * Add new remote repo to your local repo
```
# git remote add origin ssh://path/to/remote/repo/linux
# git remote -v
git	ssh://other/linux (fetch)
git	ssh://other/linux (push)
origin	ssh://path/to/remote/repo/linux (fetch)
origin	ssh://path/to/remote/repo/linux (push)

# git pull origin
# git br -r <to find different branch in origin repo>
  origin/a-new
  origin/b-new

# git co -b <br name> origin/a-new
```
1. git tag
  * 为你的某个提交打上标签.无论它是哪个branch中(master, branch-1, branch-2, ..., branch-n. master也是一个branch嘛).
  * 例如,取下来linux-stable的代码,git tag时,你会看到很多版本,它们都是在各自的分支上.如果不是长期支持的版本分支,会被维护两个小版本号(如3.1-3.3).如果是长期支持的版本,要到指定年限才停止维护.
  * 创建tag
    * 轻量标签
```
git tag tag_name_light [commit-id]
如果带有commit-id,则给指定commit-id加上tag,否则给当前commit加上tag
```
    * 带附注标签
```
git tag -a tag_name -m "comments for this tag" [commit-id]
如果带有commit-id,则给指定commit-id加上tag,否则给当前commit加上tag
如果没有"-m",那么会跳出默认编辑器,让你添加comments
```
    * 这两种标签区别
```
轻量级,git show的时候,看不到tag name and messages
带附注的标签则可以

$ git show tag_light
commit 4023bfc9f351a7994fb6a7d515476c320f94a574
Author: Al Viro <viro@zeniv.linux.org.uk>
Date:   Sat Sep 13 21:59:43 2014 -0400
...(直接是commit内容)

$ git tag -a tag_comment -m "tag with comments"
$ git show tag_comment
tag with comments
Tagger: junwei-zhang <Jerry.Zhang@sony.com.cn>
Date:   Fri Sep 26 14:20:33 2014 +0900

tag_comment

commit 83373f702829dd9f6dcc56d275978d986fafee48
Merge: 9226b5b 4023bfc
Author: Linus Torvalds <torvalds@linux-foundation.org>
Date:   Sun Sep 14 17:37:36 2014 -0700
... (先是tag的信息,然后再显示commit内容)
```
  * 查看tag
    * 显示所有tag
```
git tag
```
    * 显示所有关心的tag
```
git tag -l "v3.4.*"
```
    * 显示某个tag
```
git show v3.4.104
```
  * 提交tag
```
git push --tags

git push不会提交标签到服务器
```
  * 删除tag
```
git tag -d tag_name
```
1. git log
  * 图形化git log
    * gitk
    * git log --graph
```
* 8a9794e [JetsonTK1] Refreshed defconfig refs #27317
*   cc8673f Merge branch 'master' of git+ssh://nserv/export/git/linux-3.10.y-BRANCH_SS
|\
| *   e4ef13e Merge branch 'master' of git+ssh://nserv.sm.sony.co.jp/export/git/linux-3.10.y-BRANCH_SS
| |\
| * | 70ef284 [defconfig] Updated defconfig for CONFIG_R8168=m refs #27563
* | | 9e6c9d1 [JetsonTK1] Add Add Generic PCI support for Jetson TK1 refs #27317
| |/
|/|
* | ef2f0f9 [IIO] Add ak8963 to panda ext dtsi file. refs #27331
* | 63413e1 [IIO] Applied iio-magn-ak8975-fix-unnecessary-casting-between-char.patch refs #27331
* | 72c96df [Patch Fix] Imported iio-magn-ak8975-fix-unnecessary-casting-between-char.patch refs #27331
* | e2b8de4 [IIO] Applied iio related patches refs #27331
* | 7faa0ca [Patch Fix] Imported iio related patches refs #27331
|/
* 6a9b22e [JetsonTK1] Refreshed defconfig refs #27270
```
  * git log `--format=<%T>`
    * 格式化log输出
    * git log `--format=%H`
      * 只显示commit id，即commit hash
  * git log `<from>..<to>`
    * `<from>` and `<to>` 可以是branch名，tag名，commit id.
    * git log `v3.10.24..v3.10.25`
      * show the log from `v3.10.24` to `v3.10.25`
    * git log `v3.10.24`
      * show the log from `v3.10.24` to `HEAD`
  * git log `--grep`
    * git log `--grep=<key>`
      * 在commit log中grep关键字，区分大小写
    * git log `-i --grep=<key>`
      * 在commit log中grep关键字，不区分大小写
    * git log `--grep=<key1>` `--grep=<key2>`
      * 两个关键词为“或”的关系
    * git log `--grep=<key1>` `--grep=<key2>` `--all-match`
      * 两个关键词为“与”的关系
  * git log `--stat`
    * 显示文件修改状态
```
commit 1e4503166c217ad5a170fcdb000a4ba3a29448bb
Author: junwei-zhang <Jerry.Zhang@...>
Date:   Mon Dec 8 08:24:39 2014 +0000

    [JetsonTK1] Updated tegra12_defconfig for GPU support
    refs #xxxxx

 arch/arm/configs/tegra12_defconfig                           | 18 +++++++++++++++++-
 patches/sony/refboard/jetsontk1/jetsontk1-tegra-config.patch | 35 ++++++++++++++++++++++++++++-------
 2 files changed, 45 insertions(+), 8 deletions(-)
```

# git bisect 二分查找bad commit #
* 开始使用git git bisect后，只有master分支。
* 选定good和bad的commit id，git会自动跳到二分查找的commit id上。
* 当你验证后，再对当前commit进行good或bad的评价。
* 最终你会找到出现问题的commit id。
```
junwei-zhang@u1404:~/gitroot/linux-3.10.y-BRANCH_SS$ git bisect start
junwei-zhang@u1404:~/gitroot/linux-3.10.y-BRANCH_SS$ git br
* master
junwei-zhang@u1404:~/gitroot/linux-3.10.y-BRANCH_SS$ git bisect good ea8ad3ed6b412ec041ee728cc6e8e77003ff0b21
junwei-zhang@u1404:~/gitroot/linux-3.10.y-BRANCH_SS$ git bisect bad 575a9193d27ff4ec42151c36923481a4694b72d1
Bisecting: 90 revisions left to test after this (roughly 7 steps)
[f665d86400e12e887569080c32a978f947ea1fda] [iio] ABC
junwei-zhang@u1404:~/gitroot/linux-3.10.y-BRANCH_SS$ git log
commit f665d86400e12e887569080c32a978f947ea1fda
Author: user <user@mail.com>
Date:   Tue Mar 3 13:51:25 2015 +0800

    [iio] ABC

commit b78c72b0024b6eefd36290c98d40739f7ca7c191
Author: user <user@mail.com>
Date:   Tue Mar 3 11:46:27 2015 +0800

    [iio] DEF
junwei-zhang@u1404:~/gitroot/linux-3.10.y-BRANCH_SS$ git bisect good 
Bisecting: 45 revisions left to test after this (roughly 6 steps)
[764cabb693dcc67cdcd83f3fa8e4005484420cc9] Merge branch 'master' of BCD
junwei-zhang@u1404:~/gitroot/linux-3.10.y-BRANCH_SS$ git bisect bad 
Bisecting: 22 revisions left to test after this (roughly 5 steps)
[282a58906e51ad430befb99799e9b49e6aabf261] [Patch Fix] Renamed A
junwei-zhang@u1404:~/gitroot/linux-3.10.y-BRANCH_SS$ git bisect bad 
Bisecting: 10 revisions left to test after this (roughly 4 steps)
[245fb92ce99164bec82adffb240570b827b65bf2] [Patch Fix] Applied B
junwei-zhang@u1404:~/gitroot/linux-3.10.y-BRANCH_SS$ git bisect good 
Bisecting: 5 revisions left to test after this (roughly 3 steps)
[e87178b8b1e19d97c4e76ecaab108498e8376964] [Spritzer] Implement C
junwei-zhang@u1404:~/gitroot/linux-3.10.y-BRANCH_SS$ git bisect good 
Bisecting: 2 revisions left to test after this (roughly 2 steps)
[084b1fc7a1bfa5c479476f1017411828e44817f2] [Patch Fix] fixed D
junwei-zhang@u1404:~/gitroot/linux-3.10.y-BRANCH_SS$ git bisect bad 
Bisecting: 0 revisions left to test after this (roughly 1 step)
[d668eb0a5ab415ff487816fdce978ff7b2a584df] [Patch Fix] Applied E
junwei-zhang@u1404:~/gitroot/linux-3.10.y-BRANCH_SS$ git bisect bad
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[e66804f035c5bf79bec6d5ab6d80e837a114888e] [Spritzer] Add F
junwei-zhang@u1404:~/gitroot/linux-3.10.y-BRANCH_SS$ git bisect good
d668eb0a5ab415ff487816fdce978ff7b2a584df is the first bad commit
commit d668eb0a5ab415ff487816fdce978ff7b2a584df
Author: userX <userX@mail.com>
Date:   Sun Mar 15 12:46:24 2015 +0530

    [Patch Fix] Applied G

:040000 040000 b1f96b64b4e7e3a76c13663567d08ed73edf8131 8531a31f4fad337b4631e4f8b66040abc8ce2d99 M	.pc
:040000 040000 1b8cf6434ed686232ac5a68c3d34d79853cce45b faa260fb320074e9d057f437093e0f6045674b0d M	Documentation
:100644 100644 a0c19819975baeffddee43b25f596d0533ed9df3 14e14fabaf163be15bc666babc889deadc625206 M	MAINTAINERS
:040000 040000 081a3376b6968992fa2dc8e66f46930c79cd14be f2b905ca9b2f9dc14a3033b4b1f0eadbef9dd5b5 M	arch
:040000 040000 709e120a5b2105d740674cd47819c4c65976fbe2 e173fcc0731de2d5a4c2c47833034299471a03bd M	drivers
:040000 040000 be5b73cadfb25147f94a05633c486032c6c54a9a 6d41fff4739d33e081d6359e063bbb9e25932e8a M	include
:040000 040000 758c244102807cc89467a9b6721b94a14d9abbcb 25f9b48983c09866bfacf50c285b71452cd08f3e M	lib
:040000 040000 150c59ec2fa167ea740158de30fd957323fdb598 6ad09c686504711bce3fbe36b4400dff2032c1e2 M	patches
junwei-zhang@u1404:~/gitroot/linux-3.10.y-BRANCH_SS$ git bisect log
git bisect start
# good: [ea8ad3ed6b412ec041ee728cc6e8e77003ff0b21] [script] good
git bisect good ea8ad3ed6b412ec041ee728cc6e8e77003ff0b21
# bad: [575a9193d27ff4ec42151c36923481a4694b72d1] [BDKTE] bad
git bisect bad 575a9193d27ff4ec42151c36923481a4694b72d1
# good: [f665d86400e12e887569080c32a978f947ea1fda] [iio] ABC
git bisect good f665d86400e12e887569080c32a978f947ea1fda
# bad: [764cabb693dcc67cdcd83f3fa8e4005484420cc9] Merge branch 'master' of BCD
git bisect bad 764cabb693dcc67cdcd83f3fa8e4005484420cc9
# bad: [282a58906e51ad430befb99799e9b49e6aabf261] [Patch Fix] Renamed A
git bisect bad 282a58906e51ad430befb99799e9b49e6aabf261
# good: [245fb92ce99164bec82adffb240570b827b65bf2] [Patch Fix] Applied B
git bisect good 245fb92ce99164bec82adffb240570b827b65bf2
# good: [e87178b8b1e19d97c4e76ecaab108498e8376964] [Spritzer] Implement C
git bisect good e87178b8b1e19d97c4e76ecaab108498e8376964
# bad: [084b1fc7a1bfa5c479476f1017411828e44817f2] [Patch Fix] fixed D
git bisect bad 084b1fc7a1bfa5c479476f1017411828e44817f2
# bad: [d668eb0a5ab415ff487816fdce978ff7b2a584df] [Patch Fix] Applied E
git bisect bad d668eb0a5ab415ff487816fdce978ff7b2a584df
# good: [e66804f035c5bf79bec6d5ab6d80e837a114888e] [Spritzer] Add F
git bisect good e66804f035c5bf79bec6d5ab6d80e837a114888e
# first bad commit: [d668eb0a5ab415ff487816fdce978ff7b2a584df] [Patch Fix] Applied G
junwei-zhang@u1404:~/gitroot/linux-3.10.y-BRANCH_SS$ git bisect
usage: git bisect [help|start|bad|good|skip|next|reset|visualize|replay|log|run]
junwei-zhang@u1404:~/gitroot/linux-3.10.y-BRANCH_SS$ git bisect reset
Previous HEAD position was e66804f... [Spritzer] Add the device tree node for the driver of the HIFC in spritzer refs #28237
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
junwei-zhang@u1404:~/gitroot/linux-3.10.y-BRANCH_SS$

```

# Reference #
* [Git 用户手册](http://blog.csdn.net/zzulp/article/details/6238527)
* [Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

