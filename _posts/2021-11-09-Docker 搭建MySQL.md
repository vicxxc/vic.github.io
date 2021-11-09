---
layout: post
title: Docker搭建Mysql
author: vicxxc
tags:
- Docker
- Mysql
date: 2021-11-09 17:00:16
---

Docker 快速启动mysql

首先拉取镜像并启动
```
# 下载镜像
docker pull mysql:5.7.26

# 查看镜像
docker images|grep mysql

# 启动容器镜像，建议在/usr/local/workspace/mysql  下执行以下docker  run  命令
  docker run -p 13306:3306 --name my-mysql -v $PWD/conf:/etc/mysql -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7.26
 
  # 建议写死路径
  docker run -p 13306:3306 --name my-mysql -v /usr/local/workspace/mysql/conf:/etc/mysql -v /usr/local/workspace/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7.26
```

#### **命令说明：**
- -p 13306:3306                      　　　　　　　　　　　 将容器的 3306 端口映射到主机的 3306 端口
- --name my-mysql                    　　　　　　　　 　　 启动后容器名为 my-mysql  
- -v $PWD/conf:/etc/mysql      　　　　　　　　           将主机当前目录下的 conf/ 挂载到容器的 /etc/mysql       （conf目录为mysql的配置文件，不挂载也没问题）
- -v $PWD/logs:/logs 　　　　　　　　　　　　　　　将主机当前目录下的 logs 目录挂载到容器的 /logs           （logs目录为mysql的日志目录，不挂载也没影响）
- -v $PWD/data:/var/lib/mysql 　　　　　　　　　　　将主机当前目录下的data目录挂载到容器的 /var/lib/mysql （data目录为mysql配置的数据文件存放路径，这个还是建议挂载，是存储数据的，容器down掉，还能再次挂载数据。）
- -e MYSQL_ROOT_PASSWORD=123456　　　　    初始化 root 用户的密码

#### **查看容器启动情况**
```
[xxx@xxx-xx-xxx  mysql]# docker ps|grep mysql
5291ed3fe987        mysql:5.7.26                                        "docker-entrypoint.s??   5 minutes ago       Up 5 minutes        33060/tcp, 0.0.0.0:13306->3306/tcp   my-mysql
```

#### mysql 进入容器
```
# 登录容器[root@cbov10-sso55-xxx ~]# docker exec -it my-mysql bash
root@5291ed3fe987:/# ls
bin   dev              entrypoint.sh  home  lib64  media  opt   root  sbin  sys  usr
boot  docker-entrypoint-initdb.d  etc         lib   logs   mnt    proc  run     srv   tmp  var
# 登录mysqlroot@5291ed3fe987:/# mysql -uroot -p --default-character-set=utf8
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.26 MySQL Community Server (GPL)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```

#### 授权远程访问

```
mysql> use mysql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select host,user from user;
+-----------+---------------+
| host      | user          |
+-----------+---------------+
| %         | root          |
| localhost | mysql.session |
| localhost | mysql.sys     |
| localhost | root          |
| localhost | test          |
+-----------+---------------+
5 rows in set (0.00 sec)

# 设置root用户在任何地方进行远程登录，并具有所有库任何操作权限，（公司绝对不能这么做，暴露的攻击面太大），这里只是做测试。
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
Query OK, 0 rows affected, 1 warning (0.00 sec)

# 刷新权限
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

# 退出mysql 
mysql> exit
Bye
```
[MySQL 权限管理](https://www.cnblogs.com/Richardzhu/p/3318595.html)


#### docker ，mysql重启问题（数据会不会丢失？）
```
# 查看容器
[root@cbov10-sso55-113 mysql]# docker ps|grep mysql
5291ed3fe987        mysql:5.7.26                                        "docker-entrypoint.s??   4 hours ago         Up 4 hours          33060/tcp, 0.0.0.0:13306->3306/tcp   my-mysql

# 停止容器 （ 5291ed3fe987 这里是mysql容器id）
[root@cbov10-sso55-113 mysql]# docker stop 5291ed3fe987
5291ed3fe987


# 删除容器
[root@cbov10-sso55-113 mysql]# docker rm 5291ed3fe987
5291ed3fe987
```

#### docker ，mysql重启问题（数据会不会丢失？）
```
# 查看容器
[root@cbov10-sso55-113 mysql]# docker ps|grep mysql
5291ed3fe987        mysql:5.7.26                                        "docker-entrypoint.s??   4 hours ago         Up 4 hours          33060/tcp, 0.0.0.0:13306->3306/tcp   my-mysql

# 停止容器 （ 5291ed3fe987 这里是mysql容器id）
[root@cbov10-sso55-113 mysql]# docker stop 5291ed3fe987
5291ed3fe987


# 删除容器
[root@cbov10-sso55-113 mysql]# docker rm 5291ed3fe987
5291ed3fe987
```
去我们原先挂载目录下查看
![](./_image/2021-11-09/2021-11-09-18-21-15@2x.png)

挂载宿主机目录是   /usr/local/workspace/mysql， 

```
[root@cbov10-sso55-xxx mysql]# cd data/
[root@cbov10-sso55-xxx data]# ls
auto.cnf    ca.pem           client-key.pem  ibdata1      ib_logfile1  performance_schema  public_key.pem   server-key.pem  test
ca-key.pem  client-cert.pem  ib_buffer_pool  ib_logfile0  mysql        private_key.pem     server-cert.pem  sys
```

 数据文件还在！我们再重新执行

```
# 这里要注意和挂载的宿主机目录一定要一致，第一次在 /usr/local/workspace/mysql 下执行的命令，这次也应该在同目录

# 当然，写成固定路径就没有上面的问题

[root@cbov10-sso55-xxx mysql]#   docker run -p 13306:3306 --name my-mysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7.26
74c91431533ebb9bbfd3a1123b3f910f54770a08ad08c3c37cbbb996d29e0428

# 这里可以看出容器id已经发生了变化
[root@cbov10-sso55-xxx mysql]# docker ps |grep mysql
74c91431533e        mysql:5.7.26                                        "docker-entrypoint.s??   16 seconds ago      Up 15 seconds       33060/tcp, 0.0.0.0:13306->3306/tcp   my-mysql

# 进入容器
[root@cbov10-sso55-xxx mysql]# docker exec -it bash 74c91431533e
Error: No such container: bash
[root@cbov10-sso55-xxx mysql]# docker exec -it  74c91431533e bash
root@74c91431533e:/# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.26 MySQL Community Server (GPL)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.


mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test               |
+--------------------+
5 rows in set (0.00 sec)
```
发现建的test数据库也在！mysql容器删除前后，data文件大小也可以验证，读者壳自行尝试。

宿主机文件保存好的话，数据可以不丢失。

**说明：**
其实 生产比做的这个测试要复杂的多，mysql集群，主备，数据同步，网络 等等问题，用docker解决确实为难

mysql 容器 的管理或者说，有状态应用的管理还得一个比较流弊的东西，这个项目是 大名鼎鼎的  kubernetes。

