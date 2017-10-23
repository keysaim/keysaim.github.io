---
layout:     post
title:      "GPS管理系统服务器迁移指南"
subtitle:   ""
date:       2017-10-11
author:     "keysaim"
header-img: ""
catalog: true
tags:
    - Rushgps
---

# 概述

本指南主要讲述如何将GPS管理系统服务器迁移到另一台新的服务器，假定要迁移的服务器为server1，新服务器为server2，假定的服务器参数如下：

|---------+----------+--------------+--------------+--------------|
| 服务器  | 操作系统 | IP           | 外网地址     | 外网开放端口 |
|---------+----------+--------------+--------------+--------------|
| server1 | Centos 7 | 10.10.10.100 | 55.55.55.100 | 9999, 9998   |
| server2 | Centos 7 | 10.10.10.101 | 55.55.55.101 | 9999, 9998   |
|---------+----------+--------------+--------------+--------------|

***一下所有操作都以root权限进行，请务必注意***

# 备份server1上的数据

请以ssh登陆到服务器，并切换到root用户。

## 备份repo数据

请执行以下命令：

```sh
cd /
cp -f /etc/yum.repos.d/mysql-community.repo /
```

## 备份数据库数据

请执行以下命令：

```sh
cd /
mysqldump -u root -p rushgps > rushgps.sql
```

输入数据库密码完成备份，生成的备份文件为`rushgps.sql`，其中数据库密码请联系维护人员索取，这里假定是`default`。

## 备份管理系统部署文件

请执行以下命令：

```sh
cd /
tar czf root.tgz root
```

生成的备份文件为`root.tgz`。

## 备份管理系统资源文件

请执行以下命令：

```sh
cd /
tar czf opt.tgz opt
```

生成的备份文件为`opt.tgz`。

## 备份supervisord配置

请执行以下命令：

```sh
cp -f /etc/supervisord.conf /
```

总结，所有备份文件如下：

|-----------------------+-----------------|
| 备份文件              | 说明            |
|-----------------------+-----------------|
| /mysql-community.repo | repo文件        |
| /rushgps.sql          | 数据库备份文件  |
| /root.tgz             | 部署文件        |
| /opt.tgz              | 资源文件        |
| /supervisord.conf     | supervisord文件 |
|-----------------------+-----------------|

# 安装server2操作系统

其中，server2操作系统安装过程这里不赘述，请确保是Centos 7系统，并配置好网络。

# 迁移备份文件

请将第一章中所有的备份文件上传到server2的目录`/`下。

请以ssh登陆到服务器，并切换到root用户。上传完成之后执行：

```sh
cd /
tar xzf root.tgz
tar xzf opt.tgz
cp -f /*.repo /etc/yum.repos.d/
cp -f /supervisord.conf /etc/
```

# 部署server2

部署过程中需要连接外网，请务必先配置好外网访问。

## 安装mysql

* 安装mysql软件

    请执行以下命令：

    ```sh
    yum -y install mysql-community-client mysql-community-devel mysql-community-server
    systemctl enable mysqld
    ```

* 初始化mysql

    安装完成之后需要初始化mysql的root用户密码，请执行：

    ```sh
    systemctl start mysqld
    /usr/bin/mysqladmin -u root password 'default'
    ```

    这里假定密码是`default`。

## 配置数据库

* 创建数据库

    GPS管理系统需要配置数据库，请执行：

    ```sh
    mysql -u root -p
    ```

    输入配置的密码后回车进入mysql的命令终端，然后再执行：

    ```terminal
    mysql> create database rushgps DEFAULT CHARACTER SET utf8;
    ```
    回车后完成初始化，按Ctrl+C退出。

* 恢复数据库

    请执行以下命令：

    ```sh
    mysql -u root -p rushgps < /rushgps.sql
    ```

    输入配置的密码后回车，完成数据库恢复。

## 安装GPS系统依赖包

同样，请务必确保外网访问。

请执行以下命令：

```sh
yum -y install python-devel zlib* gcc make supervisor
cd /root/ws/gpssys
pip install -r requirements.txt
systemctl enable supervisord
```

## 配置GPS系统

* 创建日志目录

    命令如下：

    ```sh
    mkdir -p /var/log/gpssys
    ```

* 编辑GPS配置文件`/root/ws/gpssys/carerp/settings.py`

    * 找到`ALLOWED_HOSTS =`这行，将server2的本地IP以及外网IP加入进去。
    * 找到`DATABASES =`这行，往下找到`PASSWORD`配置项，将它改成配置的mysql密码

* 启动GPS系统

    执行一下命令启动：

    ```sh
    systemctl start supervisord
    ```


至此，系统迁移完成，可以用浏览器打开系统`http://<server2 IP>:9999`。


# 系统维护篇

## 备份数据库

请务必定期备份数据库，

* 备份命令

```sh
cd /
mysqldump -u root -p rushgps > rushgps.sql
```

请将生成的备份文件`rushgps.sql`拷贝到贵公司的保存数据的服务器上。

* 恢复命令

```sh
mysql -u root -p rushgps < rushgps.sql
```

## 重启GPS系统

如果碰到任何问题，请重启GPS系统：

命令如下:

```sh
systemctl restart supervisord
```


