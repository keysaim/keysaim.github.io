---
layout:     post
title:      "用docker搭建全栈式应用（三）管理篇"
subtitle:   ""
date:       2016-06-23
author:     "keysaim"
header-img: ""
catalog: true
tags:
    - docker
    - 教程
---

# 用docker搭建全栈式应用 (三）—— 管理篇
## 简介
在[上一篇][previous blog]我们讲述了如何基于Dockerfile实现docker image的自动构建，并基于此重新实现了咱们本系列中的全栈式应用。然而，虽然构建过程实现了自动化，但是在实际部署的时候我们还是需要手动的理清所有服务的依赖关系，并依次启动所有服务，更严重的是，每次重启的时候，我们都需要重复这些操作，这个是不能接受的。

那么有啥办法改善么？记得业内似乎总有这么个说法，大凡超过重复操作必须得用脚本实现自动化，此谓IT男的基本素养。既然如此，咱们也得用脚本自动化咱们的操作了。非常直接的一种方案就是咱自己写个shell的脚本，如下：
* 启动脚本

    ```shell
    docker run -d --name redis-master redis-master
    docker run -d --name redis-slave1 --link redis-master:master redis-slave1
    docker run -d --name redis-slave2 --link redis-master:master redis-slave2
    docker run -d --name app2 --link redis-master:redis app 0.0.0.0:8002
    docker run -d --name haproxy --link app1:app1 --link app2:app2 -p 6301:6301 ha-app
    ```
    看起来还可以吗，简单的几条语句。但是，咱可不要忘了一点，每个语句咱还木有检查执行的结果。还有这个只是启动的脚本，还有停止的脚本，清除的脚本等等。更要命的是，如果下次我部署其它类型的应用，尼玛还得把这一些列的脚本复制并修改一通，这个有点弱爆了的感觉啊。

一般来讲，IT界总是存在这一的规律，当一个烦恼只是你一个人的时候，OK你可以随便整了；当你觉得可能会是很多人的烦恼的时候，这时必须得已经有人提供了解决方案了。所以，在当下技术更新如此剧烈的年代，切记不要随便就自己吭哧吭哧的闷声瞎搞了，第一要务就是google之。一般情况下，总是与大神提供了解决方案，如果真心没有，那么恭喜你，赶紧自己整一个并开放出来，那么你就是那个大神了。

同样，在咱们本文碰到的问题中，早就有对应的解决方案了，那就是基于Docker compose工具的服务管理。本文将详述如何基于Docker compose管理服务。当然，例子还是那个例子咯。

## Docker Compose工具简介
关于Docker Compose，[官方](https://docs.docker.com/compose/overview/)其实有非常详尽的介绍。这里就简单的总结一下。

那么，Docker Compose是什么呢？
* Docker的一个工具
* 定义，运行并管理多个容器应用
* 基于Compose文件
* 通常应用于开发、测试、CI环境

Compose的使用一般与Dockerfile结合，包括三个步骤：
* 定义好所有应用的Dockerfile
* 在Compose文件中定义、配置好所有服务
* `docker-compose up`启动你的服务

下面，咱们就以本系列的经典例子：全栈式应用，来讲述Docker Compose的使用。

## 基于Docker Compose搭建、管理全栈式应用
### Docker Compose的安装
安装过程其实非常简单，[官方](https://docs.docker.com/compose/install/)提供了非常详细的安装过程。这里讲述其中一种基于command line的安装方式。
* 使用curl下载Compose工具

    ```
    curl -L https://github.com/docker/compose/releases/download/1.7.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
    ```
    当然，这里使用的是1.7.1版本，如果有新版本发布了，你只需要修改成相应的版本即可。

* 添加Compose工具执行权限

    ```
    chmod +x /usr/local/bin/docker-compose
    ```

* 测试一下是否下载成功

    ```
    $ docker-compose --version
    docker-compose version: 1.7.1
    ```

### Compose文件的编写
前面我们讲了，一般来说，Compose的使用包括了三个步骤。由于在[上一篇][previous blog]已经编写好了Dockerfile文件，这里咱们就直接开始Compose文件的编写这一步骤了。Docker默认使用的compose文件名字为`docker-compose.yml`，这里还是老规矩，先列出本例子中使用的`docker-compose.yml`文件，然后再详细解释。

```
version: "2"
services:
    redis-master:
        build: master
        image: redis-master
        container_name: redis-master
        volumes:
            - /opt/demo/redis:/data

    redis-slave1:
        build: slave1
        image: redis-slave1
        container_name: redis-slave1
        links:
            - redis-master:master

    redis-slave2:
        build: slave2
        image: redis-slave2
        container_name: redis-slave2
        links:
            - redis-master:master

    app1:
        build: app
        image: app
        container_name: app1
        links:
            - redis-master:redis
        command: 0.0.0.0:8001

    app2:
        build: app
        image: app
        container_name: app2
        links:
            - redis-master:redis
        command: 0.0.0.0:8002

    haproxy:
        build: haproxy
        image: haapp
        container_name: haproxy
        links:
            - app1:app1
            - app2:app2
        ports:
            - 6301:6301
```

仔细瞅一下这个文件，其实还是蛮一目了然的，不过这里现在咱们开始一点点进行解释吧。

首先，必须注意到的是，docker compose文件其实采用的是常用的yaml文件格式进行定义，因此所有的定义必须符合yaml文件的规范。接下来分别解释每个yaml section。

* `version`

    顾名思义，这个表示当前的`docker-compose.yml`文件采用哪个版本的格式。这里采用了截止本文最新的版本2，至于跟版本1有啥不同，还请参考[官方文档](https://docs.docker.com/compose/compose-file/#versioning)。

* `services`

    `services`部分定义了需要管理的所有的服务，docker compose将根据这部分的定义对它们进行管理，包括所有service对应的containers的启动、停止等。

* `redis-master`

    这部分定义了`redis-master`的container。在[上一篇][previous blog]中，我们已经介绍了如何通过Dockerfile构建image，并手动启动了相应的container。这里，通过docker compose工具，我们可以将这两个过程都交给docker compose自动完成。这部分则对这两个过程进行了定义，包括如何构建以及如何启动。

    * `build`

        这部分定义了该服务构建时的目录，docker compose会首先查找本地cache是否以及有了所需要的image，如果没有，则根据这里提供的目录，找到对应的Dockerfile，然后构建出相应的image。所以，这部分其实是可选的。

    * `image`

        这部分定义了该服务所需要的image的名字，如前述，docker compose会先查找本地cache有没有image定义的名字对应的image，如果有则直接启动，如果没有则会试图构建，构建出来的image名字为这部分定义的名字。这部分必须定义。

    * `container_name`

        这部分定义了启动container之后，container的名字。这部分也是可选的，如果没有定义则会自动生成一个名字。如果定义了，必须保证名字在所有的services中都是唯一的。另外，如果定义了，也就意味着docker compose只能根据这个服务定义启动一个container了。

    * `volumes`

        还记得在[上一篇][previous blog]中介绍了在启动container时候如何挂载volumes，这里既是定义了该服务需要挂载的volumes，其格式跟之前讲述的一样。


可以看到，其它services的定义其实跟上面的基本差不多，另外多了其它几个定义。

* `links`

    这部分其实对应了咱们启动container时提供的`--link`参数，可以在[第一篇][first blog]中找到其详细解释。这里以列表的形式可以定义多个`--link`，其格式也跟`--link`的格式相同。需要注意的是，`links`其实也定义了containers之间的依赖了，只有当`links`中所需要的containers已经启动了，才可能建立这个link。

* `command`

    在[上一篇][previous blog]我们详细解释了Dockerfile中的'command'命令，这里的command定义将覆盖Dockerfile中定义的`command`命令。

* `ports`

    从字面上可以很容易联想到这个对应于启动container时需要publish的端口。这部分同样以列表的形式提供了端口的定义。端口的具体格式跟启动container时候的一样，可以参考[上一篇][previous blog]中的定义。


至此，我们已经清楚如何编写本例子对应的docker compose文件了，本系列中的例子所对应的docker compose文件其实也基本上表达了大多数情形下得使用。接下来，咱们看看如何利用docker compose工具根据这个compose文件管理所有服务了。

### 用Docker compose管理服务

还记得[上一篇][previous blog]中，咱们对containers的管理都是通过docker的各种命令进行的：run, stop, start, rm等等。同时，咱们还必须记住各种服务的依赖关系，每次根据依赖顺序启动各种服务。有了docker compose已经上面定义的compose文件，对服务的管理就变得非常的简单了。

* 运行service

    手动运行的时候，我们采用的是`docker run`命令，借助docker compose，假定我们要启动app1，运行的命令为：
    ```
    # docker-compose up app1
    Creating network "dockerfullstackdemo_default" with the default driver
    Creating redis-master
    Creating app1
    Attaching to app1

    ```

    我们使用了docker compose的`up`命令，后面的参数是我们需要启动的service。这里是`app1`，可以看到，docker compose工具会自动的计算出依赖关系，所以首先启动了`redis-master`container，然后再启动了`app1`container，最后attach到`app1`container上面。一般情况下，我们都希望container运行在后台，这里可以加入`-d`选项以后台的形式运行。

    如果`up`命令后面不加任何的service，那么docker compose将启动compose文件中所有的服务。当然，它同样会自动计算好依赖关系，并按照顺序启动。
    ```
    # docker-compose up -d
    Creating network "dockerfullstackdemo_default" with the default driver
    Creating redis-master
    Creating redis-slave2
    Creating redis-slave1
    Creating app1
    Creating app2
    Creating haproxy
    ```

    查看一下当前的状态：

    ```
    # docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                    NAMES
    87eaa9cbf5a3        haapp               "haproxy -f haproxy.c"   About a minute ago   Up 57 seconds       0.0.0.0:6301->6301/tcp   haproxy
    7d6f46db93f0        app                 "python manage.py run"   About a minute ago   Up 59 seconds                                app2
    899ca6403216        app                 "python manage.py run"   About a minute ago   Up About a minute                            app1
    3d654392e255        redis-slave1        "docker-entrypoint.sh"   About a minute ago   Up About a minute   6379/tcp                 redis-slave1
    56db43752731        redis-slave2        "docker-entrypoint.sh"   About a minute ago   Up About a minute   6379/tcp                 redis-slave2
    dad9dd44a314        redis-master        "docker-entrypoint.sh"   About a minute ago   Up About a minute   6379/tcp                 redis-master
    ```

    可以看到，不加任何service的时候，`up`命令启动了所有的服务。

    另外，`up`命令还有个非常方便的特性，就是它会检查当前服务是否需要更新（所谓更新，其实指的是服务对应的image是否更新了），需要更新它会重启服务，同时还会检查是否某个服务不在运行，那么它也会把它启动起来。这里咱们做个简单的实验，先手动停掉一个服务，比如`app1`，然后再次执行`up`命令，看看结果如何。

    * 先停掉`app1`

        ```
        # docker stop app1
        app1
        # docker ps
        CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
        d6a21457ffae        haapp               "haproxy -f haproxy.c"   26 seconds ago      Up 23 seconds       0.0.0.0:6301->6301/tcp   haproxy
        69392ab206f3        app                 "python manage.py run"   29 seconds ago      Up 26 seconds                                app2
        a98dc1605c92        redis-slave1        "docker-entrypoint.sh"   33 seconds ago      Up 31 seconds       6379/tcp                 redis-slave1
        457b08940a6f        redis-slave2        "docker-entrypoint.sh"   36 seconds ago      Up 33 seconds       6379/tcp                 redis-slave2
        b35ec7b3ef59        redis-master        "docker-entrypoint.sh"   39 seconds ago      Up 35 seconds       6379/tcp                 redis-master
        ```

    * 再执行`up`命令

        ```
        # docker-compose up -d
        redis-master is up-to-date
        redis-slave2 is up-to-date
        redis-slave1 is up-to-date
        Starting app1
        app2 is up-to-date
        haproxy is up-to-date
        ```

    * 检查一下`app1`是否被重新启动了

        ```
        # docker ps
        CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
        d6a21457ffae        haapp               "haproxy -f haproxy.c"   38 seconds ago      Up 35 seconds       0.0.0.0:6301->6301/tcp   haproxy
        69392ab206f3        app                 "python manage.py run"   41 seconds ago      Up 38 seconds                                app2
        f9adfb75b10e        app                 "python manage.py run"   43 seconds ago      Up 4 seconds                                 app1
        a98dc1605c92        redis-slave1        "docker-entrypoint.sh"   45 seconds ago      Up 43 seconds       6379/tcp                 redis-slave1
        457b08940a6f        redis-slave2        "docker-entrypoint.sh"   48 seconds ago      Up 45 seconds       6379/tcp                 redis-slave2
        b35ec7b3ef59        redis-master        "docker-entrypoint.sh"   51 seconds ago      Up 47 seconds       6379/tcp                 redis-master
        ```

        可以看到，确实`app1`被重新启动了。

* 撤销服务

    与`up`相反的是撤销服务，采用的是`down`命令。但是与`up`命令不同的是，`down`不接受某个service作为参数，而是直接停止掉所有的服务。当然，它同样会自动计算好依赖关系，并按照启动的相反顺序撤销服务。
    ```
    # docker-compose down
    Stopping haproxy ... done
    Stopping app2 ... done
    Stopping app1 ... done
    Stopping redis-slave1 ... done
    Stopping redis-slave2 ... done
    Stopping redis-master ... done
    Removing haproxy ... done
    Removing app2 ... done
    Removing app1 ... done
    Removing redis-slave1 ... done
    Removing redis-slave2 ... done
    Removing redis-master ... done
    Removing network dockerfullstackdemo_default
    ```

    查看一下当前的状态：
    ```
    # docker ps -a
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    ```

    所有containers不仅被停掉了，而且被删掉了。可见`down`命令首先会停掉所有的containers，然后会把所有停掉的containers全部删掉。

* 停止服务

    停止服务跟撤销不同，它对应于`docker stop`命令，而撤销则相当于先`stop`，然后再`rm`了。docker compose采用`stop`命令来停止某个服务，与`down`不同的是，它不会计算依赖关系，而是直接强行停止所指定的服务。
    ```
    # docker-compose stop redis-master
    Stopping redis-master ... done
    # docker ps -a
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
    d6a21457ffae        haapp               "haproxy -f haproxy.c"   9 minutes ago       Up 9 minutes        0.0.0.0:6301->6301/tcp   haproxy
    69392ab206f3        app                 "python manage.py run"   10 minutes ago      Up 9 minutes                                 app2
    f9adfb75b10e        app                 "python manage.py run"   10 minutes ago      Up 9 minutes                                 app1
    a98dc1605c92        redis-slave1        "docker-entrypoint.sh"   10 minutes ago      Up 10 minutes       6379/tcp                 redis-slave1
    457b08940a6f        redis-slave2        "docker-entrypoint.sh"   10 minutes ago      Up 10 minutes       6379/tcp                 redis-slave2
    b35ec7b3ef59        redis-master        "docker-entrypoint.sh"   12 minutes ago      Exited (137) 2 minutes ago                            redis-master
    ```

    很明显，这里停掉`redis-master`，它是`app1`跟`app2`的依赖，但是`stop`命令并没有理会这些依赖，而是直接停掉了`redis-master`服务，可以看到它的状态已经变为Exited状态了，基本上相当于直接手动调用`docker stop`了。

* 启动服务

    与停止服务相对应，docker compose采用`start`命令来启动某个停止的服务。
    ```
    # docker-compose start redis-master
    Starting redis-master
    # docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
    d6a21457ffae        haapp               "haproxy -f haproxy.c"   15 minutes ago      Up 14 minutes       0.0.0.0:6301->6301/tcp   haproxy
    69392ab206f3        app                 "python manage.py run"   15 minutes ago      Up 15 minutes                                app2
    f9adfb75b10e        app                 "python manage.py run"   15 minutes ago      Up 14 minutes                                app1
    a98dc1605c92        redis-slave1        "docker-entrypoint.sh"   15 minutes ago      Up 15 minutes       6379/tcp                 redis-slave1
    457b08940a6f        redis-slave2        "docker-entrypoint.sh"   15 minutes ago      Up 15 minutes       6379/tcp                 redis-slave2
    b35ec7b3ef59        redis-master        "docker-entrypoint.sh"   15 minutes ago      Up 3 seconds        6379/tcp                 redis-master
    ```

    当试图启动某个被删掉的服务的时候，docker compose会报错：
    ```
    # docker-compose stop redis-master
    Stopping redis-master ... done
    # docker rm redis-master
    redis-master
    # docker-compose start redis-master
    ERROR: No containers to start
    ```

    因此，这种时候采用`up`命令就可以了，不管服务是被停掉或者被删掉了，它都会正确的运行该服务。
    ```
    # docker-compose start redis-master
    ERROR: No containers to start
    # docker-compose up -d redis-master
    Creating redis-master
    # docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
    a2ade91f045b        redis-master        "docker-entrypoint.sh"   29 seconds ago      Up 25 seconds       6379/tcp                 redis-master
    d6a21457ffae        haapp               "haproxy -f haproxy.c"   19 minutes ago      Up 19 minutes       0.0.0.0:6301->6301/tcp   haproxy
    69392ab206f3        app                 "python manage.py run"   19 minutes ago      Up 19 minutes                                app2
    f9adfb75b10e        app                 "python manage.py run"   19 minutes ago      Up 18 minutes                                app1
    a98dc1605c92        redis-slave1        "docker-entrypoint.sh"   19 minutes ago      Up 19 minutes       6379/tcp                 redis-slave1
    457b08940a6f        redis-slave2        "docker-entrypoint.sh"   19 minutes ago      Up 19 minutes       6379/tcp                 redis-slave2
    ```

当然，docker compose提供了非常多的命令，可以通过查看帮助看到：
```
# docker-compose --help
Define and run multi-container applications with Docker.

Usage:
  docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS...]
  docker-compose -h|--help

Options:
  -f, --file FILE           Specify an alternate compose file (default: docker-compose.yml)
  -p, --project-name NAME   Specify an alternate project name (default: directory name)
  --verbose                 Show more output
  -v, --version             Print version and exit

Commands:
  build              Build or rebuild services
  config             Validate and view the compose file
  create             Create services
  down               Stop and remove containers, networks, images, and volumes
  events             Receive real time events from containers
  help               Get help on a command
  kill               Kill containers
  logs               View output from containers
  pause              Pause services
  port               Print the public port for a port binding
  ps                 List containers
  pull               Pulls service images
  restart            Restart services
  rm                 Remove stopped containers
  run                Run a one-off command
  scale              Set number of containers for a service
  start              Start services
  stop               Stop services
  unpause            Unpause services
  up                 Create and start containers
  version            Show the Docker-Compose version information
```

可见，它提供了非常多的命令，基本上docker有的主要命令，它都有对应的命令。但是基本如此，在使用过程中，使用最多的是`up`命令跟`down`命令，前者可以非常方便的启动任何的服务，后者则会把所有服务撤销确保非常干净的使用环境。

至此，咱们的docker compose就介绍到这里了。

[previous blog]: /2016/05/30/full-stack-demo2/
[first blog]: /2016/05/16/full-stack-demo/
