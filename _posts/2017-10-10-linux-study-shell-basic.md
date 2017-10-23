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

## 别名

可以用`alias`命令给一个命令取一个别名：

```terminal
# alias print=echo
# print "hello"
hello
# type print
print is aliased to `echo'
```

别名一个常用的用法是用来缩写已知的命令：

```terminal
# type ls
ls is aliased to `ls --color=auto'
```

可见`ls`命令实际上是命令`ls --color=auto`的别名，这样就相当于改变了`ls`命令的默认行为了。

前面咱们通过`type`命令来查看命令的别名，实际上更加推荐采用`alias`或者`which`来查看：

```terminal
# alias ls
alias ls='ls --color=auto'
# which ls
alias ls='ls --color=auto'
    /usr/bin/ls
```

如果要取消别名，则可以采用`unalias`命令:

```terminal
# which ls
alias ls='ls --color=auto'
    /usr/bin/ls
# unalias ls
# which ls
/usr/bin/ls
```

## 显示shell展开的结果

由于`shell`展开的存在，你输入的命令被展开之后可能会发生变化，如果需要知道`shell`展开之后的命令，可以使用内部命令`set`来修改`shell`的默认参数来显示：

```terminal
# set -x
++ printf '\033]0;%s@%s:%s\007' root traffic-base1 '~'
# echo hello         world
+ echo hello world
hello world
++ printf '\033]0;%s@%s:%s\007' root traffic-base1 '~'
```

其中，以`+`开头的就是展开之后的命令，可见展开之后，`shell`将多余的空格去掉了。如果不要再显示了，可以输入命令`set +x`。

# shell控制操作符 (Control Operators）

## 分号`;`

`shell`命令输入时，你可以将多个命令输入在一行，只要在不同命令之间以分号`;`隔开，当然分号不能是在引号中。

## `&`符号

通常情况下，`shell`会在前台执行命令，并等待命令结束才返回。如果需要将命令放到后台去执行，可以使用`&`符号放在命令最后面，这样的话命令会被放在后台执行，`shell`会立刻返回而不用等待命令结束。这里注意的是，即便放在后台执行，但是如果不处理好命令的输入，则命令的输出可能会继续在当前的终端输出，后面会讲述如何处理命令的输出。

## `$?`操作符

每个命令执行完后都会有个退出码（Exit Code），其值为0时表示命令成功，否则命令失败。这个退出码可以通过`$?`来访问，执行完命令后立马访问`$?`可以获取该命令的退出码，并以此来判断命令是否成功。每个命令的执行都会产生新的退出码，所以请务必在命令执行完，立刻访问`$?`来获取退出码。

初看起来，`$?`似乎是一个`shell`变量，但实际上并非如此，因为你无法对`$?`赋值。`$?`准确来说是`shell`的一个参数。

## `&&`操作符

此操作符表示逻辑与，你可以将两个命令用此操作符连接起来，如`cmd1 && cmd2`，只有当`cmd1`执行成功之后，`cmd2`才会被执行。这里的成功指的是`cmd1`的退出码是0。

```terminal
# hello && echo world
-bash: hello: command not found
# echo hello && echo world
hello
world
```

当然，`&&`也可以将多个命令连接起来，其执行类似，只有当前面的命令成功，后面的才会执行。因此，将多个命令写在一行用`&&`可以实现，只不过`&&`必须按照逻辑与的关系执行，而`;`号的话会执行所有的命令。

## `||`操作符

很显然，与`&&`相对，`||`操作符表示逻辑或的关系，同样可以连接两个命令，如`cmd1 || cmd2`，只有当`cmd1`失败了，才会执行`cmd2`，这里的失败指的是`cmd1`的退出码非0。

## `&&`与`||`混合

这两个操作符是可以混合使用的，其遵循的原则保持一致，且是从左向右依次判断，结合这两种操作符，可以实现类似于`if then else`的逻辑结构。如`cmd1 && cmd2 || cmd3`意思就是如果`cmd1`成功，则执行`cmd2`，否则执行`cmd3`。但务必注意的是，此处并非真正意思上的`if then else`逻辑，因为如果`cmd2`也执行失败，`cmd3`其实也会被执行。如下例：

```terminal
# echo hello && echo ok || echo world
hello
ok
# echo hello && rm dfsdf || echo world
hello
rm: cannot remove ‘dfsdf’: No such file or directory
world
```

`&&`相当于将两条命令逻辑上连成了一条命令，这样就变成了`cmd1-2 || cmd3`，其中`cmd1-2`就是`cmd1 && cmd2`，因此，`cmd3`只要在`cmd1-2`失败的情况下都会被执行，而`cmd1-2`失败的情况有两种，一种是`cmd1`失败，一种是`cmd1`成功但是`cmd2`失败。同样的，`||`也会将两条命令连成一条命令，如`cmd1-2 || cmd3 && cmd4`就相当于`cmd1-2_3 && cmd4`，`cmd4`是否会执行，决定于`cmd1-2_3`是否失败，以具体例子说明：

```terminal
# echo hello && echo ok || echo world && rm dsdfsf || echo end
hello
ok
rm: cannot remove ‘dsdfsf’: No such file or directory
end
```

这行命令相当于`cmd1 && cmd2 || cmd3 && cmd4 || cmd5`，可以看出`cmd1`，`cmd2`，`cmd4`还是有`cmd5`被执行了，而`cmd3`没有执行。咱们来解析一下，为何是如此的执行结果。首先，`shell`从左往右扫描执行：

* 发现`cmd1 && cmd2`，由`&&`连成一个命令`cmd1-2`，因为两个命令都是成功的，所以都被执行了，这样可以认为`cmd1-2`成功
* 执行成功之后，接下来是`||`操作符，这里并不会因为前面的命令是成功的，而不再执行后面所有的命令，而是`||`操作符相当于将`cmd1-2`与`cmd3`连接成了`cmd1-2_3`，因为`cmd1-2`成功了，所以`cmd3`不再执行，但是`cmd1-2_3`相当于执行成功了
* 继续执行，发现是`&&`操作符，同样将`cmd1-2_3`与`cmd4`连接起来，记为`cmd1-2_3-4`，因为`cmd1-2_3`执行成功了，所以`cmd4`也被执行，但是`cmd4`执行失败了，所以`cmd1-2_3-4`相当于执行失败
* 继续执行，发现是`||`操作符，同样将`cmd1-2_3-4`与`cmd5`连成`cmd1-2_3-4_5`，因为`cmd1-2_3-4`执行失败，所以`cmd5`被执行

可见，`shell`永远都是从左往右扫描执行，`&&`跟`||`会将前后两个命令连接起来，根据两种操作符的规则就可以知道多个连起来的命令是如何执行的了。

## `#`符号

跟其它很多语言一样，`#`在`shell`里面用来注释。

## `\\`转义符号

`\\`符号可以用来转义一些特殊符号，如`$`，`#`等。

特别指出的是，如果转义符号放在行末单独使用，则用来连接下一行。

# shell变量

## 基本概念

### 定义跟引用

`shell`中也可以使用变量，变量不需要像其它语言一样需要预先声音。`shell`中赋值给一个不存在的变量就相当于定义了改变量，如`name="Mr. Hao"`，就定义了`name`变量，后续如果再对`name`赋值，就相当于改变改变量的值。与很多语言不同的是，`shell`中变量引用以`$`符号开头，后面跟变量的名字。如前面的变量，引用如下`echo "$name"`。需要注意的是，在`shell`中，变量名是大小写敏感的。

在`shell`展开中会自动展开变量的引用，即便该变量处在双引号中。但是，如果变量引用在单引号中，`shell`不会对其进行解析。

```terminal
# name="Mr. Hao"
# echo "$name"
Mr. Hao
# set -x
# echo '$name'
+ echo 'Mr. Hao'
$name
```

### 查找变量

可以使用`set`命令来查找所定义的变量：

```terminal
# set | grep -E '^name='
name='Mr. Hao'
```

### 删除变量

与很多语言不同的是，在`shell`中定义的变量是可以删除的，使用`unset`命令删除定义的变量。

```terminal
# set | grep -E '^name='
name='Mr. Hao'
# unset name
# set | grep -E '^name='
```

## 特殊变量

在`shell`中预定义了很多特殊的变量，这一节咱们来说一下常见的几个变量。

### `$PS1`变量

在`shell`终端输入命令时，咱们总是可以看到在输入行首总是会有提示符，如：

```terminal
mrhao:~$
```

其中，`mrhao:~# `就是提示符，这个字串实际上是由`shell`变量`$PS1`决定的。如果咱们改变一下该变量的值，提示符也会相应的改变：

```terminal
mrhao:~$ PS1="hello > "
hello > echo "PS1 value is '$PS1'"
PS1 value is 'hello > '
hello >
```

为了方便在提示符中显示系统的某些实时信息，`$PS1`变量定义了一些特殊的字符：

|------+--------------------|
| 字符 | 说明               |
|------+--------------------|
| \w   | 表示工作目录       |
| \u   | 表示用户名         |
| \h   | 表示系统的hostname |
|------+--------------------|

当然，这里只列举了几个，详细的可以查看Linux手册。另外，`$PS1`中还可以对对其中不同部分采用不同颜色显示，如：

```terminal
# RED='\[\033[01;31m\]'
# WHITE='\[\033[01;00m\]'
# GREEN='\[\033[01;32m\]'
# BLUE='\[\033[01;34m\]'
# PS1="$GREEN\u$WHITE@$BLUE\h$WHITE\w\$ "
mrhao@mrhao-host~$ echo "$PS1"
\[\033[01;32m\]\u\[\033[01;00m\]@\[\033[01;34m\]\h\[\033[01;00m\]\w$
```
