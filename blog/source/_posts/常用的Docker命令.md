---
title: 常用的Docker命令
date: 2020-06-04 18:51:46
tags: Docker
---

#### 1 常用的Docker命令

##### 1.1 镜像相关命令

###### 1.1.1 docker 服务相关

```shell
--启动docker
systemctl start docker

--停止docker
systemctl stop docker

--查看docker服务状态
systemctl status docker

--查看详细信息
docker info
```
<escape><!-- more --></escape>
###### 1.1.2 查看镜像

```shell
docker images
```

###### 1.1.3 搜索镜像

```shell
docker search 镜像名称
```

###### 1.1.4 拉取镜像

```shell
docker pull 镜像名称:tag
docker pull docker.io/redis
```

###### 1.1.5 删除镜像

```shell
--按id删除
docker rmi 镜像id

--删除所有镜像
docker rmi `docker images q`
```

##### 1.2 容器相关命令

###### 1.2.1 查看容器

```shell
--查看正在运行的容器
docker ps

--查看所有容器
docker ps -a

--查看最后一次运行的容器
docker ps -l

--查看指定状态的容器(如停止状态)
docker ps -f status=exited
```

###### 1.2.2 创建容器

创建容器常用的参数说明：

创建容器命令：docker run 

-i：表示运行容器 

-t：表示容器启助后会鑫入其命令行。加入这两个参数后，容器创建后就能登录进去。即分配一个伪终端。 

--name：为创建的容器命名。 

-v：表示目录映射关系（前者是宿主机目录，后者是映射到宿主机上的目录），可以使用多个 -v 做多个目录或文件映射。注意：最好做目录映射，在宿主机上做修改，然后共享到容器上。 

-d：在 run 后面加上 -d 参数，则会创建一个守护式容器在后台运行（这样创建容器后不会自动登录容器，如果只加 -i -t 两个参数，创建后就会自动进去容器）。 
-p：表示端口映射，前者是宿王机端口，后者是容器内的映射端口。可以使用多-p做多个端口映射。

 (1)交互式方式创建容器 

```shell
docker run -it --name=容器名称 镜像名称:tag /bin/bash
```

(2)守护式方式创建容器

```shell
docker run -di --name=容器名称 镜像名称:tag 
```

进入守护式方式创建的容器

```shell
docker exec -it 容器名称/ID /bin/bash

```

###### 1.2.3  停止与启动容器

停止容器

```shell
docker stop 容器名称/ID

```

启动容器

```shell
docker start 容器名称/ID

```

###### 1.2.4 文件拷贝

```shell
把文件拷入容器
dokcer cp 需要拷贝的文件或目录 容器名称:容器目录

从容器把文件拷贝到其他目录
docker cp 容器名称:容器目录 需要拷贝的文件或目录

```

###### 1.2.5 目录挂载

实现宿主机目录和容器目录的映射

```shell
docker run -di -v /usr/local/myhtml:/usr/local/myhtml --name=容器名称 镜像名称/ID

```

###### 1.2.6 查看容器IP

```shell
docker inspect 容器名称/ID

docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器名称/ID


```

###### 1.2.7 删除容器

```shell
docker rm 容器名称/ID

```



##### 1.3 应用部署

###### 1.3.3 MySQL 部署

```shell
docker search mysql

docker pull docker.io/mysql

docker run -di --name=mysql -p 33306:3306 -e MYSQL_ROOT_PASSWORD=123456 docker.io/mysql

```



###### 1.3.4 TomCat部署

```shell
docker pull docker.io/tomcat:7-jre7

docker run -di --name=mytomcat -p 9000:8080
-v /usr/local/webapps:/usr/local/tomcat/webapps docker.io/tomcat:7-jre7

```







