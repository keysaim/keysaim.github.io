---
layout:     post
title:      "Linux系统学习（三）shell基础"
subtitle:   ""
date:       2017-10-10
author:     "keysaim"
header-img: ""
catalog: true
tags:
    - Linux
    - study
    - 学习
---

# 概述

首先，咱们来了解一下，什么是`Shell`。操作系统内核给我们提供了各种接口，同时也提供了各种用户层的库，理论上我们基于这些可以编写程序实现各种我们想要的功能，不过问题是，咱们不可能做什么事情都要重新编写程序，这样使用起来也太困难了。因此，操作系统（包括Linux）通常都会引入一个`Shell`这样的特殊程序，这个程序会接受输入的命令然后执行，并可能将执行结果呈现出来。总结来说，`Shell`是一个从输入设备或者文件读取命令，并且解释、执行器的用户态程序。

在Linux系统中，通常使用的`Shell`程序包括有：
* Bash (Bourne Again Shell)
* Sh (The original Bourne Shell)
* Csh (C Shell)
* Ksh (Korn Shell)

一般来说，`Bash`应该是使用最多的`Shell`程序了，本文也主要基于`Bash`来展开。

# Shell展开（Shell Expansion）

`Shell`是一个命令解释器，因此在终端输入命令之后，`Shell`是需要扫描命令并做适当的修改，这个过程称为Shell展开。Shell展开是Shell解释执行之前极为重要的一步，了解它将有利于你对Shell命令或者脚本的理解，本章节将逐步带大家来了解这个过程。

## 空格的移除

这里的空格包括了制表符（Tab）。当Shell程序扫描输入的命令时，会以连续的空格为界，将命令切分成一组参数，因此你输入多个空格为界跟输入一个空格的效果是一样的。通常来讲，第一个参数就是要执行的命令，而后面的参数则是改命令的参数。一下几个命令其实是等效的：

```terminal
# echo Hello World
Hello World
# echo Hello
Hello World
# echo   Hello
Hello World
#    echo
Hello World
```

## 引号

当然，有时候你需要


