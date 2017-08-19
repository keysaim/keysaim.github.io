---
layout:     post
title:      "Linux学习从入门到精通推荐书籍"
subtitle:   "从入门到精通"
date:       2017-08-18
author:     "keysaim"
header-img: ""
catalog:    true
tags:
    - linux
    - 推荐
    - 书籍
    - 命令行
    - 脚本
    - 系统管理
    - sysadmin
    - 网络
    - 安全
---

# 如何学习Linux

在现在的生活，生产，研究等领域，Linux已经无所不在，从我们使用的手机，车载设备，到服务器，桌面电脑等，Linux已经成为这个世界方方面面的基石。尤其对于参与技术有关工作的你学习Linux是必须的，那么，该如何有效的学习呢？Linux从诞生至今，已经是一个非常庞大且复杂的系统，下图是Linux系统代码行数的统计（参考[linuxcounter](https://www.linuxcounter.net/statistics/kernel)）：
![Linux代码行数变化](/img/blog-linux-code-lines.png)

可见截止本文为止，Linux的代码行数逼近2千万行，你就每天看1万行也得要6，7年，况且这还只是Linux内核的代码量，再加上每个Linux发行版本有关的代码，估计得突破天际了。因此，要在短期内全面的学习Linux的方方面面对于一个正常的人来说几乎不太可能。所以，学习Linux的关键便在于对于学习Linux的目的一定要明确，通常来讲可能会涉及到一下方面：

* 了解及入门
* 成为Linux的系统管理员
* 学习Linux应用编程
* 学习Linux内核开发

当然，这只是其中几个大的方面，即便如此，其中每个方面都是一个非常大的议题。比如说`学习Linux内核开发`，这个就包括了无数小的方面，内核本身就包括了非常多的细分方向，比如有的搞网络，有的搞文件系统，有的搞驱动开发等等。所以，对于Linux，还请千万慎重的评价自己是否真的`精通`，学无止境，真要`精通`Linux的主要方面，有可能需要穷尽你的个人生涯。

当然，对于学习Linux，前人已经铺好了无数的基石，有无数可以参考学习的资料，而且内核也是开源的，必要的时候可以查看[其代码](https://github.com/torvalds/linux)，甚至已经有非常多帮你分析内核代码的[书籍资料](https://www.quora.com/What-are-the-best-resources-to-learn-about-Linux-kernel-internals)，甚至还有很多[中文资料](https://www.zhihu.com/question/19606660)。所以，不论你打算要学习到如何的程度，已经有无数的资料可以参考，也有极为庞大的[社区](http://www.linuxandubuntu.com/home/top-10-communities-to-help-you-learn-linux)可以依靠。本文将就Linux学习推荐一些经典免费的书籍，主要侧重覆盖从入门到成为系统管理员的有关方面，学习对象为初学Linux，以及需要重新系统学习Linux的读者，将涵盖以下方面：

* 入门基础
* Linux命令行及工具
* Linux Bash脚本
* Linux发行版本
* Linux系统管理
* Linux基本开发

> 为啥需要重点学习系统管理方面呢？有的Linux开发人员可能会说，不是有专门的Linux系统管理员吗，有必要花大力气学习系统管理吗？这个博主表示是非常有必要，开发可能侧重于功能的实现，而且往往侧重于细节，然而系统管理则直接面向功能本身，更多的是从整个系统的宏观角度来熟悉Linux。咱们有句话说`不识庐山真面目，只缘身在此山中`便是这个道理，开发者对于细节或许极为了解，但是未必对整个系统功能有足够的熟悉。而如果对于宏观的系统整体有足够的理解，对于开发本身来说也是有很大的促进作用的。

# 书籍推荐

## 入门基础书籍

### Introduction to Linux

这是一本免费的书，来自于[Linux文档项目](Linux_Doc_Proj)。虽然免费，但是不影响它的流行程度，该书比较系统的介绍了Linux的一些基本概念，包括文件系统，命令行，网络等。但是鉴于Linux现在也是版本帝，有些内容可能跟不上最新的版本，但是，这完全不影响对于基本概念的理解。

### Linux Fundamentals

从这本书的书名就可以看出，作者[Paul Cobbaut](http://www.linkedin.com/in/cobbaut)就是侧重于介绍Linux最基础的有关知识。涉及到Linux的历史，如何安装以及一些简单但是常用的命令。

## Linux命令行及工具书籍

### GNU/Linux Command−Line Tools Summary

这本书同样来自于[Linux文档项目](Linux_Doc_Proj)。适于初学Linux命令行的读者。

### Bash Reference Manual from GNU

此书来自于[GNU](https://www.gnu.org/home.en.html)，着重介绍Linux命令行。

### The Linux Command Line

如果你把前面的几本基本的命令行的书籍啃完，并迫切希望能够进一步深入了解命令行，那么这本出自[William Shotts](http://www.oreilly.com/pub/au/4962)的书是必须一读的，此书500多页的篇幅，极为详尽的介绍了Linux命令行，也许你自诩比较熟悉命令行，相信此书还是能够带个你新的见识。

## Linux Bash脚本书籍

### Bash Beginners Guide

顾名思义，此书就是为初学者准备的，同样来自于[Linux文档项目](Linux_Doc_Proj)。

### Advanced Bash-Scripting Guide

如果你对Linux Bash脚本有了基本的认识，那么这本书将是你进阶的必备书籍。此书900多页的篇幅涉及Bash脚本的方方面面，不论对于打算进阶或者已经较为熟悉的人来说都是一本重要的参考书籍。

### The AWK Programming Language

[AWK](https://www.tutorialspoint.com/unix_commands/awk.htm)命令是一个极为强大的Linux命令，同时提供非常强大的脚本支持。也正是因为强大，所以就有专门的书籍来介绍这个命令，如果你要把自己的Linux命令再提升一点的话，建议看下这本书来深入的学习该命令。

### Linux 101 Hacks

不论从这本书的书名，还是这本书的来源[The Geek Stuff](http://www.thegeekstuff.com/2009/02/linux-101-hacks-download-free-ebook/)，此书都暗示着其将以新颖独特的角度为你介绍Linux脚本。

## Linux发行版本书籍

### CentOS System Administration Essentials

这本书较为系统的介绍了Centos系统的有关知识，包括了文件系统，包管理系统，用户系统，安全中心以及一些常用应用软件介绍，对于使用Centos系统的人员还是有所帮助的。

### Ubuntu Manual

这本书来源于[Ubuntu Manual网站](https://ubuntu-manual.org/)，以不多的篇幅较为系统的介绍了Ubuntu系统的日常使用。

### For Linux Mint: Just Tell Me Damnit!

这本书集中介绍了[Linux Mint](https://www.linuxmint.com/)系统，涉及了安装，包管理，定制桌面等方面。

### Solus Linux Manual

顾名思义，此书介绍[Solus Linux](https://solus-project.com/)系统，篇幅较短。

### The Debian Administration’s Handbook

这本书号称[Debian Linux](https://www.debian.org/)系统的圣经，涵盖了Debian的历史，安装，包管理，虚拟机，存储等方面，对于使用Debian系统的人员来说，此书必备。


## Linux系统管理书籍

虽然此章节被独立命名为`Linux系统管理`，但是，前面的章节其实都可以认为在此范畴，只不过更加偏向于基础。所以，在这章节中都是侧重于较为深入的系统管理有关知识，最好是在前面章节的基础之上再学习此章节。

### Linux System Administration

这本书也是出自于[Paul Cobbaut](http://www.linkedin.com/in/cobbaut)之手，覆盖了网络，磁盘，用户，内核，库等管理。

### Advanced Linux System Administration

如果你觉得自己很懂Linux系统管理，也非常希望别人能够知道你很懂，那么你应该去参加[LPIC](LPIC)。而要参加该认证，此书是必看的官方指定用书。

### Pro Linux System Administration, 2nd Edition

这是一本非常详细的Linux系统管理的书籍，全书1000+页的篇幅涵盖了Linux系统管理的很多方面，即适合初学者，也可以作为有一定基础的人系统学习的重要参考。同时，该书的第二部分介绍了很多Linux系统管理的应用，如NTP，DNS，邮件，文件共享，性能监控等等，非常值得一看。

### Linux Bible 9th Edition

不用讲了，敢取这么牛逼的名字，而且书的[评价还不错](https://www.amazon.com/Linux-Bible-Christopher-Negus/dp/1118999878)，必然是好书。此书将近1000页的篇幅，从不同程度介绍了Linux系统，比如如何入门，如何成为Linux的熟练用户，如果成为系统管理员，如何成为Linux安全维护人员等等，还是比较实至名归的。

### Linux Servers

此书又来自于[Paul Cobbaut](http://www.linkedin.com/in/cobbaut)，从书名就能推测其范畴，主要侧重讲述如何打造你的Linux服务器，包括web server，mysql数据库，DHCP等。

### Linux Networking

Linux网络对于系统管理员来说是最为重要的一块之一，同样出自于`Paul Cobbaut`之手，较为系统的介绍了Linux网络基础知识，网络配置，同时着重介绍了常用的网络服务等。

### Linux Storage

此书作者估计你都能猜到了，不错，又是`Paul Cobbaut`。该书同样较为系统的介绍了Linux的存储系统，涉及文件管理，磁盘管理，数据库等方面。

### Linux Security

作者就不介绍了，你懂的。很多时候对于Linux系统都更侧重于功能方面，对于安全方面往往做的不够。然而现在网络安全正面临越来越严峻的挑战，由网络安全带来的损失也是越来越大，因此，对于Linux的安全管理已经成为系统管理最为重要的一部分。此书同样系统的介绍了Linux的安全管理有关方面，涉及用户/组安全，文件安全，iptables防火墙，selinux安全等方面。

## Linux基本开发书籍

最后，稍微推荐一下Linux开发有关的书籍。

### Advanced Linux Programming

此书面向致力于Linux软件开发人员，介绍了Linux多进程，多线程，进程间通信，以及硬件接口等方面，对于从事有关开发工作还是很有帮助的。

# 书籍下载

本博文中所列书籍都可以在博主的[CSDN个人下载空间](http://download.csdn.net/user/walkerhau)找到，资源名为`Linux入门及系统管理推荐书籍`，由于大小限制，分为三个压缩包，下载所有压缩包到一台Linux机器，解压运行命令：

```sh
cat linux-basic.tgz.* | tar xz
```

其中有一个文件整理的时候出错了，文件名为`Ubuntu-Manual.pdf`，其实是`Solus`的电子书。你可以从[Ubuntu Manual官网](https://ubuntu-manual.org/)直接下载。

资源需要一定的资源分下载，本来想免费，但是博主个人觉得还是不错的资源，也废了自己不少时间整理，鉴于免费容易轻视，所以面向真要打算认真学习的人，收取一定的资源分。如果没有足够的资源分也没关系，你完全可以根据本文所列书名Google之，都是有免费电子版的。实在不愿自己搜罗的，也可以在评论区留下你的邮箱，博主会不定时发给你邮箱，压缩包总共将近140M，还请确保你的邮箱能够接收如此大的附件。

# 结语

Linux博大精深，很多人即便从事一辈子Linux开发也未必能够熟悉Linux的各个方面。为了能够支撑自己在Linux的路上走的足够远，一个牢固的基础是必须得有的，本文侧重推荐Linux系统管理的有关书籍，希望读者能够对Linux的宏观整体有个非常透彻的理解，为以后选择某个方向深入研究铺好路。同时，也欢迎各位的其它推荐，欢迎在评论区留言，有合适的书籍，博主也会不定时更新在博文之中。

# 参考

* [Linux Document Project](Linux_Doc_Proj)
* [Ubuntu Manual](https://ubuntu-manual.org/)
* [LPIC wikipedia](LPIC)
* [learn linux for free](https://itsfoss.com/learn-linux-for-free/)

[Linux_Doc_Proj]: http://www.tldp.org/index.html
[LPIC]: https://en.wikipedia.org/wiki/Linux_Professional_Institute_Certification
