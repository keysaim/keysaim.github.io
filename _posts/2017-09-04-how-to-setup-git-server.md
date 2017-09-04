---
layout:     post
title:      "如何搭建本地的Git服务器"
subtitle:   ""
date:       2017-09-04
author:     "keysaim"
header-img: ""
catalog: true
tags:
    - git
    - 教程
---

# 概述

本文将介绍如何在本地搭建Git服务器。我们知道Git其实是个分布式的版本管理系统，与中心化的版本管理系统如SVN有根本的不同，每个使用者都可以在本地存储一份独立的备份，每个Git的使用者并不会因为没有中心服务器而不能工作（如果是SVN之类的，如果服务器挂了是不能够提交改动的）。然而，在进行团队开发的时候，有时候还是非常需要有一个统一的地方管理唯一的一份完整的代码，这样可以非常方便的进行团队协作开发。这里讲述一种极为简单的搭建本地Git服务器的方法。

# 如何搭建

## 准备一台Linux服务器

这里不考虑windows系统，所以请务必准备一台Linux系统，分发版本没有关系，这里假定使用的是`Centos 7`。

## 配置Git服务器

* 添加用户

    ```console
    $ sudo adduser git
    $ sudo passwd git
    Changing password for user git.
    New password:
    Retype new password:
    passwd: all authentication tokens updated successfully.
    ```

* 配置好ssh

    ```console
    $ sudo su git
    $ cd
    $ mkdir .ssh && chmod 700 .ssh
    $ touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
    ```

## 初始化项目

假定我们有一个叫`test`的项目需要管理，那么首先我们需要在Git服务器上面创建并初始化该项目。

```console
$ sudo su git
$ cd
$ mkdir test
$ cd test
$ git init --bare
Initialized empty Git repository in /home/git/test
```

注意，其中`git init --bare`就是用来初始化Git项目的，`--bare`参数表示只存储Git的管理文件而不展现`test`项目本身的文件。查看下初始化之后的目录：

```console
$ ls
branches  config  description  HEAD  hooks  info  objects  refs
```

可见里面只有Git管理有关的文件。

## 上传`test`项目

假定Git服务器IP为`10.10.10.10`，回到你的工作机器，打开`test`项目目录，上传项目：

```console
$ cd test
$ git remote add origin git@10.10.10.10:/home/git/mlc
$ git remote -v
origin  git@10.10.10.10:/home/git/mlc (fetch)
origin  git@10.10.10.10:/home/git/mlc (push)
```

如果发现`git remote add origin`执行失败：

```console
$ git remote add origin git@10.10.10.10:/home/git/mlc
fatal: remote origin already exists.
```

说明曾经已经设置过`origin`地址了，可以更改如下：

```console
$ git remote remove origin
$ git remote -v
$ git remote add origin git@10.10.10.10:/home/git/mlc
$ git remote -v
origin  git@10.10.10.10:/home/git/mlc (fetch)
origin  git@10.10.10.10:/home/git/mlc (push)
```

设置好之后，可以上传`test`项目了：

```console
$ git push origin master
git@10.10.10.10's password:
Counting objects: 4, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 531 bytes | 0 bytes/s, done.
Total 4 (delta 0), reused 0 (delta 0)
To 10.10.10.10:/home/git/mlc
 * [new branch]      master -> master
```

可见，Git服务器已经可以正常工作了。

# 结语

本文只是简单的介绍原生的Git服务器的搭建，实际上，在现实工作中很多情况下都会有专门的基于Git的管理系统，使用起来非常方便。这里面非开源的最有名当然是`Github`了，开源的里面目前最好的应该是`Gitlab`了，参照官方文档，你可以非常方便的搭建自己的Git服务器了。

# 参考

* [Git on the Server - Setting Up the Server](https://git-scm.com/book/en/v2/Git-on-the-Server-Setting-Up-the-Server)
