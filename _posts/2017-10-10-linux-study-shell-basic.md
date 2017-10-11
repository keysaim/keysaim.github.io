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
* Sh (Bourne Shell)
* Bash (Bourne Again Shell)
* Csh (C Shell)
* Ksh (Korn Shell)

一般来说，`Bash`应该是使用最多的`Shell`程序了，本文也主要基于`Bash`来展开。

# Shell展开（Shell Expansion）

`Shell`程序是一个命令解释器，因此在终端输入命令之后，`Shell`将扫描命令并做适当的修改，这个过程称为Shell展开。Shell展开是Shell解释执行之前极为重要的一步，了解它将有利于你对Shell命令或者脚本的理解，本章节将逐步带大家来了解这个过程。

## 命令参数解析

这里的空格包括了制表符（Tab）。当Shell程序扫描输入的命令时，会以*连续*的空格为界，将命令切分成一组参数，因此你输入多个空格为界跟输入一个空格的效果是一样的。通常来讲，第一个参数就是要执行的命令，而后面的参数则是改命令的参数。一下几个命令其实是等效的：

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

当然，有时候你需要在一个参数中包括空格，这样的话你就需要将这个参数以引号引起来，引号包括了单引号`'`跟双引号`"`，两者都可以。`shell`会将引号中的字符串视为一个参数，不论里面有没有空格。当然，特别指出的是，不要用反引号`\``，反引号将在后面详细讲述。

如命令`echo 'Hello World!'`在`shell`解析之后会有两个参数，分别为`echo`跟`Hello World!`。而如果不用引号`echo Hello World!`，则将解析为三个参数。

> 特别提一下，对于`echo`命令，如果需要输出需要转义的字符，如回车等，则需要执行`echo -e "Hello World!\n"`，如果不加`-e`，则`\n`会被直接显示出来。
>
>    ```terminal
>    # echo "hello\n"
>    hello\n
>    # echo -e "hello\n"
>    hello
>
>    ```

## 命令

### 内部命令

对于`shell`来说，命令有内部命令（Builtin Commands）跟外部命令（External Commands）之分，所谓内部命令指的是包含在`shell`解析器中的命令。内部命令一般有[4种类型](http://www.gnu.org/software/bash/manual/bashref.html#Shell-Builtin-Commands)：

* `sh`内部命令

    这些内部命令来源于`Bourne Shell`，通常包括了以下命令：
    `: . break cd continue eval exec exit export getopts hash pwd readonly return shift test/[ times trap umask unset`。

* `bash`内部命令

    这些内部命令来源于`Bourne Again Shell`，通常包括了以下命令：
    `alias bind builtin caller command declare echo enable help let local logout mapfile printf read readarray source type typeset ulimit unalias`。

* 修改`shell`行为的内部命令

    这些内部命令用来修改`shell`的默认行为。包括了`set shopt`命令。

* 特殊内部命令

    由于历史原因，POSIX标注将一些内部命令划分为特殊内部命令，特殊的之处在于这些命令的查找时间以及命令运行后的状态等方面，只有当Bash以[POSIX模式](http://www.gnu.org/software/bash/manual/bashref.html#Bash-POSIX-Mode)运行时，这些命令才是特殊命令，否则它们跟其它内部命令没啥区别。特殊内部命令包括了`break : . continue eval exec exit export readonly return set shift trap unset`。

通常来说，内部命令可能会被提前至于内存中，因此运行起来会比外部命令要快。对于外部命令，可以认为除了内部命令之后就可以认为是外部命令了，通常来讲，`/bin`跟`/sbin`下的都是外部命令，当然，应用有关的通常也是外部命令。

我们可以通过`type`命令来查看一个命令是否是内部命令：

```terminal
# type cd
cd is a shell builtin
# type awk
awk is /usr/bin/awk
```

另外，对于很多内部命令，它们可能对应的会有外部命令版本，可以通过`type`命令来查看：

```terminal
# type -a echo
echo is a shell builtin
echo is /usr/bin/echo
# type -a cd
cd is a shell builtin
cd is /usr/bin/cd
```

反过来，我们一般可以通过命令`which`来查询一个命令是否是外部命令：

```terminal
# which awk
/usr/bin/awk
# which .
/usr/bin/which: no . in (/opt/rh/rh-python34/root/usr/bin:/usr/java/default/bin/:/usr/local/git/bin:/opt/ActiveTcl-8.5/bin:/root/perl5/bin:/root/env/maven/apache-maven-3.3.3/bin:/root/soft/wrk/wrk-4.0.1:/root/usr/go/bin:/usr/local/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin)
```

不过得注意，很多内部命令都有外部版本，因此通过`which`查询出来的是其外部命令版本：

```terminal
# which echo
/usr/bin/echo
# type echo
echo is a shell builtin
```

对于内部命令的详细说明，可以查看[GNU的文档](http://www.gnu.org/software/bash/manual/bashref.html#Shell-Builtin-Commands)。

