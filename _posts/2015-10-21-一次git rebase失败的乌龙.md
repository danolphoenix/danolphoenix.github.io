---
layout: post
title: 一次git rebase失败的乌龙
tags: git rebase 
categories: git
---

代码有点旧了，开发完成后（其实就改了一行包的版本号），发现库里的代码已经前进了好多个commit
于是先git fetch，将服务器上的远端分支取回来到本地，假设存放在本地的服务器上的分支拷贝为origin/master
以下是我在网上抄的例子，具体现实中的输出已经被secureCRT给吃掉了。。但是出错请形是一样的

~~~bash
$ git rebase origin/master
First, rewinding head to replay your work on top of it…
Applying: Much faster default template engine. Fixes #418.
Using index info to reconstruct a base tree…
M src/templates/default.js#当然我出错的不是这个
<stdin>:31: trailing whitespace.
<stdin>:34: trailing whitespace.
warning: 2 lines add whitespace errors.

Falling back to patching base and 3-way merge…
Auto-merging src/templates/default.js
CONFLICT (content): Merge conflict in src/templates/default.js
Failed to merge in the changes.
Patch failed at 0001 Much faster default template engine. Fixes #418.
The copy of the patch that failed is found in:
  /vagrant/json-editor/.git/rebase-apply/patch

When you have resolved this problem, run “git rebase –continue”.
If you prefer to skip this patch, run “git rebase –skip” instead.
To check out the original branch and stop rebasing, run “git rebase –abort”.
~~~

大致意思是在rebase的过程中，冲突了，修改好冲突，然后执行git add ,git rebase --continue

~~~bash
$ git add src/templates/default.js
$ git rebase –continue
Applying: Much faster default template engine. Fixes #418.


No changes - did you forget to use ‘git add’?
If there is nothing left to stage, chances are that something else
already introduced the same changes; you might want to skip this patch.

When you have resolved this problem, run “git rebase –continue”.
If you prefer to skip this patch, run “git rebase –skip” instead.
To check out the original branch and stop rebasing, run “git rebase –abort”.

$ git rebase –continue
Applying: Much faster default template engine. Fixes #418.
No changes - did you forget to use 'git add’?
If there is nothing left to stage, chances are that something else
already introduced the same changes; you might want to skip this patch.

When you have resolved this problem, run “git rebase –continue”.
If you prefer to skip this patch, run “git rebase –skip” instead.
To check out the original branch and stop rebasing, run “git rebase –abort”.

$ git status
rebase in progress; onto cfd3419
You are currently rebasing branch 'terria’ on 'cfd3419’.
 (all conflicts fixed: run “git rebase –continue”)

nothing to commit, working directory clean

No changes - did you forget to use ‘git add’?
~~~

是说没有修改？？我明明修改了的啊。。。为什么说No changes。。难道是没有检测到？
又换了N个方法颠来倒去地整了好一会儿，没辙。。其过程中并没有仔细看我修改的那一行涉及的地方。。。

最后到代码库上看了一下最新的提交的人都提交了些啥，发现L哥已经把我准备提交的修改的那一行包的版本号进行了修改，给做了个commit他自己提交了。。
这也跟我说一声儿啊。。。
看来git是对的，确实没有修改。。。
把commit给abandon掉，直接到环境里看结果，反正包已经打上了