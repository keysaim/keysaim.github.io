---
layout:     post
title:      "如何使用特定的SSH Key提交GIT"
subtitle:   ""
date:       2017-08-15
author:     "keysaim"
header-img: ""
catalog: true
tags:
    - github
    - git
    - 教程
---

# 问题提出

最近在自己的MAC上面提交`Github`代码的时候发现居然失败了：

```console
$ git push origin master
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

这不是坑爹吗，`Github`都提交过无数次了，咋就失败了呢？莫非`Github`上的ssh key被删掉了么。于是打开[github ssh](https://github.com/settings/keys)，尝试再次把ssh key加上，却提示key已经存在了。于是赶紧回到本地repo查看下用户是不是对的:

```console
$ git config -l
...
user.email=keysaim@gmail.com
user.name=keysaim
```

再查看下本地的ssh key：

```console
$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCu4Jy/+uFGiC89luBejzCEyPbY0SRoppyzrB4g1v3zv1OleylMzdf+eTTRcYgMbYoY6ZQs4M2NHX20iO6vf6j2uPvUsB++pP0G6Q7+VlrUlC19B07IVx7Mo2xmHCe4bMshFSugqOl+hV6zVjGpYJcLI9XtWQ6F/br4tkYD/J8KWns+SNha8gJVBckV1ncGlR+Q7ji4OM4+eIhKEEK4Wo7Cf7KaT71fIVFl7XRx5kmdtEN3F+wT4LjNb2okl8Pu4mmxCMwJvXzj0Jr9PkVzhSAhDkWG3mMt3kC5PhhRhCP7uwkGFsOEm5uGS907wTxY9cJNIl8FikOfmvDa5XrfMbMx nbaoping@xxx.com
```

发现邮件居然是`nbaoping@xxx.com`（此处已打码），显然跟本地repo的`keysaim@gmail.com`不同，`git`提交的时候没有特殊配置，会使用默认的ssh key，也就是`~/.ssh/id_rsa.pub`，而提交的用户信息跟此key并不能对应上，故此`github`拒绝了此次提交。既然如此，那把本地repo的用户信息改成key所对应的信息不就好了吗？是的，但是此信息都已经打码了就充分说明本博主是十分不愿暴露它的，咱必须得想其它辙。

好了，现在的问题就是，如何使用特定的ssh key提交Git？本文就来讲述一种通用的解决办法。

# 指定`git`提交使用的ssh key

* 查看repo对应的hostname

    ```console
    $ git remote -v
    origin  git@github.com:keysaim/keysaim.github.io.git (fetch)
    origin  git@github.com:keysaim/keysaim.github.io.git (push)
    ```

    其中`github.com`就是repo使用的`hostname`。

* 查看repo的用户信息

    ```console
    $ git config -l
    ...
    user.email=keysaim@gmail.com
    user.name=keysaim
    ```

    最关键的是邮件信息`keysaim@gmail.com`。如果没有用户信息，可以先配置：

    ```console
    $ git config user.email "keysaim@gmail.com"
    $ git config user.name "keysaim"
    ```

    注意，很多教程里面以及`git`的错误提示里面会建议在`git config`后面加入参数`git config --global`，这里，千万不要加入此参数，否则它会去尝试修改你的`git`的全局配置，也就是你所有repo默认的用户信息。你可以在文件`~/.git/config`查看你的全局配置，其中`[user]`段就是你的默认用户信息。咱们这里就是为了能够给这个repo指定特定的ssh key，显然不适合使用全局的配置。

* 为repo的用户生成新的ssh key

    ```console
    $ ssh-keygen -C "keysaim@gmail.com"
    Generating public/private rsa key pair.
    Enter file in which to save the key (/Users/nbaoping/.ssh/id_rsa): id_rsa.github
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in id_rsa.github.
    Your public key has been saved in id_rsa.github.pub.
    The key fingerprint is:
    SHA256:G0djI0bh+XwGcwOZ0AsbQ8ffB51pYrfSlRALNZw3igc keysaim@gmail.com
    The key's randomart image is:
    +---[RSA 2048]----+
    |       .==o+o=+.+|
    |       o+o=E.=+Oo|
    |        ==Bo*oB.+|
    |       ..*.B.=.o.|
    |        S + + .. |
    |         + o     |
    |        .        |
    |                 |
    |                 |
    +----[SHA256]-----+
    ```

    其中，`-C`是用来指定该key的用户信息的，这里咱们使用了`keysaim@gmail.com`。该命令是一个交互式的命令，其中大部分你都可以直接回车，但是对于第一个提示`Enter file in which to save the key`，***请务必输入你想要的文件名，否则它将覆盖你默认的ssh key，这个可是不可逆的***。这里使用文件名`id_rsa.github`。如果没有指定文件夹在路径中，该命令会在当前目录下生成key文件：

    ```console
    $ ls id_rsa.github*
    id_rsa.github  id_rsa.github.pub
    ```

    其中`id_rsa.github`是私钥，而`id_rsa.github.pub`为公钥。将key文件移到ssh目录下`~/.ssh/`：

    ```console
    $ mv id_rsa.github* ~/.ssh/
    ```

* 配置ssh以使用新的key

    修改ssh的配置文件`~/.ssh/config`，加入如下配置：

    ```
	Host github.com
		HostName github.com
		User git
		IdentityFile /Users/nbaoping/.ssh/id_rsa.github
		IdentitiesOnly yes
    ```

    下面逐行解释：

    * `Host github.com`

        用来指定该key的Host名字，此处必须使用本地repo的hostname `github.com`。

    * `Hostname github.com`

        此处指定`Host`对应的具体域名，这里跟`Host`保持一致。（`Host`跟`Hostname`可以不一致，但是`Host`必须跟repo的hostname保持一致，也就是git到时候会用自己repo的hostname来ssh配置文件里面找是不是有对应的`Host`，找到了就使用该配置，具体访问的域名会采用`HostName`）

    * `User git`

        说明该配置的用户得是git

    * `IdentityFile /Users/nbaoping/.ssh/id_rsa.github`

        这行最为关键，指定了该使用哪个ssh key文件，这里的key文件一定指的是私钥文件。之前我们生成了新的私钥文件`~/.ssh/id_rsa.github`，由于博主使用的是MAC，`~`被翻译成`/Users/nbaoping/`了，如果是在一般的Linux环境下，改路径前缀该是`/home/nbaoping/`。

    * `IdentitiesOnly yes`

        请配置为`yes`，具体意义可以[参考讨论](https://serverfault.com/questions/450796/how-could-i-stop-ssh-offering-a-wrong-key/450807#450807)。

* 将生成的ssh key加入`github`

    打开[github ssh key](https://github.com/settings/keys)配置页面，点击`New SSH Key`，给刚刚生成的key取名，如`keysaim-mac`。把`~/.ssh/id_rsa.github.pub`（请务必注意是公钥文件，千万不要搞错了）里面的内容拷贝过来，点击`Add SSH Key`按钮保持。

* 提交

    做完上面的步骤之后，就可以提交了：

    ```console
    $ git push origin master
    Counting objects: 63, done.
    Delta compression using up to 8 threads.
    Compressing objects: 100% (62/62), done.
    Writing objects: 100% (63/63), 838.96 KiB | 0 bytes/s, done.
    Total 63 (delta 2), reused 0 (delta 0)
    remote: Resolving deltas: 100% (2/2), done.
    To github.com:keysaim/keysaim.github.io.git
       73a2043..88cacc1  master -> master
    ```

    可以看到，这次提交成功了。

# 结语

当你需要把某些repo以不同的用户提交的时候，可以按照本文给他们配置特殊的ssh key，但是注意的一点就是，这种配置事基于`Host`，也就是repo的hostname，如果需要确保不同的repo使用不同的ssh key，需要每个repo使用不同的hostname。

# 参考

* [Specify an SSH key for git push for a given domain](https://stackoverflow.com/questions/7927750/specify-an-ssh-key-for-git-push-for-a-given-domain)

