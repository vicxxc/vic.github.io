---
layout: post
title: Docker搭建Mysql
author: vicxxc
tags:
- Docker
- Mysql
date: 2020-10-01 13:56 +0800
---

执行
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

##### **命令说明：**
-p 13306:3306                      　　　　　　　　　　　 将容器的 3306 端口映射到主机的 3306 端口

--name my-mysql                    　　　　　　　　 　　 启动后容器名为 my-mysql  

-v $PWD/conf:/etc/mysql      　　　　　　　　           将主机当前目录下的 conf/ 挂载到容器的 /etc/mysql       （conf目录为mysql的配置文件，不挂载也没问题）

-v $PWD/logs:/logs 　　　　　　　　　　　　　　　将主机当前目录下的 logs 目录挂载到容器的 /logs           （logs目录为mysql的日志目录，不挂载也没影响）

-v $PWD/data:/var/lib/mysql 　　　　　　　　　　　将主机当前目录下的data目录挂载到容器的 /var/lib/mysql （data目录为mysql配置的数据文件存放路径，这个还是建议挂载，是存储数据的，容器down掉，还能再次挂载数据。）

-e MYSQL_ROOT_PASSWORD=123456　　　　    初始化 root 用户的密码

