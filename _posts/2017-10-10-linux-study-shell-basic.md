---
layout:     post
title:      "Linux系统介绍（三）shell基础"
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

首先，咱们来了解一下，什么是`Shell`。操作系统内核给我们提供了各种接口，同时也提供了各种用户层的库，理论上我们基于这些可以编写程序实现各种我们想要的功能，不过问题是，咱们不可能做什么事情都要重新编写程序，这样使用起来也太困难了。因此，操作系统（包括Linux）通常都会引入一个`Shell`这样的特殊程序，这个程序会接受输入的命令然后执行，并可能将执行结果呈现出来。总结来说，`Shell`是一个从输入设备或者文件读取命令，并且解释、执行的用户态程序。

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

```
# echo Hello World
Hello World
# echo   Hello World
Hello World
#    echo Hello World
Hello World
```

## 引号

当然，有时候你需要在一个参数中包括空格，这样的话你就需要将这个参数以引号引起来，引号包括了单引号`'`跟双引号`"`，两者都可以。`shell`会将引号中的字符串视为一个参数，不论里面有没有空格。当然，特别指出的是，不要用反引号`` ` ``，反引号将在后面详细讲述。

如命令`echo 'Hello World!'`在`shell`解析之后会有两个参数，分别为`echo`跟`Hello World!`。而如果不用引号`echo Hello World!`，则将解析为三个参数。

> 特别提一下，对于`echo`命令，如果需要输出需要转义的字符，如回车等，则需要执行`echo -e "Hello World!\n"`，如果不加`-e`，则`\n`会被直接显示出来。
>
>    ```
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

    由于历史原因，POSIX标准将一些内部命令划分为特殊内部命令，特殊的之处在于这些命令的查找时间以及命令运行后的状态等方面，只有当Bash以[POSIX模式](http://www.gnu.org/software/bash/manual/bashref.html#Bash-POSIX-Mode)运行时，这些命令才是特殊命令，否则它们跟其它内部命令没啥区别。特殊内部命令包括了`break : . continue eval exec exit export readonly return set shift trap unset`。

**内部命令可能会被提前至于内存中，因此运行起来会比外部命令要快。**对于外部命令，可以认为除了内部命令之后就可以认为是外部命令了，通常来讲，`/bin`跟`/sbin`下的都是外部命令，当然，应用有关的通常也是外部命令。

我们可以通过`type`命令来查看一个命令是否是内部命令：

```
# type cd
cd is a shell builtin
# type awk
awk is /usr/bin/awk
```

另外，对于很多内部命令，它们可能对应的会有外部命令版本，可以通过`type`命令来查看：

```
# type -a echo
echo is a shell builtin
echo is /usr/bin/echo
# type -a cd
cd is a shell builtin
cd is /usr/bin/cd
```

反过来，我们一般可以通过命令`which`来查询一个命令是否是外部命令：

```
# which awk
/usr/bin/awk
# which .
/usr/bin/which: no . in (/opt/rh/rh-python34/root/usr/bin:/usr/java/default/bin/:/usr/local/git/bin:/opt/ActiveTcl-8.5/bin:/root/perl5/bin:/root/env/maven/apache-maven-3.3.3/bin:/root/soft/wrk/wrk-4.0.1:/root/usr/go/bin:/usr/local/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin)
```

总结一下，通过`which`查询出来的是其外部命令版本，通过`type`默认查询出来的是内部命令：

```
# which echo
/usr/bin/echo
# type echo
echo is a shell builtin
```

对于内部命令的详细说明，可以查看[GNU的文档](http://www.gnu.org/software/bash/manual/bashref.html#Shell-Builtin-Commands)。

## 别名

可以用`alias`命令给一个命令取一个别名：

```
# alias print=echo
# print "hello"
hello
# type print
print is aliased to `echo'
```

别名一个常用的用法是用来缩写已知的命令：

```
# type ls
ls is aliased to `ls --color=auto'
```

可见`ls`命令实际上是命令`ls --color=auto`的别名，这样就相当于改变了`ls`命令的默认行为了。在这种情况下，如果仍然想用原先的命令，可以在别名前加反斜杠`` \ ``：

```
# type ls
ls is aliased to `ls --color=auto'
# \ls
Test1  test2  test.cpp  test.log  time.log
```


前面咱们通过`type`命令来查看命令的别名，实际上更加推荐采用`alias`或者`which`来查看：

```
# alias ls
alias ls='ls --color=auto'
# which ls
alias ls='ls --color=auto'
    /usr/bin/ls
```

如果要取消别名，则可以采用`unalias`命令:

```
# which ls
alias ls='ls --color=auto'
    /usr/bin/ls
# unalias ls
# which ls
/usr/bin/ls
```


## 显示shell展开的结果

由于`shell`展开的存在，你输入的命令被展开之后可能会发生变化，如果需要知道`shell`展开之后的命令，可以使用内部命令`set`来修改`shell`的默认参数来显示：

```
# set -x
++ printf '\033]0;%s@%s:%s\007' root traffic-base1 '~'
# echo hello         world
+ echo hello world
hello world
++ printf '\033]0;%s@%s:%s\007' root traffic-base1 '~'
```

其中，以`+`开头的就是展开之后的命令，可见展开之后，`shell`将多余的空格去掉了。如果不要再显示了，可以输入命令`set +x`。

# shell控制操作符 (Control Operators）

## `$?`操作符

每个命令执行完后都会有个退出码（Exit Code），其值为0时表示命令成功，否则命令失败。这个退出码可以通过`$?`来访问，执行完命令后立马访问`$?`可以获取该命令的退出码，并以此来判断命令是否成功。每个命令的执行都会产生新的退出码，所以请务必在命令执行完，立刻访问`$?`来获取退出码。

初看起来，`$?`似乎是一个`shell`变量，但实际上并非如此，因为你无法对`$?`赋值。`$?`准确来说是`shell`的一个内部参数。

## 分号`;`

`shell`命令输入时，你可以将多个命令输入在一行，只要在不同命令之间以分号`;`隔开，当然分号不能是在引号中。
> 必须注意的是，如果将多个命令以`;`连接在一起，执行的结果通过`$?`查询出来将只是最后一个命令的结果

## `&`符号

通常情况下，`shell`会在前台执行命令，并等待命令结束才返回。如果需要将命令放到后台去执行，可以使用`&`符号放在命令最后面，这样的话命令会被放在后台执行，`shell`会立刻返回而不用等待命令结束。
> 注意的是，即便放在后台执行，但是如果不处理好命令的输入，则命令的输出可能会继续在当前的终端输出，后面会讲述如何处理命令的输出。

## `&&`操作符

此操作符表示逻辑与，你可以将两个命令用此操作符连接起来，如`cmd1 && cmd2`，只有当`cmd1`执行成功之后，`cmd2`才会被执行。这里的成功指的是`cmd1`的退出码是0。

```
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

```
# echo hello && echo ok || echo world
hello
ok
# echo hello && rm dfsdf || echo world
hello
rm: cannot remove ‘dfsdf’: No such file or directory
world
```

`&&`相当于将两条命令逻辑上连成了一条命令，这样就变成了`cmd1-2 || cmd3`，其中`cmd1-2`就是`cmd1 && cmd2`，因此，`cmd3`只要在`cmd1-2`失败的情况下都会被执行，而`cmd1-2`失败的情况有两种，一种是`cmd1`失败，一种是`cmd1`成功但是`cmd2`失败。同样的，`||`也会将两条命令连成一条命令，如`cmd1-2 || cmd3 && cmd4`就相当于`cmd1-2_3 && cmd4`，`cmd4`是否会执行，决定于`cmd1-2_3`是否失败，以具体例子说明：

```
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

## `\`转义符号

`` \ ``符号可以用来转义一些特殊符号，如`$`，`#`等。

> 特别指出的是，如果转义符号放在行末单独使用，则用来连接下一行。

# shell变量

## 基本概念

### 定义跟引用

`shell`中也可以使用变量，变量不需要像其它语言一样需要预先申明。`shell`中赋值给一个不存在的变量就相当于定义了变量，如`name="Mr. Hao"`，就定义了`name`变量，后续如果再对`name`赋值，就相当于改变改变量的值。与很多语言不同的是，`shell`中变量引用以`$`符号开头，后面跟变量的名字。如前面的变量，引用如下`echo "$name"`。**需要注意的是，在`shell`中，变量名是大小写敏感的。**

在`shell`展开中会自动展开变量的引用，即便该变量处在双引号中。但是，如果变量引用在单引号中，`shell`不会对其进行解析。

```
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

```
# set | grep -E '^name='
name='Mr. Hao'
```

### 删除变量

与很多语言不同的是，在`shell`中定义的变量是可以删除的，使用`unset`命令删除定义的变量。

```
# set | grep -E '^name='
name='Mr. Hao'
# unset name
# set | grep -E '^name='
```

### `export`声明

通常情况下，`shell`在执行命令的时候会为该命令创建子进程。如果希望将当前的变量作用到子进程，则需要将变量`export`声明，这种变量称之为环境变量，如：

```
# var1="hello"
# export var2="world"
# bash
# echo "var1=$var1, var2=$var2"
var1=, var2=world
```

其中，`bash`命令开启了一个新的`shell`，可见只有`export`声明的变量在新的`shell`中才是可见的。环境变量可以通过`env`命令列举出来，在后面一节会详细讲述。此外，如果需要将非`export`变量重新声明为`export`变量，则只需要用`export`重新声明一下即可：

```
# var1=hello
# env | grep var1
# export var1
# env | grep var1
var1=hello
```

### `env`命令

如果需要查看当前`shell`中有哪些`export`声明的变量，可以使用`env`命令，该命令会列出当前所有`export`声明的变量。请注意与`set`命令的区别，`set`命令会列出所有的变量，包括哪些不是`export`声明的变量。通常，我们把`env`命令输出的变量称之为`环境变量`。

此外，`env`也常用来为子`shell`预先定义一些临时变量，如：

```
# var1="hello"
# env var1="tmp" bash -c 'echo "$var1"'
tmp
# echo $var1
hello
```

其中，用`env`命令定义了临时变量`var1`，然后`bash`命令开启了一个子`shell`，并在子`shell`中执行了`echo "$var1"`命令。可见，输出了定义的临时变量，在命令结束后，又回到之前的`shell`，输出的也是之前`shell`中定义的值。当然，在使用`env`定义临时变量的时候，为了方便，通常我们可以省略`env`命令，如：

```
# var1="hello"
# var1="tmp" bash -c 'echo "$var1"'
tmp
# echo $var1
hello
```

另外，`env`命令还有一种常用的用法，就是用来开启一个干净的子`shell`，即在子`shell`中不继承所有的变量，即便这些变量在之前的`shell`中采用`export`声明，此时`env`命令需要加入`-i`的参数，如：

```
# export var1="hello world"
# bash -c 'echo "var1=$var1"'
var1=hello world
# env -i bash -c 'echo "var1=$var1"'
var1=
```

可见，使用`env -i`之后，即便`var1`被`export`声明，但是在子`shell`中也没有被继承。

### 变量解释

在前面章节，我们知道`shell`采用`$`符号引用变量，在`$`符号后紧跟变量的名字。而`shell`在提取变量名字的时候一般以非字母数字（non-alphanumeric）为边界，这有时候就会产生问题，如：

```
# prefix=Super
# echo Hello $prefixman and $prefixgirl
Hello  and
```

可见，`shell`并不能提取我们定义的变量`prefix`，因为其后并没有非字母数字的字符为界。这种情况下，我们可以使用`{}`将变量名保护起来。

```
# prefix=Super
# echo Hello ${prefix}man and ${prefix}girl
Hello Superman and Supergirl
```

### 非绑定（unbound）变量

所谓非绑定（unbound）变量其实指的是没有预先定义的变量，或者说不存在的变量。默认情况下，`shell`在解释这种变量的时候会以空字符串替代：

```
# echo $unbound_var

```

如果需要`shell`在这种情况下报错，可以配置`shell`选项`set -o nounset`，或者简写为`set -u`：

```
# echo $unbound_var
bash: unbound_var: unbound variable
# set +u
# echo $unbound_var

```

当然，由例子中可以看到，要取消该配置，可以相应的设置`set +o nounset`，或者简写为`set +u`。

## 特殊变量

在`shell`中预定义了很多特殊的变量，这一节咱们来说一下常见的几个变量。

### `$PS1`变量

在`shell`终端输入命令时，咱们总是可以看到在输入行首总是会有提示符，如：

```
mrhao:~$
```

其中，`mrhao:~$ `就是提示符，这个字串实际上是由`shell`变量`$PS1`决定的。如果咱们改变一下该变量的值，提示符也会相应的改变：

```
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

```
# RED='\[\033[01;31m\]'
# WHITE='\[\033[01;00m\]'
# GREEN='\[\033[01;32m\]'
# BLUE='\[\033[01;34m\]'
# PS1="$GREEN\u$WHITE@$BLUE\h$WHITE\w\$ "
mrhao@mrhao-host~$ echo "$PS1"
\[\033[01;32m\]\u\[\033[01;00m\]@\[\033[01;34m\]\h\[\033[01;00m\]\w$
```

### `$PATH`变量

当我们在Linux的terminal里面输入命令的时候，`shell`需要在一系列的目录中查找输入的命令，如果没有查找到会直接报`command not found`的错误。而这些查找的目录就定义在`$PATH`变量中。

```
# echo $PATH
/opt/rh/rh-python34/root/usr/bin:/usr/java/default/bin/:/usr/local/git/bin:/opt/ActiveTcl-8.5/bin:/root/perl5/bin:/root/env/maven/apache-maven-3.3.3/bin:/root/soft/wrk/wrk-4.0.1:/root/usr/go/bin:/usr/local/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
```

其中，每个目录以`:`隔开，如果需要增加目录，可以：

```
# PATH=$PATH:/opt/local/bin
# echo $PATH
/opt/rh/rh-python34/root/usr/bin:/usr/java/default/bin/:/usr/local/git/bin:/opt/ActiveTcl-8.5/bin:/root/perl5/bin:/root/env/maven/apache-maven-3.3.3/bin:/root/soft/wrk/wrk-4.0.1:/root/usr/go/bin:/usr/local/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/opt/local/bin
```

> 加入新的路径的时候请务必带上之前的路径，`$PATH:<new path>`否则，很多默认的系统路径将被覆盖，导致很多命令失效。

特别注意的是，`$PATH`变量中目录的顺序是很重要的，如果`shell`在前面的目录中找到了命令，则不会查找后面的目录。如果你想把某个重名的命令优先执行，就需要把它对应的目录放在`$PATH`的前面。

### 网络代理变量

在Linux系统中，很多时候我们需要访问外部网络，比如使用`curl`命令下载文件等等。而有的时候，访问访问外部网络咱们需要设置代理，在Linux系统中，使用网络代理非常简单，只要配置几个变量即可：

|-------------+-------------------------------------------------------------------------|
| 变量        | 说明                                                                    |
|-------------+-------------------------------------------------------------------------|
| http_proxy  | 设置访问`http`请求所需要的代理，如`http_proxy=http://10.10.10.100:80`   |
| https_proxy | 设置访问`https`请求所需要的代理，如`https_proxy=http://10.10.10.100:80` |
| ftp_proxy   | 设置访问`ftp`请求所需要的代理，如`ftp_proxy=http://10.10.10.100:80`     |
| no_proxy    | 设置哪些域名或者IP不需要走代理，如`no_proxy=localhost,127.0.0.1`        |
|-------------+-------------------------------------------------------------------------|

### `$PWD`变量

`PWD`变量是一个由`shell`自动设置的变量，其值表示当前目录的绝对路径，与命令`pwd`输出相同。

# `shell`嵌入与`shell`选项

## `shell`嵌入（shell embedding）

`shell`可以嵌入在同一个命令行中，也就是`shell`在扫描解释命令行的时候，可能会从当前的`shell`进程中`fork`出一个新的`shell`进程，并将有关命令放在新进程中运行。如下例：

```
# var1=hello
# echo $(var1=world; echo $var1)
world
# echo $var1
hello
```

如其中`$()`便开启了一个新的`shell`进程，或者成为子`shell`，并在此`shell`中运行命令`var1=world; echo $var1`，此时输出的是子`shell`中定义的`var1`。当命令结束后，子`shell`进程退出，并将输出的结果`world`返回给之前的`shell`（或者父`shell`）的`echo`命令，父`shell`最后输出`world`。而且，在子`shell`中定义相同的`var1`变量并不会改变父`shell`中的变量。

***特别注意的是，因为子`shell`是`fork`出来的进程，根据Linux进程`fork`的特点，子进程将共享父进程的数据空间，而只在写的时候拷贝新的数据空间，因此，创建出来的子`shell`是会继承所有父`shell`的变量，不论该变量是否被`export`声明***

```
# var1=hello
# var2="$(echo $var1 world)"
# echo $var2
hello world
```

可见，虽然`var1`变量没有`export`声明，但是在子`shell`中还是可见的。这点与使用`bash -c`开启的`shell`是不同的。

用`$()`可以将子`shell`嵌入到命令行中，当然，`$()`是可以嵌套使用的，这样可以用来在子`shell`中开启它的子`shell`。

```
# A=shell
# echo $C$B$A $(B=sub;echo $C$B$A; echo $(C=sub;echo $C$B$A))
shell subshell subsubshell
```

### 反引号（backticks）

在上面我们可以通过`$()`将子`shell`嵌入命令行中，为了方便，我们同样可以用反引号`` ` ``将子`shell`嵌入。

```
# var1=hello
# echo `var1=world; echo $var1`
world
# echo $var1
hello
```

但是，使用反引号不能够嵌套子`shell`，因此如果需要嵌套子`shell`时，只能使用`$()`。

> 反引号跟单引号是本质的不同的，单引号与双引号一样，用来将连续的字串作为整体引起来，只不过单引号中将不执行变量的引用解析，而反引号则是嵌入子`shell`。


## `shell`选项

其实在前面咱们已经使用了不少`shell`的选项，如`set -u`在变量不存在是报错，`set -x`将`shell`展开的结果显示出来等。此外，可以才用`echo $-`将当期设置的`shell`选项打印出来。

# `shell`历史记录

在`shell`中执行命令的时候，`shell`会将最近的命令使用历史记录下来，这样你可以很方便的查看最近做了什么操作。

## 查看历史记录

命令`history`可以用来查看`shell`的历史记录，里面记录了你最近输入的所有命令。当然，很多时候你更加关心最近的几个命令，你可以使用`history 10`来显示最近的10个命令。另外，`shell`通常还会将最近的历史记录写在`~/.bash_history`文件中，因此查看该文件同样可以查看历史记录。

## 执行历史的命令

`shell`提供了很多高级用法使得你可以很方便的执行以前执行过的命令。

首先，咱们先显示一下过去的10个命令，可以看到每个命令前面都有其对应的序号。

```
# history 10
 1000  history
 1001  history 10
 1002  echo "hello world"
 1003  ls -l
 1004  ps -ef | grep named
 1005  env | grep http
 1006  grep hello /var/log/messages
 1007  tmux ls
 1008  find . -name "hello"
 1009  history 10
```

下面列举比较常用的`shell`重复执行历史记录中命令的方法：

|-----------+-------------------------------------------------------------------------------------------------------|
| 命令      | 说明                                                                                                  |
|-----------+-------------------------------------------------------------------------------------------------------|
| !!        | 在`shell`中输入两个感叹号会执行上一个命令                                                             |
| !keyword  | 输入一个感叹号后跟关键字，会搜索历史记录中最先以该关键字开始的命令。如`!find`会执行序号为1008的命令。 |
| !?keyword | 执行历史记录中第一个包括keyword关键字的命令                                                           |
| !n        | 其中n代表历史记录中的序号，表示执行序号为n的命令。                                                    |
| !-n       | 执行倒数第n个命令，如`!-1`其实就相当于`!!`                                                            |
| cmd!*     | 执行命令`cmd`，其中`!*`会以上一条命令的所有参数替代                                                   |
|-----------+-------------------------------------------------------------------------------------------------------|

另外，对于`!keyword`的用法，还有一个高级功能，你可以将符合该条件的命令进行改造后执行，如：

```
# echo "test1"
test1
# !ec:s/1/2/
echo "test2"
test2
```

其中，`:s/1/2/`将命令`echo "test1"`替换成`echo "test2"`然后执行了。对于`cmd!*`，示例如下：

```
# ctt /etc/passwd | cut -d: -f1
-bash: ctt: command not found
# cat !*
cat /etc/passwd | cut -d: -f1
root
bin
daemon
adm
...
```

## 搜索历史记录

在`shell`终端中按Ctrl-r会打开`shell`的搜索模式，在改模式下输入关键字会显示最近包含改关键字的命令，再按一下Ctrl-r会继续显示前面一条符合条件的命令，找到你需要的命令后回车就可以执行改命令了。

## 修改历史记录的有关配置

有多个配置可以用来改变历史记录的有关信息，通常都是通过有关环境变量来配置：

|---------------+--------------------------------------------------------------------------------------------|
| 环境变量      | 说明                                                                                       |
|---------------+--------------------------------------------------------------------------------------------|
| $HISTSIZE     | 这个变量用来配置`shell`应该保持多少行的历史记录，在很多发行版本中，默认值一般为500或者1000 |
| $HISTFILE     | 这个变量用来配置历史记录文件存放的位置，通常来讲，默认路径为`~/.bash_history`              |
| $HISTFILESIZE | 这个变量用来配置历史记录文件可以存放多少行的历史记录                                       |
|---------------+--------------------------------------------------------------------------------------------|

## 阻止记录某些命令

在有些时候，我们并不想把某些命令记录在历史记录中，比如有的命令里面包括了敏感信息如密码等。在新版本的`shell`中，通常我们可以在输入的命令前面加入空格，这样`shell`就不会记录这样的命令，当然，如果你的发行版本默认并不支持，你可以配置环境变量来打开这个功能：

```sh
export HISTIGNORE="[ \t]*"
```

例如：

```
# history 5
 1023  ls -l
 1024  echo ""
 1025  history 5
 1026  ls
 1027  history 5
#  echo "password=123456"
password=123456
# history 5
 1025  history 5
 1026  ls
 1027  history 5
 1028   echo "password=123456"
 1029  history 5
# export HISTIGNORE="[ \t]*"
# history 5
 1027  history 5
 1028   echo "password=123456"
 1029  history 5
 1030  export HISTIGNORE="[ \t]*"
 1031  history 5
#  echo "password=123456"
password=123456
# history 5
 1027  history 5
 1028   echo "password=123456"
 1029  history 5
 1030  export HISTIGNORE="[ \t]*"
 1031  history 5
```

可见，在设置`$HISTIGNORE`变量之后，在前面加了空格的命令将不再记录。这在保护敏感信息的时候非常有用。

# 文件匹配(File Globbing)

文件匹配(File Globbing)又成为动态文件名生成，用它可以非常方便的在`shell`中输入文件名。

## `*`星号

`*`星号在`shell`中用来匹配任意数量的字符，比如文件名`File*.mp4`，将匹配以`File`开头，`.mp4`结尾的任何文件名。`shell`在扫描解释命令的时候会自动去查找符合该匹配的所有文件或目录。当然，你也可以只用`*`来匹配所有的文件及目录，但请注意，只使用`*`跟不带`*`还是有所区别的，

```
# ls
definition.yaml  example  __init__.py  tags.yaml  test.py  test_sample.html  test_sample.py
# ls *
definition.yaml  __init__.py  tags.yaml  test.py  test_sample.html  test_sample.py

example:
testcase
```

可见，带上`*`后不仅把当前目录的所有文件及目录显示出来，而且还把目录下的内容显示出来了。

## `?`问号

问号用来匹配一个字符，如`File?.mp4`可以匹配`File1.mp4`。

## `[]`方括号

`[]`方括号也用来匹配一个字符，但是在括号里面可以指定一个字符集用来限定匹配的字符必须在该字符集内，字符集里面的字符顺序没有关系。

```
# ls
file1  file2  file3  File4  File55  FileA  fileab  Fileab  FileAB  fileabc
# ls File[5A]
FileA
# ls File[A5]
FileA
# ls File[A5][5b]
File55
```

如果需要匹配不在某个字符集里面的字符，可以在`[]`第一个字符加入`!`：

```
# ls file[!5]*
file1  file2  file3  fileab  fileabc
```

特别的，为了方便，`[]`中可以使用`-`来定义一些连续的字符集（Range匹配），常用的这类字符集包括：

|--------+--------------------|
| 字符集 | 说明               |
|--------+--------------------|
| 0-9    | 表示数字字符集     |
| a-z    | 表示小写字母字符集 |
| A-Z    | 表示大写字母字符集 |
|--------+--------------------|

当然，你也不必要把所有范围都包括在内，如`[a-d]`可以用来限定从`a`到`d`的小写字母集。另外，用`-`连起来的字符集还可以跟其它字符集一起使用，如`[a-d_]`表示`a`到`d`的小写字母加上`_`所组成的字符集。

* Range匹配的大小写问题

    对于`[]`的Range匹配，还有一点很重要。在很多发行版本中，默认情况下，`[]`的Range匹配是忽略大小写的

    ```
    # ls
    Test1  test2
    # ls [a-z]*
    Test1  test2
    # ls [A-Z]*
    Test1  test2
    # ls [t]*
    test2
    # ls [T]*
    Test1
    ```

    > 注意，是`[]`的Range匹配会忽略大小写，而如果不是Range匹配还是大小写敏感的：
    >
    > ```
    > # ls
    > Test1  test2
    > # ls [T]*
    > Test1
    > # ls [t]*
    > test2
    > ```

    如果需要大小写敏感，可以设置环境变量`LC_ALL`：

    ```
    # LC_ALL=C
    # ls [a-z]*
    test2
    # ls [A-Z]*
    Test1
    ```

    当然，请务必注意，`LC_ALL`的会改变当前的语言环境，还请慎重使用，建议只在临时的子`shell`中使用。

## 阻止文件匹配(File Globbing)

有时候我们就是需要输出`*`等匹配符号，这个时候就需要阻止`shell`做相应的匹配。可以使用转义符号`` \ ``来做到这点，或者将匹配符号放在引号中：

```
# echo *
Test1 test2
# echo \*
*
# echo '*'
*
# echo "*"
*
```

# shell快捷键

`shell`中支持非常多的快捷键，可以非常方便我们输入命令：

|--------+---------------------------------------------------------------------------|
| 快捷键 | 说明                                                                      |
|--------+---------------------------------------------------------------------------|
| Ctrl-d | 表示`EOF`的意思，在shell终端中输入该快捷键会退出该终端                    |
| Ctrl-z | 该快捷键用来暂停一个在shell终端中正在执行的进程，暂停后可以用`fg`命令恢复 |
| Ctrl-a | 输入命令时跳到行首                                                        |
| Ctrl-e | 跳到行尾                                                                  |
| Ctrl-k | 删除从光标到行尾的部分                                                    |
| Ctrl-y | 粘贴刚刚删除的部分                                                        |
| Ctrl-w | 删除从光标至其左边第一个空格处                                            |
|--------+---------------------------------------------------------------------------|
