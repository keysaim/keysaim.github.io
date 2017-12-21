---
layout:     post
title:      "Linux系统介绍（五）常用命令"
subtitle:   ""
date:       2017-11-17
author:     "keysaim"
header-img: ""
catalog: true
tags:
    - Linux
    - study
    - 学习
---

# cat命令

很多时候我们通过`cat`命令来查看文件内容，它会将文件的所有内容显示出来。当然，`cat`也可以通过管道接收数据，它主要完成的是将从管道接收的输入导到输出。

# more跟less命令

有时候用`cat`命令来显示一个较大的文件并不方便，整个文件内容一次性显示出来简直就是刷屏了。如果需要一页页的显示内容，可以使用`more`或者`less`命令，这两个命令会以分页的形式显示文件内容，至于使用哪个命令完全看个人习惯了。此外，这两个命令不仅可以分页显示，而且在分页模式下，你可以用快捷键方便的浏览及搜索：

    * 按`d`下翻页
    * 按空格下翻页
    * 按回车下移一行
    * 按`/`进入搜索模式，输入要搜索的关键字，按回车搜索。
    * 按`n`搜索下一个
    * 按`q`退出查看

# tee命令

`tee`命令一般从管道接收数据，这点与`cat`类似，将stdin导到stdout。不同的是，`tee`同时还可以指定一个文件作为输出。这点非常有用，有时候我们想一般看到命令的输出，同时又希望将输出保存到文件中，这时候用`tee`最为合适。


```
# date | tee time.log
Mon Nov 20 14:05:02 EST 2017
# cat time.log
Mon Nov 20 14:05:02 EST 2017
```

# date命令

`date`命令用来显示时间跟时区，比较常见的用法有：

* 默认显示

    ```
    # date
    Sun Nov 19 20:08:21 EST 2017
    # date -u
    Mon Nov 20 01:08:28 UTC 2017
    ```

    其中，`-u`参数表示显示UTC标准时间，即时区为0的时间。

* 指定显示格式

    除了默认输出，我们也可以指定显示的格式：

    ```
    # date +'%A %d-%m-%Y UTC %:z'
    Sunday 19-11-2017 UTC -05:00
    ```

    `date`支持非常多元化的格式，具体可以参考[这里](https://www.cyberciti.biz/faq/linux-unix-formatting-dates-for-display/)。

* 显示当前时间的秒数

    通常，在计算当前时间的秒数的时候，我们通常会以[Unix Epoch Time](https://en.wikipedia.org/wiki/Unix_time)为基准，用`date`命令可以非常方便的显示当前时间的秒数：

    ```
    # date +%s
    1511141040
    # date +%s --date='2017/11/19 09:56:00'
    1511103360
    ```

    其中，也可以通过参数`--date`指定时间来计算。反过来，如果我们知道了时间的秒数，需要显示其相对于Unix Epoch Time的时间，可以这么做：

    ```
    # date; date +%s
    Mon Nov 20 10:33:07 EST 2017
    1511191987
    # date --date=@1511191987
    Mon Nov 20 10:33:07 EST 2017
    ```

* 时间偏移计算

    有时候需要知道多少天前是什么时间，这时候需要用到时间偏移计算了：

    ```
    # date --date='100 seconds ago'
    Sun Nov 19 20:35:44 EST 2017
    # date --date='100 hours ago'
    Wed Nov 15 16:37:28 EST 2017
    # date --date='100 days ago'
    Fri Aug 11 21:37:34 EDT 2017
    ```

    `date`命令可以识别多种时间偏移写法，除了示例中的，还有`minutes months years`等，当然，也可以这样写:

    ```
    # date --date='+ 1000 seconds'
    Sun Nov 19 20:56:28 EST 2017
    # date --date='- 1000 seconds'
    Sun Nov 19 20:23:41 EST 2017
    # date --date='2017-11-19 00:00:00 + 1000 seconds'
    Sat Nov 18 09:00:01 EST 2017
    ```

    直接用`+`或者`-`表示以后或者以前的时间，也可以指定某个时间点然后偏移。

* 设置时间

    当然，你也可以通过`date`命令来设置时间：

    ```
    # date
    Sun Nov 19 20:44:11 EST 2017
    # date --set='Sun Nov 19 20:44:30 EST 2017'
    Sun Nov 19 20:44:30 EST 2017
    # date
    Sun Nov 19 20:44:31 EST 2017
    ```

    其中`--set`也可以简写为`-s`，时间格式非常灵活：

    ```
    # date -s '2017/11/20 10:19:50'
    Mon Nov 20 10:19:50 EST 2017
    ```

# cal命令

我们用`date`可以显示时间，同时咱们还可以通过`cal`命令来显示日历：

```
# cal
    November 2017
Su Mo Tu We Th Fr Sa
          1  2  3  4
 5  6  7  8  9 10 11
12 13 14 15 16 17 18
19 20 21 22 23 24 25
26 27 28 29 30
```

当然，你也可以指定要显示的日期，比如1949年10月的日历：

```
# cal 10 1949
    October 1949
Su Mo Tu We Th Fr Sa
                   1
 2  3  4  5  6  7  8
 9 10 11 12 13 14 15
16 17 18 19 20 21 22
23 24 25 26 27 28 29
30 31
```

# time命令

有时候我们需要知道一个命令运行了多少时间，这时候我们可以用`time`命令来计时：

```
# time sleep 1

real	0m1.018s
user	0m0.001s
sys  	0m0.002s
```

其中`sleep 1`用来睡眠1秒，`real`表示实际用了多少时间，`user`表示在用户态花了多少时间，`sys`则表示在内核花了多少时间。详细可以参考[这篇问答](https://stackoverflow.com/questions/556405/what-do-real-user-and-sys-mean-in-the-output-of-time1)。

# wc命令

`wc`命令是`Word Count`的简称，顾名思义就是用来统计单词的。

```
# cat test.log
Nov 17 00:27:20 traffic-base1 named[1212]: managed-keys-zone: Unable to fetch DNSKEY set .: timed out
# cat test.log | wc -w
14
```

参数`-w`表示统计单词数，这里的单词实际上指的是被空格分开的字符串。下面列举出`wc`命令的有关参数：

|------+------------------------|
| 参数 | 说明                   |
|------+------------------------|
| -w   | 统计多少单词           |
| -l   | 统计多少行             |
| -c   | 统计有多少个字节       |
| -m   | 统计有多少个字符       |
| -L   | 统计长度最长的行的长度 |
|------+------------------------|

注意，这里的字节跟字符的差别，在英文中基本上是一样的，但是在多字节语言中，其意义就不一样了：

```
# echo '你好' | wc -c
7
# echo '你好' | wc -m
3
```

# find命令

`find`命令用来查找文件或目录，这又是一个非常强大的且常用的命令，这里只介绍几种常见的用法：

* 根据名字查找

    这是基本且常见的用法：

    ```
    # find . -name "test*"
    ./test.log
    ./test2
    ```

    示例中表示在当前目录（用`.`表示）下包括其子目录，查找文件名以`test`开头的文件或者目录。默认情况下，`find`命令是大小写敏感的，如果需要忽略大小写，则可以改用参数`-iname`。

* 根据路径查找

    `-name`参数会根据名字查找，如果需要对路径进行匹配查找，则可以用`-path`参数：

    ```
    # find . -path "./tmp/*"
    ./tmp/Test1
    ./tmp/test2
    ./tmp/test.log
    ./tmp/test.cpp
    ./tmp/time.log
    ```

* 根据类型查找

    可以根据类型查找文件：

    ```
    # find . -type f
    ./Test1
    ./test.log
    ./test2
    ```

    当然，也可以同时根据类型跟文件名一起查找：

    ```
    # find . -type f -name "test*"
    ./test.log
    ./test2
    ```

    `f`表示文件，如果是查找目录的话则用`d`。

* 根据时间查找

    `find`命令还可以根据时间来查找文件目录，其中一个用法如下：

    ```sh
    find . -newer base_file
    ```

    表示在当前目录下查找比`base_file`文件更新的文件或者目录。此外，`find`还可以根据文件的[atime, ctime, mtime](https://www.unixtutorial.org/2008/04/atime-ctime-mtime-in-unix-filesystems/)来查找文件，如下，根据修改时间来查找：

    ```
    # date
    Fri Nov 17 20:34:52 EST 2017
    # ll
    total 20
    drwxr-xr-x.  2 root root   45 Nov 17 02:46 ./
    dr-xr-x---. 48 root root 8192 Nov 17 02:46 ../
    -rw-r--r--.  1 root root    0 Nov 16 00:17 Test1
    -rw-r--r--.  1 root root   34 Nov 17 02:46 test2
    -rw-r--r--.  1 root root  102 Nov 17 00:30 test.log
    # find . -mtime -1
    .
    ./test.log
    ./test2
    ```

    其中`-mtime`表示根据修改时间查找，`-1`表示最近一天。`find`支持的时间查找总结如下：

    |--------+----------------------------------------------|
    | 参数   | 说明                                         |
    |--------+----------------------------------------------|
    | -mtime | 根据修改时间，也就是`ls -l`显示的时间        |
    | -atime | 根据访问时间，也就是`ls -lu`显示的时间       |
    | -ctime | 根据状态改变的时间，也就是`ls -lc`显示的时间 |
    |--------+----------------------------------------------|

    时间值的表示说明：基准`+0`表示一天前，`-1.5`表示最近1.5天，`+1.5`表示2.5天前

* 逻辑查找

    `find`支持与或非逻辑的查找，比如查找所有C++的源文件，实际上需要找出后缀为`.cpp`跟`.h`的文件，需要用到`find`的逻辑或的查找：

    ```sh
    find . -name "*.cpp" -o -name "*.h"
    ```

    其中`-o`是`-or`的缩写，用来表示逻辑或的关系，而`-name "*.cpp"`与`-name "*.h"`为表达式，构成了`EXP1 or EXP2`的关系，只要文件或者目录满足其中一个表达式就会输出。`find`支持的逻辑关系如下：

    |------+------+--------------------------------------------------------------------|
    | 逻辑 | 参数 | 说明                                                               |
    |------+------+--------------------------------------------------------------------|
    | 与   | -a   | `-and`的缩写，逻辑与的关系，如`find . -type f -a -name "*.log"`    |
    | 或   | -o   | `-or`的缩写，逻辑或的关系, 如`find . -name "*.cpp" -o -name "*.h"` |
    | 非   | !    | `-not`的缩写，逻辑非的关系, 如`find . ! -name "*.cpp"`             |
    |------+------+--------------------------------------------------------------------|


此外，`find`命令还有一个非常重要且常见的用法，就是在找到文件后执行某个命令，改用法如下：

```shell
find . -name "*.log" -exec rm {} \;
```

表示删除当前目录包括子目录中以`.log`为后缀的所有文件。其中，`-exec`表示在找到后需要执行命令，而命令为`rm {} \;`，实际上此命令就是一般的`shell`命令，其中`{}`用来指代找到的文件或目录，这里`;`必须转义，因为需要传递给`find`本身，如果不转义，则会直接被`shell`解析使用了。每找到一个文件或目录，都会执行指定的命令，其中`{}`部分以文件路径替代。如果需要只执行一次命令，而把所有找到的文件作为参数传递给该命令，则需要用`+`替代`\;`，如：

```shell
find . -name "*.log" -exec rm {} +
```

假定找到的文件有`test.log`，`test1.log`，用`\;`的方式相当于执行两次：`rm test.log`跟`rm test1.log`；如果使用`+`则只有一次命令`rm test.log test1.log`。

# sort命令

顾名思义，这个命令就是用来排序的。

```
# head -n 5 /etc/passwd | sort
adm:x:3:4:adm:/var/adm:/sbin/nologin
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
root:x:0:0:root:/root:/bin/bash
```

默认情况下，`sort`命令以字典顺序对每行进行排序，如果不带参数，会将整行作为一个字符串进行比较。当然，你也可以指定以第几列进行排序：

```
# head -n 5 /etc/passwd | tr : " " | sort -k 3
root x 0 0 root /root /bin/bash
bin x 1 1 bin /bin /sbin/nologin
daemon x 2 2 daemon /sbin /sbin/nologin
adm x 3 4 adm /var/adm /sbin/nologin
lp x 4 7 lp /var/spool/lpd /sbin/nologin
```

这里先将`:`换成空格（关于`tr`命令，请参照本文关于`tr`命令的章节），然后以`-k`为参数，指定以第三列进行排序。下面列举其常用的一些参数：

|------+------------------------------------------------------------------------------------------------------------|
| 参数 | 说明                                                                                                       |
|------+------------------------------------------------------------------------------------------------------------|
| -k   | 以第几列排序，列以空格为分隔                                                                               |
| -r   | 默认`sort`以升序输出，`-r`参数则可以以降序输出                                                             |
| -n   | 在指定第几列的时候，可以强制`sort`把列的值以数字值进行排序，如下面的例子                                   |
| -t   | 默认情况下认为列是以空格为分隔，`-t`参数则可以指定分隔符，这样，上面的例子其实可以直接写成`sort -t : -k 3` |
| -f   | 忽略大小写                                                                                                 |
| -u   | 如果排序后出现重复的行，加上这个参数将只显示一行                                                           |
|------+------------------------------------------------------------------------------------------------------------|

```
# cat test2
10
3
5
3
# cat test2 | sort
10
3
3
5
# cat test2 | sort -n
3
3
5
10
# cat test2 | sort -n -u
3
5
10
```

# uniq命令

`uniq`是`unique`的简写，用来消除`sort`排序后重复的行，即相当于`sort`命令中的`-u`参数。但是，`uniq`不仅可以消除重复行，它还可以显示分别重复了多少行：

```
# cat test2 | sort -n | uniq -c
      2 3
      1 5
      1 10
```

还有些常用的参数如下：

|------+------------------------------------------------------------|
| 参数 | 说明                                                       |
|------+------------------------------------------------------------|
| -i   | 忽略大小写                                                 |
| -d   | 只打印有重复的行，每组一个，如果要打印组内所有的，则用`-D` |
| -u   | 只打印没有重复的行                                         |
| -f   | 比较的时候，忽略前面的N列                                  |
| -s   | 比较的时候，忽略前面的N个字符                              |
|------+------------------------------------------------------------|

# od命令

`od`可以用来显示二进制文件：

```
# cat test2
abcdefg
1234567
# od -t x1 test2
0000000 61 62 63 64 65 66 67 0a 31 32 33 34 35 36 37 0a
0000020
```

当然，也可以直接显示字符：

```
# od -c test2
0000000   a   b   c   d   e   f   g  \n   1   2   3   4   5   6   7  \n
0000020
```

当然，如果仅仅只是显示二进制内容，还可以使用`hexdump`命令了。

# 压缩类命令

## tar命令

在Linux中，用的最多的压缩命令就是`tar`命令了，在介绍其用法之前，需要清楚几个概念：

* 存档文件(Archive File)

    存档文件用来打包多个文件成一个文件，以方便在网络上传输。请注意，打包成的文件并没有被压缩。在Linux或Unix系统中，TAR文件是最为常用的存档文件（通常以`.tar`为文件后缀）。TAR文件的更多解释可以参考[这里](http://www.bitzipper.com/tar-file.html)。用`tar`命令可以生产TAR文件，示例如下：

    ```shell
    tar cvf tmp.tar tmp/
    ```

    具体参数在下面会具体解释，不带任何压缩格式的话，`tar`命令会生成一个TAR文件。相应的，如果需要解开TAR文件，可以这么做：

    ```shell
    tar xvf tmp.tar
    ```

* 压缩文件(Compressed File)

    TAR文件是没有被压缩的，大小基本保持不变。如果需要对文件进行压缩，则需要在`tar`命令中加入压缩格式对应的参数，具体在下面会说明。

下面来详细介绍`tar`命令的用法，`tar`的用法如下：

```
# tar --help
Usage: tar [OPTION...] [FILE]...
```

`tar`命令支持非常多的参数，这里列举比较常用的几种使用方式：

* 压缩文件或目录

    ```
    # tar czvf tmp.tgz tmp/
    tmp/
    tmp/Test1
    tmp/test.log
    tmp/test2
    ```

    其中，`czvf`表示参数选项；`tmp.tgz`表示压缩后的文件名，通常`tgz`是`tar.gz`的简写；`tmp/`表示被压缩的目录。参数的解释如下：

    |------+-------------------------------------------------------------|
    | 参数 | 说明                                                        |
    |------+-------------------------------------------------------------|
    | c    | `Create`的简写，表示生产压缩文件                            |
    | z    | 表示采用`gzip`的压缩格式，文件后缀通常为`.tar.gz`或者`.tgz` |
    | v    | 表示显示压缩的过程，会列出所有被压缩的文件                  |
    | f    | 指定压缩文件                                                |
    |------+-------------------------------------------------------------|

    需要注意的是，`tar`支持不同的压缩格式，除了`gzip`之外，还有：

    |----------+---------------------------------------------|
    | 参数     | 说明                                        |
    |----------+---------------------------------------------|
    | j        | 采用`bzip2`的压缩格式，文件后缀通常为`.bz2` |
    | `--lzip` | 采用`lzip`的压缩格式，文件后缀通常为`.lz`   |
    | `--xz`   | 采用`xz`的压缩格式，文件后缀通常为`.xz`     |
    | `--lzma` | 采用`lzma`的压缩格式，文件后缀通常为`.lzma` |
    |----------+---------------------------------------------|

    更多格式可以参考[这里](http://www.gnu.org/software/tar/manual/html_node/gzip.html)。当然，最为常用的两种格式为`gzip`跟`bzip2`，用法示例如下：

    ```shell
    tar czvf tmp.tgz tmp/
    tar cjvf tmp.bz2 tmp/
    ```

    其中`v`参数可选，如果不需要显示压缩过程的话。需要注意的是，`tar`压缩目录的时候会保持目录的结构。

* 解压文件

    对应于不同的压缩格式，解压参数稍微不一样，对于`gzip`跟`bzip2`分别示例如下：

    ```shell
    tar xzvf tmp.tgz
    tar xjvf tmp.tgz
    ```

    与压缩不同，`c`换成`x`表示解压，其它参数含义与压缩一样。默认情况下，解压的文件会放在当前目录，如果需要解压到某个目录下，则可以用`-C`参数：
    ```shell
    tar xzvf -C /tmp/ tmp.tgz
    tar xjvf -C /tmp/ tmp.tgz
    ```

* 列出压缩包里面的文件

    有时候我们需要先看看压缩包里面有哪些文件，但又并不想解压文件，可以采用`-t`参数：

    ```
    # tar czf tmp.tgz tmp/
    # tar tvf tmp.tgz
    drwxr-xr-x root/root         0 2017-11-17 02:46 tmp/
    -rw-r--r-- root/root         0 2017-11-16 00:17 tmp/Test1
    -rw-r--r-- root/root       102 2017-11-17 00:30 tmp/test.log
    -rw-r--r-- root/root        34 2017-11-17 02:46 tmp/test2
    ```

    其中，`tvf`会列出压缩包中的文件，不论采用何种压缩格式，甚至是没有被压缩的TAR文件。

* 从压缩包中提取特定的文件

    在列出压缩包里面的内容后，如果只想提取里面的某些文件，可以这么做：

    ```
    # tar xzvf tmp.tgz tmp/test.log tmp/test2
    tmp/test.log
    tmp/test2
    ```

    其中，根据不同的压缩格式请替换成不同的参数。请务必注意，指定的文件必须是完整的路径，而不能只是文件名。

    另外，如果想提取一组匹配某种条件的文件，可以使用`--wildcards`参数：

    ```
    # tar xzvf tmp.tgz --wildcards "*.log"
    tmp/test.log
    ```

# split命令

`split`命令用来将一个文件分成多个文件，比如将一个特别大的文件分成平均大小为40M的多个文件等。

```
# split -b 40M go1.6.linux-amd64.tar.gz go1.6.linux-amd64.tar.gz.part
# ll go1.6.linux-amd64.tar.gz.part*
-rw-r--r--. 1 root root 41943040 Nov 20 15:03 go1.6.linux-amd64.tar.gz.partaa
-rw-r--r--. 1 root root 41943040 Nov 20 15:03 go1.6.linux-amd64.tar.gz.partab
-rw-r--r--. 1 root root   913400 Nov 20 15:03 go1.6.linux-amd64.tar.gz.partac
```

其中，`go1.6.linux-amd64.tar.gz`是要被拆分的文件，`go1.6.linux-amd64.tar.gz.part`是拆分后文件的前缀，可以看到文件被拆分为三部分了。

`split`非常常见的用法是来将某个被压缩的文件拆分成小的部分，正如上例所示。那么，如何将拆分的文件重新合并呢？我们可以用`cat`将它们合并：

```shell
cat go1.6.linux-amd64.tar.gz.part* > go1.6.linux-amd64.tar.gz
```

# grep命令

过滤数据来说，用的最多的估计就是`grep`命令了，`grep`命令可以从文件或者管道中搜索数据并打印出来，当然，其也可以直接在目录中搜索所有的文件，并把其中符合条件的行打印出来。

```
# cat test.log | grep hello
hello
# grep hello test.log
hello
# grep hello . -r
./test.log:hello
```

上面就是三种方式搜索包括`hello`关键字的行。

`grep`是Linux中使用最为频繁的命令之一，其本身也有非常强大的功能，这一节咱们将详细讲述其比较常见的用法。

## 基本查找

查看`grep`的帮助可以看到：

```
# grep --help
Usage: grep [OPTION]... PATTERN [FILE]...
```

其中`OPTION`指的是命令参数，`PATTERN`指的是匹配的字符串，比如关键字搜索。`FILE`指的是文件，当然，没有文件的时候也可以通过管道接收数据并搜索过滤。`grep`提供了非常多的命令参数用来控制查找的方式跟效果，下面列举其常用的一些参数：

|------+------------------------------------------------------------------------------------------------------------|
| 参数 | 说明                                                                                                       |
|------+------------------------------------------------------------------------------------------------------------|
| -i   | 默认情况下，`grep`命令的搜索是大小写敏感的，如果需要忽略大小写可以用这个参数                               |
| -v   | 该参数表示不包含的意思                                                                                     |
| -A2  | 其中`A`是`after`的意思，表示同时显示搜索出来行后面两行。有时候我们需要知道匹配行后面是什么，可以用这个参数 |
| -B2  | 与`A`相反，其是`before`的意思，表示同时显示匹配行前两行                                                    |
| -C2  | 有时候我们想既显示前面两行也显示后面两行，这时候就用这个参数                                               |
| -r   | 搜索目录的时候需要带上这个参数                                                                             |
| -n   | 有些时候，我们需要知道匹配到的行是第几行，可以加上这个参数把行号打印出来                                   |
|------+------------------------------------------------------------------------------------------------------------|

对于`PATTERN`，`grep`命令也支持不同的用法：
* 关键字匹配

这个是最常用的基本用法，匹配是否包括该关键字的行。

* 正则匹配

除了基本的关键字匹配，`grep`还支持极为强大的正则表达式的匹配。我们将在下一小节专门讲述正则表达式匹配。

* 或匹配

有时候我们需要匹配某个PATTERN1或者PATTERN2的行，这时候可以这么写`PATTERN`：`PATTERN1\|PATTERN2`。通过`` \| ``将多个`PATTERN`连在一起表示或的意思，只要匹配其中任一个的行都会被打印出来。

```
# grep '12306\|test' . -r
./test2:http://abcdefg.test.com
./test2:12306.com
```

## 正则表达式匹配

`grep`命令支持非常强大的正则匹配，支持三种不同的正则表达式：

* [ERE](https://en.wikibooks.org/wiki/Regular_Expressions/POSIX-Extended_Regular_Expressions)(POSIX-Extended Regular Expressions)
* [BRE](https://en.wikibooks.org/wiki/Regular_Expressions/POSIX_Basic_Regular_Expressions)(POSIX_Basic_Regular_Expressions)
* [Perl Regular expression](https://perldoc.perl.org/perlre.html)

当然，比较常用的是`ERE`正则表达式了。在`grep`命令中，采用`-E`参数即可以使用该表达式，正则表达式非常灵活，用法非常多，这里列举几个示例：

* 匹配以关键字开头的行

    ```shell
    grep -E "^keyword" . -r
    ```
* 匹配以关键字结尾的行

    ```shell
    grep -E "keyword$" . -r
    ```
* 匹配包含如`2017/10/11`日期的行

    ```shell
    grep -E "[0-9]{4}/[0-9]{2}/[0-9]{2}" . -r
    ```

    注意的是，对于数字的表示，其不支持如`\d`这样的表达方式，而需要`[0-9]`这样表达。

当然，如果需要熟练掌握`grep`的正则表达式匹配，你必须对正则表达式非常熟悉，这个就不在本篇的范畴了。

# awk命令

要说Linux中最为强大的基本命令有哪些，那`awk`无疑会榜上有名，其强大的流式处理能力，很多人甚至写书来专门讲这个命令。这里咱们暂时介绍基本的功能，以后有机会将专门开辟一文来展开。

一个常见的用法就是把文件中的某几列打印出来：

```
# cat test.log
Nov 17 00:27:20 traffic-base1 named[1212]: managed-keys-zone: Unable to fetch DNSKEY set .: timed out
# cat test.log | awk '{print $1, $2, $3}'
Nov 17 00:27:20
```

如上例，空格将一行分成了不同的列，咱们希望只把时间显示出来，而时间包括了第1，2，3列，因此，通过`awk '{print $1, $2, $3}'`就实现了此功能，其中`$1`就是引用第一列。

# cut命令

有时候，一行内的数据并不是通过空格分隔开的，而是通过其它分隔符，那如何显示想要的列呢？通过`cut`命令，咱们同样可以轻松实现：

```
# head -n 5 /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
# head -n 5 /etc/passwd | cut -d: -f1
root
bin
daemon
adm
lp
```

`/etc/passwd`中的每一行基本上都是通过`:`进行分隔，其中第一列表示用户名，如果咱们只想把用户名输出的话，可以通过`cut -d: -f1`来实现，其中`-d:`表示以`:`号为分隔符，`-f1`表示显示第一列。可见，对于`cut`命令来说，咱们可以指定分隔符，那么前面通过`awk`实现的例子也可以通过`cut`来做：

```
# cat test.log
Nov 17 00:27:20 traffic-base1 named[1212]: managed-keys-zone: Unable to fetch DNSKEY set .: timed out
# cat test.log | cut -d" " -f1-3
Nov 17 00:27:20
```

其中`-d" "`表示以空格为分隔符（注意，这里的空格必须以引号括起来，不然会被shell展开去除多余空格），`-f1-3`表示输出第1到3列，这里简用了`-`来表示范围，当然也可以写成`-f1,2,3`了。

此外，`cut`命令还可以指定输出哪些位的字符：

```
# cat test.log
Nov 17 00:27:20 traffic-base1 named[1212]: managed-keys-zone: Unable to fetch DNSKEY set .: timed out
# cat test.log | cut -c1-16
Nov 17 00:27:20
```

其中，`-c1-16`表示输出第1到16个字符，当然，你同样可以以`,`来分别列举要输出哪几个。

# tr命令

`tr`其实是`translate`的缩写，这个命令用来将某些字符翻译成另外的字符。

```
# tr --help
Usage: tr [OPTION]... SET1 [SET2]
```

这个命令就是把字符集`SET1`中的字符对应的转成字符集`SET2`中的字符。如将小写转成大写：

```
# cat test.log
Nov 17 00:27:20 traffic-base1 named[1212]: managed-keys-zone: Unable to fetch DNSKEY set .: timed out
# cat test.log | tr a-z A-Z
NOV 17 00:27:20 TRAFFIC-BASE1 NAMED[1212]: MANAGED-KEYS-ZONE: UNABLE TO FETCH DNSKEY SET .: TIMED OUT
```

这里用`-`来方便的表示一定范围的字符集，当然，你完全可以一个个列出来你要的字符集。通常情况下，`SET1`跟`SET2`的长度保持一致，因为这个转换实际上是一对一的转换，当然，`SET2`的长度是可以大于`SET1`的，多余的字符不会被使用。但是，当`SET2`长度小于`SET1`时，`tr`命令会将`SET2`中最后一个字符填充不足的位数：

```
# echo 'abcdefg' | tr ab wyz
wycdefg
# echo 'abcdefg' | tr abcdefg wyz
wyzzzzz
```

对于`tr`还有一个常见的用途，就是用来去除字符串中的换行符：

```
# head -n 5 /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
# head -n 5 /etc/passwd | tr '\n' ' '
root:x:0:0:root:/root:/bin/bash bin:x:1:1:bin:/bin:/sbin/nologin daemon:x:2:2:daemon:/sbin:/sbin/nologin adm:x:3:4:adm:/var/adm:/sbin/nologin lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

其中，`` \n ``表示换行符，示例中将换行符全部换成了空格。

最后，`tr`还有一个常见的用法，可以加上`-d`参数来删除字符集中的字符。

```
# echo abcdeWXYZ | tr -d a-z
WXYZ
```

示例中删除了所有小写字母。

# sed命令

流式处理中，`sed`也是一个极为常用的命令，它可以用来替换字串，比之前的`tr`那是要强大无数倍了。

查看`sed`的帮助：

```
# sed --help
Usage: sed [OPTION]... {script-only-if-no-other-script} [input-file]...
```

它既可以直接在文件中替换字符串，也可以加收管道的数据。如基本用法：

```
# echo abcdefgabcd | sed 's/abc/ABC/'
ABCdefgabcd
```

其中`'s/abc/ABC/'`指定了替换的规则，默认情况下只替换一次，如果需要全部替换，则需要在规则后面加入`g`：

```
# echo abcdefgabcd | sed 's/abc/ABC/g'
ABCdefgABCd
```

规则中默认使用了`/`为分隔。当然，你也可以使用其他分隔符，这在要替换的字串中带有`/`的时候特别有用：

```
# echo "http://abc.test.com" | sed 's/http:\/\//https:\/\//'
https://abc.test.com
# echo "http://abc.test.com" | sed 's|http://|https://|'
https://abc.test.com
```

如果不指定新的分隔符`|`，那么就得使用转义符`` \ ``将`` // ``进行转义了，这样可读性就差了很多，采用`|`就自然多了。`sed`支持的分隔符还包括了`:`，`_`。

此外，`sed`不仅可以用来替换字串，还可以用来删除匹配的行：

```
# cat test2
http://abcdefg.test.com
12306.com
# cat test2 | sed '/12306/d'
http://abcdefg.test.com
```

## 高级用法

* 后向引用（Back Referencing）

    `sed`规则匹配到的字符串还可以在规则定义中被引用，如下例：

    ```
    # echo "hello world" | sed 's/hello/&&/'
    hellohello world
    ```

    其中，`&`可以用来引用被匹配到的字串，在本例中，匹配到的字串是`hello`，这样通过`&`就可以引用被匹配到的字串了。此外，我们还可以通过`()`来指定匹配的字串，并用`\1`（数字表示第一个`()`）来引用（实际上是正则表达式中的Grouping）：

    ```
    # echo Sunday | sed 's/\(Sun\)/\1ny/'
    Sunnyday
    # echo Sunday | sed 's/\(Sun\)/\1ny \1/'
    Sunny Sunday
    ```

* 正则匹配

    由上述可以看出，`sed`的规则使用了正则表达式规则，但是其书写跟一般的正则书写不一样，你必须将有关的字符转义，否则`sed`仍然会将其当做普通字符进行匹配：

    ```
    # echo "this is aaaa cat" | sed 's/a{4}/a/'
    this is aaaa cat
    # echo "this is aaaa cat" | sed 's/a\{4\}/a/'
    this is a cat
    ```

    可以看出，在没有转义`{`及`}`之前，`sed`并没有匹配到目标字串`aaaa`，而将其转义之后，则以正则表达式`a{4}`匹配到了`aaaa`并进行了替换。

* 递归替换

    有时候我们需要在项目目录下替换某个字串，比如把手误写错的`#include<stdllib.h>`全部替换成`#include<stdlib.h>`，希望被替换的文件包括`*.cpp`，`*.h`的文件。其中的一种做法如下：

    ```
    $ head -n 1 test.cpp
    #include <stdllib.h>
    $ sed -i 's/#include <stdllib.h>/#include <stdlib.h>/' `find . -name "*.cpp" -o -name "*.h"`
    $ head -n 1 test.cpp
    #include <stdlib.h>
    ```

    这里用到了shell嵌入的用法（可以参看[这一篇博文](https://keysaim.github.io/2017/10/10/linux-study-shell-basic/#shell嵌入与shell选项))，通过`find`命令找出所有的源文件，然后用`sed -i`进行替换，其中`-i`表示从文件里面替换。这一用法会将`find`找到的所有文件作为参数都追加到`sed`命令后，在项目非常大的情况下可能会导致命令执行失败（因为数量庞大的文件导致追加的参数太大了），通常我们推荐采用`find`的这种用法：

    ```sh
    find . -name "*.cpp" -o -name "*.h" -exec sed -i 's/#include <stdllib.h>/#include <stdlib.h>/' {} \;
    ```

    具体可以参照本文中关于`find`命令的介绍。


