---
layout: post
title:  Git One Minute Troubleshooting
date:   2020-01-29 21:47:00 +0800
categories: git
---

这篇文章最早写于2016年1月31日，后续有多次修改。

2014年时，网易内部推广使用git作为代码管理工具。本人日常使用的比较多，喜欢读*man*，积累了两年多的后便写下了该文章。

后续也会持续更新一些 Git 快速排错经验到这上面。

修订日期：

*   2020年01月29日

---

## 0、前言

本文是一份 Git 快速排错查考，供大家在项目中使用。这些问题都是在实际项目经常遇到的，处理起来十分费时。作者希望提供这样一份查考，能方便大家解决一些 Git 使用上棘手的问题，节省宝贵的时间。本文不是一篇 Git 教程，如果需要教程请自行查阅资料（*RTFM*）。

## 1、网络问题

### 问题：

    $ git clone GIT_YOUR_REPOSITORY_URL

    Cloning into 'Troubleshoot'...

    ssh: connect to host github.com port 22: Bad file number

    fatal: Could not read from remote repository.

    Please make sure you have the correct access rights

    and the repository exists.

### 分析：

* 网络限制。

* 断网，连接代理。

### 解决：

* 网络限制

找相关 SA 处理。

* 断网，连接代理

自行检查网络解决。

## 2、无法访问仓库

### 问题：

    $ git clone GIT_YOUR_REPOSITORY_URL

    Cloning into 'Troubleshoot'...

    Permission denied (publickey).

    fatal: Could not read from remote repository.

    Please make sure you have the correct access rights

    and the repository exists.

### 分析：

需要添加公钥到 git 仓库中。

### 解决：

step1 产生ssh key:

    ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

step2:

    Enter a file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]

可以输入新的ssh key文件名，如`.ssh/id_rsa_other` ，直接回车则使用默认的ssh key。

step3:

    Enter passphrase (empty for no passphrase): [Type a passphrase]

    Enter same passphrase again: [Type passphrase again]

step4:

拷贝公钥。

Windows & linux:

    clip < ~/.ssh/id_rsa.pub

Mac OX:

    pbcopy < ~/.ssh/id_rsa.pub

step5:

    ssh-add ~/.ssh/id_rsa

使用`~/.ssh/id_rsa`作为默认 ssh key 一般不需要这步操作。

如果提示：

    ssh-add ~/.ssh/id_rsa

    Could not open a connection to your authentication agent.

需要先执行：

    $ eval "$(ssh-agent -s)"

    Agent pid 6508

最后：

    ssh-add ~/.ssh/id_rsa

## 3、提交作者显示 `unknown`

### 问题：

    * db7333b unknown  XXX Sat Nov 21 15:12:10 2015 +0800

    * ce10ac1 unknown  XXX Fri Nov 20 17:04:05 2015 +0800

    * f7de4e6 unknown  XXX Thu Nov 19 14:29:12 2015 +0800

    * b3b5ec1 unknown  XXX Wed Nov 18 10:19:52 2015 +0800

    * 2b38f20 unknown  XXX Mon Nov 16 17:42:35 2015 +0800

### 分析:

由于 git 没有设置用户名或者 email。

### 解决：

    $ git config --global user.name "Your Name"

    $ git config --global user.email your_email@example.com

## 4、移除 submodule 依赖

### 问题：

移除项目中依赖 submodule 后，又重新添加一次，经常会出现：

    $ git submodule add GIT_YOUR_REPOSITORY_URL

    A git directory for 'Submodule' is found locally with remote(s):

      origin        GIT_YOUR_REPOSITORY_URL

    If you want to reuse this local git directory instead of cloning again from

      GIT_YOUR_REPOSITORY_URL

    use the '--force' option. If the local git directory is not the correct repo

    or you are unsure what this means choose another name with the '--name' option.

### 分析：

由于删除不正确删除 submodule，导致本地残留 submodule 信息。

### 解决：

根据命令行提示使用`--force`作为`git submodule add`的参数。

但是如果本地submodule有修改，情况会比较复杂，这里就不展开讨论。

这里提供正确的删除 submodule 的方式：

    $ git submodule deinit your_submodule

    $ git rm your_submodule

    $ git commit

上述方式避免了各种删除submodule不正确导致的诡异问题。

## 5、删除 submodule 目录

### 问题：

更新代码之后，发现：

    $ git st

    On branch master

    Your branch is up-to-date with 'origin/master'.

    Untracked files:

      (use "git add <file>..." to include in what will be committed)

            Submodule/

尝试`git rm`该目录，会出现：

    $ git rm Submodule/

    fatal: pathspec 'Submodule/' did not match any files

尝试`rm -rf`该目录，会出现：

其实什么都不会出现，只是简单地删除了目录，submodule的数据依旧残留。

如果你再次添加了相同仓库地址的 submodule ，将会出现：

    $ git submodule add GIT_YOUR_REPOSITORY_URL

    A git directory for 'Submodule' is found locally with remote(s):

      origin        GIT_YOUR_REPOSITORY_URL

    If you want to reuse this local git directory instead of cloning again from

      GIT_YOUR_REPOSITORY_URL

    use the '--force' option. If the local git directory is not the correct repo

    or you are unsure what this means choose another name with the '--name' option.

### 分析：

由于删除不正确删除 submodule，导致本地残留 submodule 信息。

### 解决：

使用`git clean`清理。

先检查是否有其他未提交的内容，先提交。然后运行：

    $ git clean -nxdff

    Would remove Submodule/

再运行，使用交互式清理（避免误删其他文件）：

    $ git clean -ixdff

    Would remove the following item:

      Submodule/

    *** Commands ***

        1: clean                2: filter by pattern    3: select by numbers

        4: ask each             5: quit                 6: help

    What now> [Press 4]

    remove Submodule/? [Press y]

    Removing Submodule/

如果你不介意工程需要重新编译（注：当前git目录下的 Untracked files 都会被删除），可以直接运行：

    $ git clean -xdff

## 6、找回被删除的提交或分支

### 问题：

有时候因为特殊原因需要找回一个旧的分支，但是这个分支已经被远程服务器删除了，本地分支也被清理了。

如果最近`fetch`过远程仓库且没有`prune`或者`checkout`过本删除的分支，恭喜你，你可以找回你的分支。

### 分析：

* 曾经`fetch`，可以利用本地跟踪分支找回分支。

* 曾经`checkout`，可以利用 revision 记录，找回当初被删除的节点，然后`checkout`到节点找回分支或提交。

### 解决：

* 对于`fetch`，列出本地所有分支，包括跟踪分支:

    $ git branch --all

找到你希望找回的分支，分支名称一般是`origin/your_deleted_branch`，接着运行：

    $ git checkout -b your_new_branch_name origin/your_deleted_branch

被删除的分支被找回，现在你可以对 your_new_branch_name 进行操作。

* 对于`checkout`，运行：

    $ git reflog

浏览记录，找到当时checkout到your_deleted_branch的最近一次记录，然后把SHA值记下来，运行

    $ git checkout -b your_new_branch_name 2b38f20

被删除的分支被找回，现在你可以对 your_new_branch_name 进行操作。

## 7、强推代码，导致其他人本地提交丢失

### 问题：

由于非必要的`push --force`，导致其他人，本地未提交的代码“丢失”（`git log`无法看到）。

### 分析：

由于`push --force`导致了远程分支和本地分支的祖先不一致。

### 解决：

step1:

找到丢失的提交

git log --all

step2:

创建新的分支

    git checkout -b recover your_lost_commit

step3:

    $ git rebase origin/your_branch

step4:

    $ git push -u origin master

将本地的提交在远程各种分支上重做，本质的思路是将本地提交重回主线。

## 8、管理第三方库的更新

### 问题：

对于使用 submodule 依赖第三方库的方式，经常会遇到需要修改第三方库代码的需求，这就需要将第三方库完整拷贝出来。

并推送到项目的仓库中去，并且需要持续更新第三方库。

### 分析：

完整拷贝原始仓库、推送到私有仓库、持续更新。

### 解决：

* 完整拷贝原始仓库

    git clone --mirror GIT_YOUR_REPOSITORY_URL

* 推送到私有仓库

step1:

添加私有仓库的地址：

    git remote add --tags my_origin GIT_YOUR_REPOSITORY_URL

step2:

完整推送到私有分支：

    git push --mirror my_origin

* 持续更新

step1:

更新原始仓库：

    git remote update --prune

step2:

将更新完整推送到私有分支：

    git push --mirror my_origin

## 9、删除多余的tags

### 问题：

对于迭代很快的项目来说，tags积累到一定程度，需要清理。

### 分析：

删除tags，有两种情况

* 本地删除tags，同步到服务器

* 从服务器更新到本地，同步删除本地tags

### 解决：

* 本地删除tags，同步到服务器：

        git tag -d your_tag

        git push origin :refs/tags/your_tag  or

        git push origin :your_tag

* 从服务器同步，删除本地：

        git fetch --prune origin +refs/tags/*:refs/tags/*

## 10、分离项目中的模块，作为新的仓库

### 问题：

项目发展到一定程度，某些稳定的公共模块，可以单独分离出来，提供给更多项目使用。

### 分析：

分离的模块需要在同一个目录下。

### 解决：

step1:

准备一个完全包含原始工程内容的*新的*仓库。

step2:

    git clone GIT_YOUR_REPOSITORY_URL

step3:

    git filter-branch --prune-empty --subdirectory-filter your_folder master

step4:

    git push

## 11、查找内容修改的提交项

### 问题：

如何快速找到一段内容何时被修改，通过提交项定位问题。

### 分析：

利用`git log`。

### 解决：

    $ git log -S"content you find" -- your file path

这样相关修改的提交项就会列出来，方便定位问题。
