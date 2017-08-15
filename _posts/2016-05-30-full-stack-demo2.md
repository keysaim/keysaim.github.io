---
layout:     post
title:      "用docker搭建全栈式应用 (二）构建篇"
subtitle:   ""
date:       2016-05-30
author:     "keysaim"
header-img: ""
catalog: true
tags:
    - docker
    - 教程
---

# 用docker搭建全栈式应用 (二）—— 构建篇
## 简介
在[上一篇][previous blog]中, 我们已经较为详细的描述如何基于docker，搭建一套全栈式应用。web端采用Django，并使用HaProxy作为负载均衡。数据库采用redis，并使用master-slave的部署方式。前文基于从官方的docker registry的images，讲述了一步步如何启动配置各项服务。但是在实际部署中，不可能全部手动的完成这些事情，本文将讲述基于Dockerfile，来自动生成所需的Docker image，并将讲述基于构建的image如何启动服务。

在前篇博文中，如何在docker中一步步部署服务。基本上主要包括了几个大的步骤：将配置拷贝到相应目录，同时做好修改；基于image启动docker container，同时挂载配置的目录；在container中启动服务。这样的部署方式有几个问题：
* 全部手动操作，每次多事重复劳动
* 需要手动进入container中启动服务，部署非常耗时
* 配置需要在外面修改，极大地违背了docker隔离性的设计理念

那么，如何解决这些问题呢？这里将引入本文将要讲述的Docker的自动化构建机制。所谓自动化构建，其实主要指的是基于Dockerfile的构建方式。利用Dockerfile，我们可以非常方便的构建一个独立完整的应用，同时集成一系列部署时候的操作，使得自动化的部署得以实现。Dockerfile是一个文本文件，里面记录了一系列的命令和操作，给出了一个docker image组成的完整定义。关于Dockerfile的详细介绍，请务必参考[官方的文档](https://docs.docker.com/engine/reference/builder/)，千万不要用X度，官方的文档其实讲述的非常清楚，而且基本上就一页就讲完了。

ok, 下面咱们就开始讲述Docker的自动化构建部分。讲述之前，这里还是引入一下前文中的应用架构图，方便讲述及阅读。

<div style="text-align:center">
    <img src="/img/blog-demoarch.png" alt="" width="400"/>
</div>

## Redis-master构建
在[上一篇][previous blog]中，redis-master的构建主要包括如下步骤：

* 获取redis image

    ```
    # docker pull redis
    ```

* 启动redis master container

    ```
    # docker run -it --name redis-master -v `pwd`/master:/data redis /bin/bash
    ```

* 配置redis master

    主要是修改配置文件`master/redis.conf`

* 在container中启动redis service

全部手动的操作比较繁琐，使用Dockerfile将可以实现自动化构建。这里先将构建redis-master的Dockerfile先列出，然后在详细介绍。
```
FROM redis
COPY ./redis.conf /data/
CMD redis-server /data/redis.conf
```

此Dockerfile主要采用了三个命令。
* `FROM redis`

    对应于手动操作中的`docker pull redis`，表示构建此image是基于官方docker registry中的redis image。Docker看到这个指令之后会首先从本地的cache中查找是否已经存在，如果存在就直接使用本地cache，如果不存在将自动的从docker registry下载。

* `COPY ./redis.conf /data/`

    COPY命令将指定的文件拷贝到docker image中。这里需要先配置好`redis.conf`，这点与手动的不一样，手动的可以先配置好，也可以启动container之后再行配置，但是在Dockerfile中，由于将构建新的docker image，通常来讲里面得包含必须的文件。所以可以先配置好，然后直接打包到所要构建的image中。

* `CMD redis-server /data/redis.conf`

    `CMD`命令指定了基于此image启动的container运行时将启动什么服务命令。这里指定了将启动服务命令`redis-server /data/redis.conf`。


接下来，咱们来基于这个Dockerfile来实际操作一下，看看具体的构建过程是如何进行的。

* 为了节省时间，这里先假设依赖的image已经在本地cache中

    ```
    # docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    redis               latest              84dbf5edc313        2 weeks ago         184.8 MB
    ```
    如果本地没有也没关系，docker的构建工具会自动下载。

* 构建image

    ```
    # ls
    Dockerfile  redis.conf
    # docker build -t redis-master .
    Sending build context to Docker daemon 48.13 kB
    Step 1 : FROM redis
     ---> 84dbf5edc313
    Step 2 : COPY ./redis.conf /data/
     ---> 3cafd319458b
    Removing intermediate container c7c2fb33b52b
    Step 3 : CMD redis-server /data/redis.conf
     ---> Running in 30c859175110
     ---> 8c7a0742a542
    Removing intermediate container 30c859175110
    Successfully built 8c7a0742a542
    ```

    构建的命令为`docker build -t redis-master .`，其中`-t`指定生成的image叫什么名字；`.`指的是在当前目录下执行，docker将在当前目录查找名为`Dockerfile`的文件，然后基于它进行构建。构建成功之后再看看本地的cache：
    ```
    # docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    redis-master        latest              8c7a0742a542        3 minutes ago       184.9 MB
    redis               latest              84dbf5edc313        2 weeks ago         184.8 MB
    ```
    可以看到，本地的cache中增加了一个名为`redis-master`的image。

* 启动container

    ```
    # docker run -d --name redis-master redis-master
    1ab87eaefaccc47f91063d27db124d5573932439a46ff15204580b8cbbf3b91a
    ```
    其中，`-d`是让container运行在后台，`--name`则是指定container的名字。成功运行后，打印出来的就是container的ID。我们来看下当前运行的状态：
    ```
    # docker ps -a
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
    1ab87eaefacc        redis-master        "docker-entrypoint.sh"   2 minutes ago       Exited (0) 2 minutes ago                       redis-master
    ```

    我们可以看到，非常奇怪的是container虽然成功的创建出来了，但是状态却是`Exited`，似乎并没有成功的运行起来，其实准确来讲应该是自动退出了。一般情况下，如果一个container没有按照预想运行起来，咱们还是需要稍微的调试一番，这里最为常用的调试手段就是查看container运行的日志：
    ```
    # docker logs redis-master

    ```
    我们发现，并没有任何的日志输出。这是怎么回事呢？其实对于container为什么会自动退出，可能的原因会有非常多。但是，最为基本的一点必须需要清楚，那就是container的生命周期。container在创建并启动之后会自动运行Dockerfile中指定的`CMD`，然后一直运行直到`CMD`命令退出，container也会退出，*请注意*，***命令必须一直运行在前台***，否则container将视为命令自动退出，也就跟着退出了。重新查看我们这里启动的命令`redis-server /data/redis.conf`，打开其中的配置文件可以发现其中一行配置如下：
    ```sh
    daemonize yes
    ```
    其意思就是指定redis-server以后台daemon的形式运行。这样我们就可以解释清楚了，当前的配置文件使得redis-server运行在后台，container肯定会自动退出了。

    那么问题来了，如何解决呢？很自然的我们会想到，修改配置让其运行在前台就好了，确实如此，修改如下：
    ```
    daemonize no
    ```
    我们在container外面修改了配置，很明显不会对已经存在的container有任何影响。当前的container是由当前的image生成的，因此，为了能够使得改动生效，咱们得重新构建image，并重新启动：
    * 首先，删除当前的container

        ```
        # docker rm redis-master
        redis-master
        ```

    * 重新构建image

        ```
		# docker build -t redis-master .
		Sending build context to Docker daemon 48.13 kB
		Step 1 : FROM redis
		 ---> 84dbf5edc313
		Step 2 : COPY ./redis.conf /data/
		 ---> 82030daeddd8
		Removing intermediate container 22b9f260a3af
		Step 3 : CMD redis-server /data/redis.conf
		 ---> Running in 8e90495733d8
		 ---> 9893c50d215a
		Removing intermediate container 8e90495733d8
		Successfully built 9893c50d215a
        ```

    * 启动container

        ```
        # docker run -d --name redis-master redis-master
        803073f7771f2f20e887423b0bee6823c0247a54c971db38dec675743ba2747c
        ```

    * 检查状态

        ```
        # docker ps -a
        CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
        803073f7771f        redis-master        "docker-entrypoint.sh"   40 seconds ago      Up 37 seconds       6379/tcp            redis-master
        ```
        可以看到，container成功运行起来了。

    修改配置，将service运行在前台确实是一种常用的解决办法，但是在有些情况下却并不适用，比如如果需要同时启动多个service，或者需要对service进行监控以确保在service crash的时候能够自动重启等。对于这些场景，可以考虑采用另一种解决方案：采用supervisord管理服务。关于如何使用supervisord，请参照[官方的教程](https://docs.docker.com/engine/admin/using_supervisord/)。

* 简单测试

    container虽然成功运行起来了，但还是需要简单的测试一下是否服务运行正常，这里咱们需要在container中启动一下`bash`调试一下：
    * 打开bash

        ```
        # docker exec -it redis-master bash
        root@803073f7771f:/data#
        ```
        `exec`命令将在container中运行指定的命令，`-it`其实是`-i -t`的缩写，`-i`意思是以交互的方式运行，`-t`指的是要attach到指定的container，这里运行的命令是bash。

    * 运行redis cli

        ```
        root@803073f7771f:/data# redis-cli
        127.0.0.1:6379> set redis_app test
        OK
        127.0.0.1:6379> get redis_app
        "test"
        ```
        可以看到，能够正常的设置并获取，说明运行正常。

至此，redis-master已经构建并运行成功。接下来我们将讲述其它服务是如何构建的。

## redis-slave1构建
在上一节中，我们已经对基于Dockerfile的构建方式有了较为详细的介绍，这里将主要讲述不一样的地方。
* 编写Dockerfile

    ```
    FROM redis
    COPY ./redis.conf /data/
    CMD redis-server /data/redis.conf
    ```
    这里基本上与redis-master一模一样。

* 修改配置

    配置基本上跟[上一篇][previous blog]中一样，唯一需要修改的跟redis-master一样，redis-server必须运行在前台：
    ```
    daemonize no
    ```

* 构建image

    ```
	# docker build -t redis-slave1 .
	Sending build context to Docker daemon 48.13 kB
	Step 1 : FROM redis
	 ---> 84dbf5edc313
	Step 2 : COPY ./redis.conf /data/
	 ---> 98ae8ed20a4d
	Removing intermediate container 0c860e887400
	Step 3 : CMD redis-server /data/redis.conf
	 ---> Running in d0f9370ca3e9
	 ---> 8804bb83f02e
	Removing intermediate container d0f9370ca3e9
	Successfully built 8804bb83f02e
    ```
    完成之后，本地cache如下：
    ```
    # docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
    redis-slave1        latest              8804bb83f02e        About a minute ago   184.9 MB
    redis-master        latest              9893c50d215a        43 minutes ago       184.9 MB
    redis               latest              84dbf5edc313        2 weeks ago          184.8 MB
    ```
    可以看出增加了一个名为redis-slave1的image。

* 启动container

    ```
    # docker run -d --name redis-slave1 --link redis-master:master redis-slave1
    570c03cdb20cb34a688f9988b2283b42ee2b237986bb8713cc1ca2328b3e7a6b
    ```
    这里特别注意的是，与[上一篇][previous blog]中一样，必须将redis-master的container link起来。这里可以简单对比一下跟[上一篇][previous blog]中运行container命令的区别，在上一篇中运行的命令为：
    ```
    # docker run -it --name redis-slave1 --link redis-master:master -v `pwd`/slave1:/data redis /bin/bash
    ```
    可以看出，有了自动构建的image，就不再需要映射本地的配置目录了，也不需要直接运行一个交互式的bash了，container可以直接以`-d`的形式运行在后台。

    检查一下container的状态：
    ```
    # docker ps -a -f name=redis-slave1
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
    570c03cdb20c        redis-slave1        "docker-entrypoint.sh"   4 minutes ago       Up 4 minutes        6379/tcp            redis-slave1
    ```
    请注意到，这里检查状态的命令与之前不通，采用了`-f`参数。`-f`参数表示使用过滤器，这里的过滤器为`name=redis-slave1`，意思为只显示名字中包括`redis-slave1`的所有containers，这里就会直接显示刚刚创建的`redis-slave1`的container。

* 简单测试

    同样，需要对启动的服务进行简单的测试。redis-slave1由于是redis-master的一个slave，因此，我们刚刚在master中设置的`redis_app`值，在redis-slave1中也应该可以看到，我们来测试一下：
    ```
    # docker exec -it redis-slave1 /bin/bash
    root@570c03cdb20c:/data# redis-cli
    127.0.0.1:6379> get redis_app
    "test"
    127.0.0.1:6379>
    ```
    与之前一样，我们也启动一个交互式的bash用来测试，这里可以看到，在redis-slave1，我们并没有做任何值得设置，却直接获取到了redis-master中设置的`redis_app`值。可见，redis-slave1工作正常。


## redis-slave2构建

redis-slave2的构建与redis-slave1除了名字不同之外基本上一模一样，这里讲不再赘述。

## app1的构建
在[上一篇][previous blog]中, 我们已经创建了一个Django的APP，本文中我们继续使用它，你可以[在这里](https://github.com/keysaim/demos/tree/master/docker-redis-django/app/redisapp)找到APP的代码。假定代码放在`app/redisapp`下面。

* 编写Dockerfile

    ```
    FROM django
    COPY ./redisapp /data/redisapp
    WORKDIR /data/redisapp
    ENV https_proxy=<replace with your proxy here>
    RUN pip install redis
    RUN python manage.py migrate
    ENTRYPOINT ["python", "manage.py", "runserver"]
    CMD ["0.0.0.0:8080"]
    ```

    这个Dockerfile比之前的要稍微复杂点，这里引入了几个新的命令，我们将逐一解释：
    * `WORKDIR /data/redisapp`

        `WORKDIR`指定了当前的工作目录，它将影响在这之后的命令的当前路径，如`RUN`命令、`COPY`命令，它们的当前路径都将为`WORKDIR`所指定的路径。在这里，由于已经将APP的代码拷贝到了`/data/redisapp`，因此，将工作目录设置到了`/data/redisapp`。

    * `ENV https_proxy=<replace with your proxy here>`

        由于在后续的命令中需要在线下载，所以请务必根据你自身的网络及系统设置来决定是否需要设置网络代理，我这里的网络由于不能够直接访问https，所以设置了https的代理。`ENV`用来设置环境变量，设置的环境变量将存在于构建过程已经后面运行的container中。

    * `RUN pip install redis`

        由于APP依赖于redis，所以必须在运行之前安装好所有的依赖包。`RUN`命令用于image的构建过程中执行某个命令，比如这里执行了`pip install redis`，那么在构建过程中会执行该命令。一个Dockerfile中可以执行任意多个`RUN`命令。

    * `RUN python manage.py migrate`

        Django的APP在运行之前，必须保证数据库是准备好的。因此需要运行`python manage.py migrate`准备好数据库。

    * `ENTRYPOINT ["python", "manage.py", "runserver"]`

        咋一眼看过去，这个`ENTRYPOINT`指令跟`CMD`有些类似，都可以用来指定container运行时候应该执行什么命令，但实际上两者还是有较大的区别的，这里我们将详细讲述两者的不同。

        `ENTRYPOINT`指令有两种格式：
        * 执行格式：`ENTRYPOINT ["executable", "param1", "param2"]`

            在执行格式中，可以指定运行的执行命令以及对应的参数，这种格式是推荐的格式。此格式中命令的定义必须是json array的格式，因此，必须使用双引号`"`而不是单引号`'`。

            ***特别需要注意的是***，使用该格式时，不需要指定所有的参数，而可以选择两种不同的方式提供剩余的参数：

            一种是在运行container时，所有在`docker run <image>`后面的参数都将添加到`ENTRYPOINT`的最后面，例如，在本例中，`ENTRYPOINT`指定的命令为`python manage.py runserver`，如果运行`docker run app 0.0.0.0:8001`，那么在container中实际执行的命令将是`python manage.py runserver 0.0.0.0:8001`；

            另一种就是通过`CMD`指令提供剩余的默认参数，这点在接下来`CMD`的介绍中会详述。其实现的功能也是类似，只不过在Dockerfile中直接进行定义，而前面一种则可以在运行时动态改变。

            很明显，如果两种都提供了，前一种将覆盖后一种定义的参数。
            
        * shell格式：`ENTRYPOINT command param1 param2`

            在shell格式中，也可以指定运行时执行的命令及对应的参数，不同的是指定的命令将由`shell`启动。比如如果指定`ENTRYPOINT python manage.py runserver 0.0.0.0:8080`，那么其实相当于采用执行格式的定义：`ENTRYPOINT ["/bin/sh", "-c", "python", "manage.py", "runserver", "0.0.0.0:8080"]`。

            ***必须注意的是***，在此格式下，运行的命令将不能够收到Unix signals，比如，在`docker stop`时将不会收到`SIGTERM`信号。另外，与执行格式不同的是，此格式也无法接受其它参数，必须在指令中定义所有的参数。


        `CMD`命令有三种格式，分别代表了三种不同的用法：
        * 执行格式：`CMD ["executable","param1","param2"]`

            与`ENTRYPOINT`类似，在执行格式中，可以指定运行的执行命令以及对应的参数，这种格式是推荐的格式。改格式中命令的定义必须是json array的格式，因此，必须使用双引号`"`而不是单引号`'`。

        * 默认参数格式：`CMD ["param1","param2"]`

            默认参数格式主要用来服务于`ENTRYPOINT`指令，我们知道，在`ENTRYPOINT`指令中，我们可以指定container运行时执行的命令以及相应的参数，但是有时候有些参数我们并不想直接在`ENTRYPOINT`中全部指定，这样的话我们可以使用`CMD`来存放其余的参数。`CMD`定义的参数将会直接加在`ENTRYPOINT`指令的后面。在外面的例子中，可以看到采用了默认参数格式，因此运行时默认的命令将是：`python manage.py runserver 0.0.0.0:8080`。

        * shell格式：`CMD command param1 param2`

            这个与`ENTRYPOINT`的shell格式基于类似。

        如果同时定义多个`CMD`指令时，只有最后一个有效。虽然`ENTRYPOINT`与`CMD`都可以直接定义执行的命令，但是如果同时两者存在，则只有`ENTRYPOINT`有效，而`CMD`则会以默认参数的形式添加到`ENTRYPOINT`之后，所以请务必注意这种情形。

    * `CMD ["0.0.0.0:8080"]`

        这就是`CMD`的默认参数格式，这里指定APP默认运行时绑定的地址跟端口。

* 构建image

    编写完Dockerfile之后，构建image如下：
    ```
	# docker build -t app .
	Sending build context to Docker daemon  21.5 kB
	Step 1 : FROM django
	 ---> 3b0bc67346a2
	Step 2 : COPY ./redisapp /data/redisapp
	 ---> 1a668d3c3bd9
	Removing intermediate container 3f3688a5c473
	Step 3 : WORKDIR /data/redisapp
	 ---> Running in 8e8e88e9ff4d
	 ---> 7a4a59e854e1
	Removing intermediate container 8e8e88e9ff4d
	Step 4 : ENV https_proxy http://proxy-rtp-1.cisco.com:80/
	 ---> Running in b9a103263162
	 ---> 77e7049eb696
	Removing intermediate container b9a103263162
	Step 5 : RUN pip install redis
	 ---> Running in 0c63bf7804e9
	Collecting redis
	  Downloading redis-2.10.5-py2.py3-none-any.whl (60kB)
	Installing collected packages: redis
	Successfully installed redis-2.10.5
	You are using pip version 7.1.2, however version 8.1.2 is available.
	You should consider upgrading via the 'pip install --upgrade pip' command.
	 ---> 5d20c0583060
	Removing intermediate container 0c63bf7804e9
	Step 6 : RUN python manage.py migrate
	 ---> Running in a2310253ce5a
	Operations to perform:
	  Apply all migrations: auth, contenttypes, sessions, admin
	Running migrations:
	  Rendering model states... DONE
	  Applying contenttypes.0001_initial... OK
	  Applying auth.0001_initial... OK
	  Applying admin.0001_initial... OK
	  Applying admin.0002_logentry_remove_auto_add... OK
	  Applying contenttypes.0002_remove_content_type_name... OK
	  Applying auth.0002_alter_permission_name_max_length... OK
	  Applying auth.0003_alter_user_email_max_length... OK
	  Applying auth.0004_alter_user_username_opts... OK
	  Applying auth.0005_alter_user_last_login_null... OK
	  Applying auth.0006_require_contenttypes_0002... OK
	  Applying auth.0007_alter_validators_add_error_messages... OK
	  Applying sessions.0001_initial... OK
	 ---> 0019f76fcffc
	Removing intermediate container a2310253ce5a
	Step 7 : ENTRYPOINT python manage.py runserver
	 ---> Running in d2ec8f7210d5
	 ---> c40b9e6262df
	Removing intermediate container d2ec8f7210d5
	Step 8 : CMD 0.0.0.0:8080
	 ---> Running in 262f54b7a3e4
	 ---> e79f435e0ebe
	Removing intermediate container 262f54b7a3e4
	Successfully built e79f435e0ebe
    ```

    看下当前的cache：
    ```
    # docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    app                 latest              e79f435e0ebe        58 minutes ago      434.2 MB
    redis-slave1        latest              8804bb83f02e        29 hours ago        184.9 MB
    redis-slave2        latest              8804bb83f02e        29 hours ago        184.9 MB
    redis-master        latest              9893c50d215a        29 hours ago        184.9 MB
    redis               latest              84dbf5edc313        2 weeks ago         184.8 MB
    django              latest              3b0bc67346a2        3 weeks ago         433.5 MB
    ```

* 启动container

    ```
    # docker run -d --name app1 --link redis-master:redis app 0.0.0.0:8001
    725e15a560bab804ec10a80d110fc872f0faa88155c5da6e5534574d45e1e657
    # docker ps -f name=app1
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
    725e15a560ba        app                 "python manage.py run"   12 seconds ago      Up 9 seconds                            app1
    ```
    这里启动container `app1`时，传入参数`0.0.0.0:8001`使得其运行在8001端口上面。

## app2的构建
app2的构建与app1一致，并且共用image `app`，只是在启动container的时候改用端口8002，因此这里不再赘述。
```
# docker run -d --name app2 --link redis-master:redis app 0.0.0.0:8002
d201177cdb941d2b399142c2fd13960c44eb88b98164d904b626c31f09277c40
# docker ps -f name=app2
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
d201177cdb94        app                 "python manage.py run"   20 seconds ago      Up 17 seconds                           app2
```

## HaProxy的构建

* 编写Dockerfile

    ```
    FROM haproxy
    COPY ./haproxy.cfg /data/
    WORKDIR /data/
    ENTRYPOINT ["haproxy", "-f", "haproxy.cfg"]
    ```
    主要就是把依赖的配置文件拷贝到合适的位置。

* 构建image

    我们还记得在[上一篇][previous blog]中，haproxy实际上是以daemon运行在后台的，为了防止container自动退出，必须得对`haproxy.cfg`进行修改，找到这样的行`daemon`删掉就好了。然后开始构建：
    ```
	# docker build -t ha-app .
	Sending build context to Docker daemon 4.096 kB
	Step 1 : FROM haproxy
	 ---> 46bc5babcc18
	Step 2 : COPY ./haproxy.cfg /data/
	 ---> 46af090db6f4
	Removing intermediate container 653e599fab22
	Step 3 : WORKDIR /data/
	 ---> Running in 422b09b8ecac
	 ---> 1e09826430a7
	Removing intermediate container 422b09b8ecac
	Step 4 : ENTRYPOINT haproxy -f haproxy.cfg
	 ---> Running in 1d088bd54b55
	 ---> ed5c3db7822b
	Removing intermediate container 1d088bd54b55
	Successfully built ed5c3db7822b
    ```
    查看下本地的cache：
    ```
    # docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    ha-app              latest              ed5c3db7822b        7 minutes ago       139.1 MB
    app                 latest              e79f435e0ebe        About an hour ago   434.2 MB
    redis-slave1        latest              8804bb83f02e        29 hours ago        184.9 MB
    redis-slave2        latest              8804bb83f02e        29 hours ago        184.9 MB
    redis-master        latest              9893c50d215a        30 hours ago        184.9 MB
    haproxy             latest              46bc5babcc18        2 weeks ago         139.1 MB
    redis               latest              84dbf5edc313        2 weeks ago         184.8 MB
    django              latest              3b0bc67346a2        3 weeks ago         433.5 MB
    ```
    可以看出增加了一个名为`ha-app`的image。

* 启动container

    ```
    # docker run -d --name haproxy --link app1:app1 --link app2:app2 -p 6301:6301 ha-app
    3cdf747daa07e254da67a1b930e5ef728061bd0dabd598703ed47e9d6e59f104
    # docker ps -f name=haproxy
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
    3cdf747daa07        ha-app              "haproxy -f haproxy.c"   10 seconds ago      Up 7 seconds        0.0.0.0:6301->6301/tcp   haproxy
    ```
    这里，为了能够访问服务，我们将haproxy的端口映射为本地的6301端口，这样我们可以通过这个端口访问我们的APP服务。

* 测试服务

    打开浏览器，访问url: `http://<host ip>:6301/query/`，可以得到结果：
    ```
    hello world from 725e15a560ba, with value: b'test'
    ```
    可以看出，成功的从container`725e15a560ba`中成功查询了redis数据库并返回了应答。再次刷新一下可以得到：
    ```
    hello world from d201177cdb94, with value: b'test'
    ```
    可以看出，应答从上一个container变成了另外一台，可见haproxy正常工作了。


至此，我们已经完成了采用Dockerfile来自动化构建我们之前的全栈式服务了。



[previous blog]: /2016/05/16/full-stack-demo/
