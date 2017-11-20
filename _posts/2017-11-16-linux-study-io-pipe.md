---
layout:     post
title:      "Linux系统介绍（四）IO重定向与管道"
subtitle:   ""
date:       2017-11-16
author:     "keysaim"
header-img: ""
catalog: true
tags:
    - Linux
    - study
    - 学习
---

# IO重定向(IO redirection)

Linux的有一个强大之处就是可以通过管道(Pipe)跟IO重定向将一系列命令的输出跟输入连接起来。IO重定向是Linux中非常重要的概念，是理解Linux命令，脚本以及Linux IO的基础。

## 标准输入输出

对于`shell`来说，有三个基础的流，标准输入流(stdin或者stream 0)，标准输出流(stdout或者stream 1)，标准错误流(stderr或者stream2)。

![标准输入输出](/img/blog-linux-shell-stdinout.png)

举个例子，当我们用键盘在`shell`中执行命令的时候，可以如下图：

![键盘输入输出](/img/blog-linux-shell-keyboard-stdinout.png)

通常，stdout跟stderr都输出到了屏幕上，但对于Linux来说，其实是两种不同的输出。

## 输出重定向

可以用`>`大于号将stdout重定向到另一个IO，比如文件：

```
# echo "hello" > test.log
# cat test.log
hello
```

上面的命令将stdout重定向到文件`test.log`中，此时，如果该文件不存在则创建新文件，如果存在则覆盖已有文件。事实上，`>`重定向是`1>`的简写，`1>`可以更清楚的看到实际上是把stdout(stream 1)重定向。

必须注意的是，默认情况下，该重定向会覆盖已有文件，这个在有时候可能不经意间丢失重要数据。`shell`提供了选项使得我们可以禁止这种覆盖，`set -o noclobber`可以打开该选项。

```
# cat test.log
hello
# set -o noclobber
# echo "world" > test.log
-bash: test.log: cannot overwrite existing file
```

此外，在打开该选项之后，其实还是可以强制执行覆盖，可以采用`>|`来强制重定向到已存在的文件：

```
# echo "world" > test.log
-bash: test.log: cannot overwrite existing file
# echo "world" >| test.log
# cat test.log
world
```

## 追加输出

可以采用`>>`将输出重定向到文件并追加在文件结尾，这样就可以避免覆盖文件了。

```
# cat test.log
world
# echo hello >> test.log
# cat test.log
world
hello
```

## 标准错误重定向

如`1>`一样，我们可以通过`2>`将stderr重定向到文件，具体行为跟stdout类似。

## 同时重定向stdout跟stderr

我们可以在同一行命令中同时将stdout跟stderr重定向，如：

```
# ls test* tttt*
ls: cannot access tttt*: No such file or directory
test.log  test2
# ls test* tttt* > stdout.log 2> stderr.log
# cat stdout.log
test.log
test2
# cat stderr.log
ls: cannot access tttt*: No such file or directory
```

可以看出，stdout跟stderr被分别重定向到`stdout.log`跟`stderr.log`文件中了。

此外，还有一个常见的用法是将stderr重定向到stdout，这样就可以将所有输出都定向在一起了。

```
# ls test* tttt* > stdout.log
ls: cannot access tttt*: No such file or directory
# cat stdout.log
test.log
test2
# ls test* tttt* > stdout.log 2>&1
# cat stdout.log
ls: cannot access tttt*: No such file or directory
test.log
test2
```

可见，通过`2>&1`将stderr重定向给stdout，而stdout又重定向给文件`stdout.log`，这样所有的输出都重定向到文件`stdout.log`中了。另外，还可以通过`&>`直接将stderr跟stdout合并：

```
# ls -l test* tttt* &> stdout.log
# cat stdout.log
ls: cannot access tttt*: No such file or directory
-rw-r--r--. 1 root root 12 Nov 16 01:02 test.log
-rw-r--r--. 1 root root  0 Nov 16 00:17 test2
```

## 重定向顺序

将stderr重定向给stdout的时候，请务必注意其顺序，如上面的重定向如果写成这样，结果就完全不同了：

```
# ls test* tttt* 2>&1 > stdout.log
ls: cannot access tttt*: No such file or directory
# cat stdout.log
test.log
test2
```

可以看出，stderr其实并没有被重定向到文件`stdout.log`中，可见顺序是非常重要的。那么，如果理解这种不同呢？咱们可以这么样来理解：
* 将`>`看作是`shell`中的赋值操作`=`
* 将stdout跟stderr看作是变量，但对其引用采用`&`，这样`&1`表示对stdout变量的引用
* 假定stdout跟stderr变量的初始值是屏幕，将屏幕记为`/dev/tty`
* `shell`从左到有扫描解释命令，并对stdout跟stderr分别赋值
* 查看stdout跟stderr的最终值即可知道分别被重定向到哪里了

还是以上面的例子来解释，`ls test* tttt* 2>&1 > stdout.log`
* 命令开始前，stdout=/dev/tty, stderr=/dev/tty
* `shell`从左到右扫描并重新赋值，首先`2>&1`就相当于`stderr=$stdin`，此时`stderr`的值其实还是`/dev/tty`
* `> stdout.log`就相当于`stdout=stdout.log`，此时stdout值为`stdout.log`
* 最后，stdout值为`stdout.log`，而stderr值仍然为`/dev/tty`，所以只有stdout输出到文件`stdout.log`中了

基于这个原则，在讲述完管道之后咱们将展示如何把stdout跟stderr交换一下。

## 输入重定向

* `<`符号

既然输出有重定向，那么输入是否也可以呢？答案是肯定的，可以采用`<`将输入重定向，`<`其实是`0<`的简写。

```
# cat stdout.log
ls: cannot access tttt*: No such file or directory
-rw-r--r--. 1 root root 12 Nov 16 01:02 test.log
-rw-r--r--. 1 root root  0 Nov 16 00:17 test2
# cat <stdout.log
ls: cannot access tttt*: No such file or directory
-rw-r--r--. 1 root root 12 Nov 16 01:02 test.log
-rw-r--r--. 1 root root  0 Nov 16 00:17 test2
```

* `<<`符号

此外，还可以`<<EOF`通过手动输入直到输入`EOF`（或者Ctrl-D）。

* `<<<`符号

该符号可以直接将一个字符串重定向给输入

```
# base64 <<< hello
aGVsbG8K
```

`base64`命令参数只接受文件，通过这种方式就可以把字符串直接传给它。

## 输入输出同时重定向

`shell`是可以支持同时重定向输入跟输出的，以下方式都会被准确解析：

```
# cat <test.log > stdout.log 2> stderr.log
# <test.log > stdout.log 2> stderr.log cat
```

## 快速清除文件内容

可以通过重定向快速的清空文件内容：

```
# cat test.log
hello world
# > test.log
# cat test.log
#
```

可见，咱们并不需要写`echo "" > test.log`这样的命令来清空一个文件。当`noclobber`选项被打开时，可以通过`>|`来强制清空。

# 管道(Pipe)

在Linux中，我们可以使用管道(Pipe)将前一个命令的stdout作为输入给后面一个命令，管道由`|`表示。

```
# ls test* tttt*
ls: cannot access tttt*: No such file or directory
test.log  test2
# ls -l test* tttt* | grep log
ls: cannot access tttt*: No such file or directory
-rw-r--r--. 1 root root 12 Nov 16 01:02 test.log
```

请务必注意的是，管道只会将stdout传递给下一个命令，stderr并不会传递，为了证明这一点，咱们将后一个命令的stderr重定向到文件：

```
# ls -l test* tttt* | grep log 2> stderr.log
ls: cannot access tttt*: No such file or directory
-rw-r--r--. 1 root root 12 Nov 16 01:02 test.log
# cat stderr.log
# 
```

这时可以看出，第二个命令的stderr为空，而第一个命令的stderr仍输出到屏幕了。当然，咱们也可以将第一个命令的stderr重定向到stdout上，这样`grep`命令也可以收到了。

```
# ls -l test* tttt* 2>&1 | grep "No "
ls: cannot access tttt*: No such file or directory
```

再回到上一节的问题，咱们如何将stdout跟stderr互相交换一下呢？可以这么做：

```
# ls -l test* tttt* 3>&1 1>&2 2>&3 | grep "No " 2> stderr.log
ls: cannot access tttt*: No such file or directory
-rw-r--r--. 1 root root 12 Nov 16 01:02 test.log
-rw-r--r--. 1 root root  0 Nov 16 00:17 test2
# cat stderr.log
# 
```

如果你的Linux发行版本对grep输出的颜色设置正确，会发现只有第一行是grep出来的，由此可见`3>&1 1>&2 2>&3`居然将stdout跟stderr互换了一下，至于怎么解释，可以参照前面的赋值方式自行拆解一下。



