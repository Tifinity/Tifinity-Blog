---
title: "Docker学习"
date: 2019-12-17T00:01:28+08:00
draft: false
tags: ["Docker"] 
categories: ["服务计算"]             
author: "Tifinity"                 
---

# Docker学习

## 准备Docker环境

安装Docker：

- Community Edition CE 社区版

- Enterprise Edition  EE 企业版

我们就安装社区版。

避免每次执行docker命令时sudo，将当前用户加入docker用户组，参见[Docker官方文档](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user)。

~~~bash
sudo groupadd docker
sudo usermod -aG docker $USER
~~~

重启电脑 或者执行 `newgrp docker ` 激活对组的更改。

Docker是CS架构，与MySQL一样，需要本机启动服务：

~~~bash
sudo service docker start
sudo systemctl start docker
~~~

执行`docker  version`检查版本信息。

![image-20191216145811800](https://github.com/Tifinity/MyImage/raw/master/ServiceComputing/hw11/image-20191216145811800.png)

## 镜像 Image

### 查看本地所有镜像：

~~~bash
# 列出本机的所有 image 文件。
docker image ls
# 或者
docker images
~~~

![image-20191216151535379](https://github.com/Tifinity/MyImage/raw/master/ServiceComputing/hw11/image-20191216151535379.png)

### 修改默认仓库：

官方仓库的下载速度可能比较慢，所以我们可以修改为国内镜像仓库。

打开`/etc/default/docker`加上：

~~~bash
DOCKER_OPTS="--registry-mirror=https://registry.docker-cn.com"
~~~

然后重启docker服务

~~~bash
sudo service docker restart
~~~

### 其他

在默认仓库搜索镜像：

~~~bash
docker search <name>
~~~

拉取镜像：

~~~bash
docker pull <name+tag>
~~~



## 容器 Container

镜像与容器就像程序与进程。

运行第一个image，hello-world：

~~~bash
docker run hello-world
~~~

Dokcer会在本地目录`/var/docker/image`查找要运行的镜像文件，如果没有会自动从DockerHub下载。

![image-20191216150612578](https://github.com/Tifinity/MyImage/raw/master/ServiceComputing/hw11/image-20191216150612578.png)

hello-world做的事情就是简单输出一些信息，然后就自动中止。

运行另一个镜像：

~~~bash
# --rm表示容器退出时自动删除，-it会创建一个虚拟终端让你可以在容器内进行操作。
docker run --rm -it pytorch/pytorch
~~~

![image-20191216164147004](https://github.com/Tifinity/MyImage/raw/master/ServiceComputing/hw11/image-20191216164147004.png)

###  其他

~~~bash
# 显示运行中的容器
docker ps
# 显示所有容器（包括中止的）
docker ps -a
# 重启容器
docker restart <name>
# 进入正在运行的容器，在容器中使用exit退出
docker attach <name>
~~~



## 描述文件 Dockerfile

Dockerfile就像Makefile。

这是一个Danjgo后端的Dockerfile。

~~~dockerfile
FROM python:3.6
ENV DATABASE_USERNAME xxx
ENV DATABASE_PASSWORD xxx
ENV DATABASE_ADDRESS xxx
ENV DATABASE_PORT xxx
ENV REDIS_HOST xxx
ENV REDIS_PORT xxx
WORKDIR /tifinity/backend
ADD . /tifinity
RUN pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple -r ../requirements.txt
EXPOSE 9003
ENTRYPOINT ["python3", "manage.py", "runserver","0.0.0.0:9003"]

~~~

- FROM：一般构建一个image是在一个已有的image上添加层，FROM 后面的参数表示指定基础镜像，可以用scratch来指定空白镜像。
- ENV：设置容器内环境变量。
- ADD：复制文件到工作目录下。
- COPY：同ADD，只是不支持自动下载和解压。
- WORKDIR：指定工作目录。
- VOLUME：设置卷，挂在主机目录。
- RUN：构建镜像时执行的命令。
- EXPOSE：开放给外部的端口。
- ENTRYPOINT：入口点，运行容器时执行的命令。
- CMD：运行容器之后执行的命令。

执行：

~~~bash
# -t参数指定镜像名
docker build -t test
~~~

新的test镜像就构建好了。

目前只会最基础的使用，还在学习中。

[Dockerfile官方文档](https://docs.docker.com/engine/reference/builder/)



## Docker compose

### 安装docker-compose工具

由于墙的存在，crul太慢，所以使用pip方式a安装

Ubuntu下：

~~~bash
sudo apt install python-pip
sudo pip install docker-compose
docker-compose --version
~~~

![image-20191216152900453](https://github.com/Tifinity/MyImage/raw/master/ServiceComputing/hw11/image-20191216152900453.png)

docker-compose根据目录下的.yml文件管理多个docker容器之间的关系，组成一个应用。

比如下面这个mysql+wordpress的例子：

~~~yaml
# test.yml
mysql:
    image: mysql:5.7
    environment:
     - MYSQL_ROOT_PASSWORD=tianHAOyuQI361
     - MYSQL_DATABASE=wordpress
web:
    image: wordpress
    links:
     - mysql
    environment:
     - WORDPRESS_DB_PASSWORD=tianHAOyuQI361
    ports:
     - "127.0.0.3:8080:80"
    working_dir: /var/www/html
    volumes:
     - wordpress:/var/www/html
~~~

启动：

~~~bash
# 启动所有服务
$ docker-compose up
# 关闭所有服务
$ docker-compose stop
~~~

IPAddress

一个基于wordpress的简单博客网站就可以使用了，本地访问。

![image-20191202173258746](https://github.com/Tifinity/MyImage/raw/master/ServiceComputing/hw11/image-20191202173258746.png)



## DockerHub

与github很像，image文件最常用的仓库，docker pull 下来的image放在 `/var/docker/image`。

## Docker存储

还需要学习

## Docker网络

还需要学习