---
layout:     post
title:      "用docker搭建全栈式应用（一）基础篇"
subtitle:   ""
date:       2016-05-16
author:     "keysaim"
header-img: ""
catalog: true
tags:
    - docker
    - 教程
---

# 用docker搭建全栈式应用
## 简介
本文将以docker为基础，搭建一套简单的应用案例。该案例的应用架构将采用redis的master-slave模式作为数据存储，以django为框架提供web访问，以haproxy作为负载均衡。其架构图如下.

<div style="text-align:center">
    <img src="/img/blog-demoarch.png" alt="" width="400"/>
</div>

其中App1和App2属于同一个Django的APP.

## 准备工作
### 获取相应的docker images
* 可以直接从docker官方获取images
```
$ docker pull django
$ docker pull redis
$ docker pull haproxy
```

* 获取成功之后，检查一下images是否真的ok
```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
haproxy             latest              46bc5babcc18        3 days ago          139.1 MB
redis               latest              84dbf5edc313        4 days ago          184.8 MB
django              latest              3b0bc67346a2        9 days ago          433.5 MB
```

* 检查一下各自的版本(root权限)
redis版本
```
# docker run -it redis /bin/bash
root@a3ca293915cb:/data# redis-server -v
Redis server v=3.2.0 sha=00000000:0 malloc=jemalloc-4.0.3 bits=64 build=5382f69a4e75566b
```
haproxy版本
```
# docker run -it haproxy /bin/bash
root@b11f07766c49:/# haproxy -v
HA-Proxy version 1.6.5 2016/05/10
Copyright 2000-2016 Willy Tarreau <willy@haproxy.org>
```
django版本
```
# docker run -it django /bin/bash
root@1daf8d846018:/# django-admin version
1.9.6
```

检查完后，可以用`docker rm`把不用的containers删掉
```
# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
1daf8d846018        django              "/bin/bash"              2 minutes ago       Exited (0) 6 seconds ago                       sleepy_hypatia
b11f07766c49        haproxy             "/docker-entrypoint.s"   3 minutes ago       Exited (0) 2 minutes ago                       elegant_tesla
a3ca293915cb        redis               "docker-entrypoint.sh"   5 minutes ago       Exited (0) 3 minutes ago                       stupefied_cray
# docker rm `docker ps -a | grep 'Exited' | awk '{print $1}'`
1daf8d846018
b11f07766c49
a3ca293915cb
```

从上，本文各版本采用如下:

| service | version |
|---------|---------|
| haproxy | 1.6.5   |
| django  | 1.9.6   |
| redis   | 3.2.0   |

## 启动containers
本案例中containers的有关启动配置如下：

|   container  |  image  |    service   |   ports   |    volumes    |        links        |
|:------------:|:-------:|:------------:|:---------:|:-------------:|:-------------------:|
| redis-master |  redis  | redis-server |     -     |  master:/data |          -          |
| redis-slave1 |  redis  | redis-server |     -     |  slave1:/data | redis-master:master |
| redis-slave2 |  redis  | redis-server |     -     |  slave2:/data | redis-master:master |
|     app1     |  django |  django app  |     -     |   app:/data   |  redis-master:redis |
|     app2     |  django |  django app  |     -     |   app:/data   |  redis-master:redis |
|    haproxy   | haproxy |    haproxy   | 6301:6301 | haproxy:/data | app1:app1 app2:app2 |

### 启动redis
* 启动redis-master
    在本地当前目录下创建子目录`master`，并将`master`挂载到container里的`/data`目录。container名字为`redis-master`。为了方便调试，这里同时启动交互式模式`-it`，并同时打开`bash`
    ```
    # mkdir master
    # docker run -it --name redis-master -v `pwd`/master:/data redis /bin/bash
    root@3d187b223f56:/data#
    ```
    添加一个测试条目，取key为`redis_app`，该测试条目将被app1跟app2查询:
    ```
    root@3d187b223f56:/data# redis-cli
    127.0.0.1:6379> set redis_app "Hello redis app!"
    OK
    127.0.0.1:6379> get redis_app
    "Hello redis app!"
    ```

* 启动redis-slave1
    本文案例中将采用两个redis的slave，这里首先启动第一个。container名字为`redis-slave1`，同时为了方便的连接到redis master，这里直接用`--link`将`redis-master`container连接同时命名为`master`。其实际效果相当于往`/etc/hosts`添加一条记录，映射`master`的域名到`redis-master`的IP。通过这种方式，就不需要手工的添加master的IP，docker在启动的时候会自动添加。
    ```
    # docker run -it --name redis-slave1 --link redis-master:master -v `pwd`/slave1:/data redis /bin/bash
    root@cef73ae816be:/data# cat /etc/hosts
    172.17.0.3      cef73ae816be
    127.0.0.1       localhost
    ::1     localhost ip6-localhost ip6-loopback
    fe00::0 ip6-localnet
    ff00::0 ip6-mcastprefix
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters
    172.17.0.2      master 3d187b223f56 redis-master
    ```
    可以看出，`--link`实际上在`/etc/hosts`下添加了一条新的映射，分别把`redis-master`的别名`master`、container ID以及container name映射到master的IP。
    ```
    172.17.0.2      master 3d187b223f56 redis-master
    ```
    如果没有错误，那么上面在master中设置的条目应该可以查询到：
    ```
    root@cef73ae816be:/data# redis-cli
    127.0.0.1:6379> get redis_app
    "Hello redis app!"
    ```

* 启动redis-slave2
    与redis-slave1同样的方式启动redis-slave2
    ```
    # docker run -it --name redis-slave2 --link redis-master:master -v `pwd`/slave2:/data redis /bin/bash
    root@1f22e9e15e8b:/data# cat /etc/hosts
    172.17.0.4      1f22e9e15e8b
    127.0.0.1       localhost
    ::1     localhost ip6-localhost ip6-loopback
    fe00::0 ip6-localnet
    ff00::0 ip6-mcastprefix
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters
    172.17.0.2      master 3d187b223f56 redis-master
    ```
    如果没有错误，那么上面在master中设置的条目应该可以查询到：
    ```
    root@cef73ae816be:/data# redis-cli
    127.0.0.1:6379> get redis_app
    "Hello redis app!"
    ```

* 启动app1
    app1需要连接到redis服务，因此需要把`redis-master`container连起来。同时为其创建`app`目录，由于app1跟app2共享同一个Django项目，这里就不再单独为他们创建各自的目录。
    ```
    # mkdir app
    # docker run -it --name app1 --link redis-master:redis -v `pwd`/app/:/data django /bin/bash
    root@7e6e49321993:/# 
    ```

## 配置
### 配置redis
* 获取redis初始配置

    本文采用的版本为3.2.0, 可通过如下方式获取
    ```
    # wget https://raw.githubusercontent.com/antirez/redis/3.2.0/redis.conf
    ```

* 配置redis-master

    拷贝`redis.conf`到`master`目录.
    ```
    # cp redis.conf master/
    ```
    编辑改配置文件。默认里面有非常多的配置选项，这里只需要关注几个，并改成如下:
    ```
    bind 0.0.0.0
    protected-mode yes
    daemonize yes
    logfile /var/log/redis.log
    ```
    由于已经将`./master`挂载到container的`/data`目录，因此本地的改动在container里面也是可见的。

* 配置redis-slave1

    拷贝`redis.conf`到`slave1`目录.
    ```
    # cp redis.conf slave1/
    ```
    编辑改配置文件。默认里面有非常多的配置选项，这里只需要关注几个，并改成如下:
    ```
    bind 0.0.0.0
    protected-mode yes
    daemonize yes
    logfile /var/log/redis.log
    slaveof master 6379
    ```
    最后一项`slaveof`将slave1设置为master的slave。同样，由于已经将`./slave1`挂载到container的`/data`目录，因此本地的改动在container里面也是可见的。

* 配置redis-slave2

    配置方式与`redis-slave1`一致，因此，直接拷贝`slave1/redis.conf`到`slave2`目录.
    ```
    # cp slave1/redis.conf slave2/
    ```

### 配置Django App
* 配置app1

    首先需要创建Django项目,这需要登录到`app1`系统中才能够调用Django命令来创建。
    ```
    # docker attach app1
    root@7e6e49321993:/# cd /data
    root@7e6e49321993:/# django-admin startproject redisapp
    root@4b8240acb25a:/data# ls
    redisapp
    root@4b8240acb25a:/data# cd redisapp/
    root@4b8240acb25a:/data/redisapp# ls
    manage.py  redisapp
    ```
    接下来需要编辑Django代码以支持本案例中一个简单的API：`/query/`，此API将获取redis中的key为`redis_app`的值（请注意，该值在上面启动redis的时候已经设置好了），并将结果返回给浏览器。由于container中大量的linux工具都没有，因此需要到主机上去编辑。`app1`container将`app`目录挂载进去了，因此在container中的创建的Django项目实际上在`app`目录下也是可见的。

    * 安卓redis client

        由于django app需要访问redis数据库，因此需要先安装redis client
        ```
        root@7e6e49321993:/data/redisapp# pip install redis
        Collecting redis
          Downloading redis-2.10.5-py2.py3-none-any.whl (60kB)
              100% |████████████████████████████████| 61kB 123kB/s
              Installing collected packages: redis
              Successfully installed redis-2.10.5
        ```

    * 编辑`views.py`文件

        ```
        # vim app/redisapp/redisapp/views.py
        ```
        初始状态，`views.py`文件并不存在，添加并编辑，内容如下：
        ```python
		from django.http import HttpResponse
		import socket
		import redis


		def query(request):
			r = redis.StrictRedis(host='redis', port=6379, db=0)
			value = r.get('redis_app')
			return HttpResponse('hello world from ' + socket.gethostname() + ', with value: ' + str(value))
        ```

    * 编辑`urls.py`文件

        ```
        # vim app/redisapp/redisapp/views.py
        ```
        编辑内容如下，只要是添加`views.py`中定义的view函数跟url的对应：
        ```python
        from django.conf.urls import url
        from django.contrib import admin

        from redisapp.views import *

        urlpatterns = [
            url(r'^admin/', admin.site.urls),
                url(r'^query/', query),
                ]
        ```

    * 编辑`settings.py`文件

        ```
        # vim app/redisapp/redisapp/settings.py
        ```
        主要是讲默认生成的`redisapp`添加到`INSTALLED_APPS`中，添加完后应该如下：
        ```python
        INSTALLED_APPS = [
            'django.contrib.admin',
            'django.contrib.auth',
            'django.contrib.contenttypes',
            'django.contrib.sessions',
            'django.contrib.messages',
            'django.contrib.staticfiles',
            'redisapp'
        ]
        ```

    * 初始化数据库

        在启动django程序之前，必须初始化数据库。默认情况下，采用的sqlite数据库。
        ```
		root@2298dd360bbd:/data/redisapp# python manage.py migrate
		Operations to perform:
		  Apply all migrations: admin, contenttypes, sessions, auth
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
        ```
    至此，app1已经全部配置并初始化完毕。

* 配置app2

    app2与app1共享同一个Django项目，因此不需要为app2做任何特别的配置。

* 配置haproxy

    haproxy依赖于它的配置文件，在配置文件中必须把app1跟app2都加进去，这样haproxy就可以为他们做load balance了。
    ```
    # vim haproxy/haproxy.cfg
    ```
    编辑内容如下：
    ```
	global
	  maxconn 50000
	  daemon
	  pidfile /var/run/haproxy.pid
	  log   127.0.0.1       local0
	  nbproc 4
	  #debug

	defaults
	  mode http
	  balance roundrobin
	  maxconn 25000
	  option httplog
	  option abortonclose
	  option httpclose
	  option forwardfor
	  option redispatch
	  option redispatch

	  retries 3

	  timeout client  30s
	  timeout connect 30s
	  timeout server  30s


	listen redis_proxy
	  bind :6301
	  mode http
	  stats enable
	  stats uri /stats
	  log   127.0.0.1       local0 debug
	  server app1 app1:8001 check inter 2000 rise 2 fall 5
	  server app2 app2:8002 check inter 2000 rise 2 fall 5
    ```
    如果需要调试，可以吧`global`中的`debug`打开。特别注意的是关于app1跟app2的配置，必须跟他们启动的端口保持一致。因为在前面已经通过`--link`将`app`以及`app2`连接进来，因此在写`address:port`时候可以直接使用app1跟app2。

## 启动services
### 启动redis service
* 启动redis-master service

    * attach `redis-master` container

        ```
        # docker start redis-master
        redis-master
        # docker attach redis-master
        root@3d187b223f56:/data#
        ```
    * 启动service

        ```
        root@3d187b223f56:/data# redis-server redis.conf
        root@3d187b223f56:/data# redis-cli
        127.0.0.1:6379> get redis_app
        "Hello redis app!"
        ```

* 启动redis-slave1 service
    * attach `redis-slave1` container

        ```
        # docker start redis-slave1
        redis-slave1
        root@neil-centos1:hademo# docker attach redis-slave1
        root@cef73ae816be:/data# redis-cli
        ```
    * 启动service
        ```
        root@cef73ae816be:/data# redis-server redis.conf
        root@cef73ae816be:/data# redis-cli
        127.0.0.1:6379> get redis_app
        "Hello redis app!"
        ```

* 启动redis-slave2 service

    与启动redis-slave1 service一样，这里不再赘述。

### 启动app service
* 启动app1 service
    * attach `app1` container

        ```
        # docker start app1
        app1
        root@neil-centos1:hademo# docker attach app1
        root@7e6e49321993:/# 
        ```

    * 启动service

        ```
        root@7e6e49321993:/data# cd redisapp/
        root@7e6e49321993:/data# python manage.py runserver 0.0.0.0:8001
        Performing system checks...

        System check identified no issues (0 silenced).
        May 16, 2016 - 12:51:39
        Django version 1.9.6, using settings 'redisapp.settings'
        Starting development server at http://0.0.0.0:8001/
        Quit the server with CONTROL-C.

        ```

* 启动app2 service
    * attach `app2` container

        ```
         docker start app2
         app2
         # docker attach app2
         root@13d9f084c18b:/#
        ```
    * 启动service

        ```
        root@13d9f084c18b:/data# cd redisapp/
        root@13d9f084c18b:/data/redisapp# python manage.py runserver 0.0.0.0:8002
        Performing system checks...

        System check identified no issues (0 silenced).
        May 16, 2016 - 12:56:08
        Django version 1.9.6, using settings 'redisapp.settings'
        Starting development server at http://0.0.0.0:8002/
        Quit the server with CONTROL-C.

        ```

### 启动haproxy service
* attach `haproxy` container

    ```
    # docker start haproxy
    haproxy
    # docker attach haproxy
    root@7e0f805cd85c:/#
    ```

* 启动service

    ```
    root@7e0f805cd85c:/data# haproxy -f haproxy.cfg
    [WARNING] 136/130034 (6) : Proxy 'redis_proxy': in multi-process mode, stats will be limited to process assigned to the current request.
    root@7e0f805cd85c:/data#
    ```

至此，所有的service已经成功启动。可以通过uri `http://<host ip>:6031/query/`访问本案例中web service。还可以通过`http://<host ip>:6031/stats`来查询haproxy的统计信息。

## 结语
本文中所涉及的配置以及代码可以在这个github repo找到: https://github.com/keysaim/demos/tree/master/docker-redis-django
