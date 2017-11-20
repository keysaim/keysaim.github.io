---
layout:     post
title:      "Linux系统介绍（一）命令行"
subtitle:   ""
date:       2017-10-09
author:     "keysaim"
header-img: ""
catalog: true
tags:
    - Linux
    - study
    - 学习
---

# 概述

随着Linux的发展，现在已经有了非常多的桌面版本，比如著名的Ubuntu。用这些桌面版本系统，可以满足基本的操作，然而对于一些高级点的操作，还是离不开Linux的命令行(Command Line)。而Linux的精髓也更多的体现在命令行上，其强大的功能，海量的工具，可以帮你轻而易举的完成各种复杂的系统管理操作。本文将详细讲述Linux命令行。

# 基础命令

## 帮助类

### man

Linux有着海量的命令，而每个命令又有很多的不同参数，要记住所有的这些命令是比较困难的，因此，在使用Linux命令行的时候，必须时刻记着查看Linux的帮助，而查看帮助就是采用`man`命令。

* 查看命令帮助

    以`ls`命令为例，如果要查看帮助的话可以输入`man ls`，查看基本的帮助信息也可以直接`ls --help`。其将以分页的形式显示该命令的完整文档，操作该文档的基本命令有：

    * 按`u`上翻页
    * 按`d`下翻页
    * 按空格下翻页
    * 按回车下移一行
    * 按`/`进入搜索模式，输入要搜索的关键字，按回车搜索。
    * 按`n`搜索下一个
    * 按`N`搜索上一个
    * 按`q`退出查看

* 查看配置文件的帮助

    有些系统的配置文件也同样有对应的帮助文档，可以通过`man $configfile`来查看，比如`/etc/system/sysctl.conf`配置文件，查看其帮助可以采用命令`man sysctl.conf`。

* 查看后台进程(daemon)的帮助

    Linux在后台运行着很多的程序（称为daemon），如果需要查看某个daemon的帮助，可以用命令`man $daemon`来查看。如`man ntpd`将查看时间同步daemon的帮助文档。

* 搜索需要查看的命令

    Linux命令实在太多，有时候如果不记得准确的命令的名字，可以采用`man -k $keyword`来搜索，如`man -k syslog`将列出相关命令：

    ```
    # man -k syslog
    ipmievd (8)          - IPMI event daemon for sending events to syslog
    logger (1)           - a shell command interface to the syslog(3) system log module
    rsyslog.conf (5)     - rsyslogd(8) configuration file
    rsyslogd (8)         - reliable and extended syslogd
    ```

### whatis

`man`命令将展示完整的文档，可以通过`whatis`来查看命令的简单介绍。

### whereis

如果需要知道某个命令的完整路径，可以采用`whereis $command`来查看。

## 目录操作类

|----------+------------------------------------------|
| 命令     | 解释                                     |
|----------+------------------------------------------|
| pwd      | 查看当前目录路径                         |
| cd $path | 切换到其它路径                           |
| cd ~     | 返回home目录                             |
| cd ..    | 返回上一级                               |
| cd -     | 返回上一次的目录                         |
| ls $path | 查看目录下的内容                         |
| ls -a    | 显示目录下所有文件，包括隐藏文件         |
| ls -lh   | 列表的形式显示，`-h`以可读的方式显示大小 |
| mkdir    | 创建目录，要递归创建采用`mkdir -p`       |
|----------+------------------------------------------|

## 文件操作类

### 说明

* 大小写敏感
    在Linux系统中，文件名都是大小写敏感的。

* 所有都是文件
    Linux基本上将文件，目录，设备等等都视为文件。

### file命令

查看文件的类型，如：

```
# file bugs.tgz
bugs.tgz: gzip compressed data, from Unix, last modified: Tue Dec 13 01:38:27 2016
```

可以显示文件的类型及修改时间等信息。查看特殊文件如设备文件的时候，还可以带上`-s`的参数，这样可以识别更多的信息：

```
# file /dev/sda
/dev/sda: block special
# file -s /dev/sda
/dev/sda: x86 boot sector; partition 1: ID=0x83, active, starthead 32, startsector 2048, 1024000 sectors; partition 2: ID=0x8e, starthead 221, startsector 1026048, 208689152 sectors, code offset 0x63
```

### touch

`touch`用来创建空文件，或者用来更新文件时间为当前时间。如果加上`-t`参数，可以为文件设置指定的时间。

```
# date
Mon Oct  9 03:19:24 EDT 2017
# ls -l
total 4
-rw-r--r--. 1 root root 6 Jun 19  2016 login.html
# touch test.txt
# ls -l
total 4
-rw-r--r--. 1 root root 6 Jun 19  2016 login.html
-rw-r--r--. 1 root root 0 Oct  9 03:19 test.txt
# touch login.html
# ls -l
total 4
-rw-r--r--. 1 root root 6 Oct  9 03:19 login.html
-rw-r--r--. 1 root root 0 Oct  9 03:19 test.txt
# touch -t 201701011010 login.html
# ls -l
total 4
-rw-r--r--. 1 root root 6 Jan  1  2017 login.html
-rw-r--r--. 1 root root 0 Oct  9 03:19 test.txt
```

### 删除、复制、移动

|--------+--------------------------------------------------------------------------------------|
| 命令   | 解释                                                                                 |
|--------+--------------------------------------------------------------------------------------|
| rm     | 永久删除一个文件，对于此命令，没有所谓的垃圾箱，请慎重                               |
| rm -i  | 询问是否真的要删除                                                                   |
| rm -rf | 通常`rm`只是删除文件，如果需要删除目录的时候，必须带上`-r`参数，`-f`参数表示强制删除 |
| cp     | 拷贝文件                                                                             |
| cp -rf | 拷贝目录下所有的文件，并强制覆盖                                                     |
| mv     | 移动文件到另一个目录，或者在当前目录下对某个文件改名字                               |
|--------+--------------------------------------------------------------------------------------|

## 文件内容操作类

### head

查看某个文件的头几行，可以用`head`命令：

```
# head -n 5 /var/log/messages
Oct  9 03:10:01 traffic-base1 rsyslogd: [origin software="rsyslogd" swVersion="7.4.7" x-pid="697" x-info="http://www.rsyslog.com"] rsyslogd was HUPed
Oct  9 03:20:01 traffic-base1 systemd: Started Session 79213 of user root.
Oct  9 03:20:01 traffic-base1 systemd: Starting Session 79213 of user root.
Oct  9 03:28:26 traffic-base1 puppet-agent[14530]: Unable to fetch my node definition, but the agent run will continue:
Oct  9 03:28:26 traffic-base1 puppet-agent[14530]: Connection refused - connect(2)
# head -c 5 /var/log/messages
Oct  
```

其中`-n`参数表示显示多少行，不带此参数，默认显示10行。`-c`表示显示多少个字符，这里显示了前面5个字符。

### tail

与`head`相反，`tail`用来显示文件最后几行。同样`-n`可以用来限制多少行。

`tail`有个非常重要的用处，***就是用来监听某个动态文件的内容，比如实时查看某个日志文件***：

```
# tail -n 5 -F /var/log/messages
Oct  9 03:28:27 traffic-base1 puppet: from /usr/share/ruby/vendor_ruby/puppet/util/command_line.rb:146:in `run'
Oct  9 03:28:27 traffic-base1 puppet: from /usr/share/ruby/vendor_ruby/puppet/util/command_line.rb:92:in `execute'
Oct  9 03:28:27 traffic-base1 puppet: from /usr/bin/puppet:8:in `<main>'
Oct  9 03:30:01 traffic-base1 systemd: Started Session 79218 of user root.
Oct  9 03:30:01 traffic-base1 systemd: Starting Session 79218 of user root.
Oct  9 03:40:01 traffic-base1 systemd: Started Session 79223 of user root.
Oct  9 03:40:01 traffic-base1 systemd: Starting Session 79223 of user root.

```

它将不停的侦听文件的改变，并实时的将最后新写入的行打印出来。

### cat $file1 $file2 $file3 ...

要显示文件的所有内容，可以用`cat`命令。`cat`也可以用来创建新文件：

```
# cat > test.txt
Today is a good day!
# cat test.txt
Today is a good day!
```
这样就可以直接输入文件内容了，输入完成之后按Ctrl+d结束输入。当然，也可以定制结束符：

```
# cat > test.txt <<stop
> It's a good day!
> stop
# cat test.txt
It's a good day!
```

### tac

`tac`名字其实就是`cat`倒过来写，所以其作用也是一样，就是把文件以倒着的顺序显示出来。

### more跟less

`cat`会直接把整个文件一次性的显示出来，当文件较大时，显示可能会刷屏，这样不利于查看。如果需要以翻页的形式显示文件的内容，可以采用`more`或者`less`命令，其查看方式跟`man`命令的查看方式类似，可以参考前面的说明。

### strings

`strings`命令会将文件中的可读字符显示出来，即便改文件是一个二进制文件。

```
# strings a.out | tail -n 5
_edata
_Znwm@@GLIBCXX_3.4
_ZN3Out5InnerC1EPS_
main
_init
```
其中，`a.out`是一个二进制文件。

