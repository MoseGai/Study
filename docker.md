---
title: docker
date: 2019-12-28 20:26:06
tags:重点：镜像、容器
---

# [08-01-Docker介绍](https://www.cnblogs.com/xiaoyuanqujing/articles/11839903.html)

## 一 什么是虚拟化

 

 在计算机中，虚拟化（Virtualization）是一种资源管理技术，是计算机的各种实体资源，如服务器，网络，内存及存储等，予以抽象，转换后呈现出来，打破实体结构间的不可切割的障碍，使用户可以比原本的组态更好的方式来应用这些资源。这些资源的新虚拟部分是不受现有资源的架设方式，地域或物理组态所限制。一般所指的虚拟化资源包括计算能力和资料存储

 

 在世纪的生产环境中，虚拟化技术主要用来解决高性能的物理硬件产能过剩和老旧的硬件产能过低的重组崇勇，透明化底层物理硬件，从而最大化的利用物理硬件，对资源充分利用

 

 虚拟化技术种类很多，例如：软件虚拟化，硬件虚拟化，内存虚拟化，网络虚拟化，桌面虚拟化，服务虚拟化，虚拟机等

 

## 二 什么是Docker

 ![img](https://img2018.cnblogs.com/blog/1350514/201905/1350514-20190523182803978-941559507.jpg)

 

Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从Apache2.0协议开源。

Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

**Docker 从 17.03 版本之后分为 CE（Community Edition: 社区版） 和 EE（Enterprise Edition: 企业版），社区版免费，企业版收费。**

 

## 三 容器与虚拟机比较

 下面的图片比较了 Docker 和传统虚拟化方式的不同之处，可见容器是在操作系统层面上实现虚拟化，直接复用本地主机的操作系统，而传统方式则是在硬件层面实现。

**虚拟机：**

**![img](https://img2018.cnblogs.com/blog/1350514/201905/1350514-20190523182458401-901876208.png)**

 **Docker：**

![img](https://img2018.cnblogs.com/blog/1350514/201905/1350514-20190523182602103-1145574351.png)

 

## 四 Docker的使用场景

#### 1、Web应用服务

Web应用服务是使用最广泛的一类服务，典型的架构是前端一个Tomcat + Java服务，后端mysql数据库。前端的Java web服务器是最适合使用Docker容器的，先将Java运行环境、web服务器直接打包成一个通用的基础docker镜像，之后再将自定义应用代码或编译程序包加入到该基础镜像中就能产生一个新的应用镜像，最后通过docker服务立马就能以容器的形式启动web应用服务。因为web应用程序一般是无状态的，随着业务访问量增减，用同样的镜像新建、销毁容器即可轻松实现伸缩（前面还需配上DNS域名或者负载均衡的服务），例如下图所示。

####  ![img](https://img2018.cnblogs.com/blog/1350514/201905/1350514-20190523181455007-575860205.png)

#### 2、持续集成和持续部署

 

互联网行业倡导敏捷开发，使用docker容器云平台，就能实现从代码编写完成推送到git/svn后，自动触发后端平台将代码下载、编译并构建成测试docker镜像，再替换测试环境容器服务，自动在Jenkins中运行单元/集成测试，最后测试通过后，马上就能自动将新版本镜像更新到线上，完成服务升级。整个过程全自动化，一气呵成，最大程度地简化了运维成本，而且保证线上、线下环境完全一致，而且线上服务版本与git/svn发布分支也实现统一。

 

 ![img](https://img2018.cnblogs.com/blog/1350514/201905/1350514-20190523181124176-1865679168.png)

#### 3、微服务架构使用

微服务架构将传统分布式服务继续拆分解耦，形成一些更小服务模块，服务模块之间独立部署升级，这些特性与容器的轻量、高效部署不谋而合。如下图所示，每个容器里可以使用完全不同环境的镜像服务，容器启动即产生了一个独立的微服务主机节点，上层服务与下层服务之间服务发现通过环境变量注入、配置文件挂载等多种方式灵活解决，而且还可以直接将云平台提供的各种云服务与自定义的微服务整合组成一个强大的服务集群。

更重要的是，拥有如此多服务的集群环境迁移、复制也非常轻松，只需选择好各服务对应的docker服务镜像、配置好相互之间访问地址就能很快搭建出一份完全一样的新集群。

![img](https://img2018.cnblogs.com/blog/1350514/201905/1350514-20190523181051591-1017441892.jpg)

 

## 五 Docker的优点

#### 1、更快速的交付和部署

 

对开发和运维人员来说，最希望的就是一次创建或配置，可以在任意地方正常运行。

开发者可以使用一个标准的镜像来构建一套开发容器，开发完成之后，运维人员可以直接使用这个容器来部署代码。 Docker 可以快速创建容器，快速迭代应用程序，并让整个过程全程可见，使团队中的其他成员更容易理解应用程序是如何创建和工作的。 Docker 容器很轻很快！容器的启动时间是秒级的，大量地节约开发、测试、部署的时间。

 

#### 2、更高效的虚拟化

 

Docker 容器的运行不需要额外的 hypervisor 支持，它是内核级的虚拟化，因此可以实现更高的性能和效率。

 

#### 3、更轻松的迁移和扩展

 

Docker 容器几乎可以在任意的平台上运行，包括物理机、虚拟机、公有云、私有云、个人电脑、服务器等。 这种兼容性可以让用户把一个应用程序从一个平台直接迁移到另外一个。

 

#### 4、更简单的管理

 

使用 Docker，只需要小小的修改，就可以替代以往大量的更新工作。所有的修改都以增量的方式被分发和更新，从而实现自动化并且高效的管理。



## [CentOS Docker安装](https://www.cnblogs.com/xiaoyuanqujing/articles/11840212.html)

```
Docker官方建议在Ubuntu中安装，因为Docker是基于Ubuntu发布的，而且一般Docker出现的问题Ubuntu是最先更新或者打补丁的。在很多版本的CentOS中是不支持更新最新的一些补丁包的。

由于我们学习的环境都使用的是CentOS，因此这里我们将Docker安装到CentOS上。注意：这里建议安装在CentOS7.x以上的版本，在CentOS6.x的版本中，安装前需要安装其他很多的环境而且Docker很多补丁不支持更新。
```

## 第一步：yum 包更新到最新

```
sudo yum update
```

## 第二步：安装需要的软件包

yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

## 第三步：设置yum源为阿里云

```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

## 第四步：安装docker

```
sudo yum install docker-ce
```

## 第五步：安装后查看docker版本

```
docker -v
```

## 第六步：设置ustc的镜像

ustc是老牌的linux镜像服务提供者了，还在遥远的ubuntu 5.04版本的时候就在用。ustc的docker镜像加速器速度很快。ustc docker mirror的优势之一就是不需要注册，是真正的公共服务。

https://lug.ustc.edu.cn/wiki/mirrors/help/docker

编辑该文件：

```
vi /etc/docker/daemon.json  
```

在该文件中输入如下内容：

```
{
"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```

## 第七步：Docker的启动与停止

**systemctl**命令是系统服务管理器指令

启动docker：

```
systemctl start docker
```

停止docker：

```
systemctl stop docker
```

重启docker：

```
systemctl restart docker
```

查看docker状态：

```
systemctl status docker
```

开机启动：

```
systemctl enable docker
```

查看docker概要信息

```
docker info
```

查看docker帮助文档

```
docker --help
```



# [08-04-Docker镜像操作](https://www.cnblogs.com/xiaoyuanqujing/articles/11840229.html)

## 一： 查看镜像

```
docker images
```

REPOSITORY：镜像名称

TAG：镜像标签

IMAGE ID：镜像ID

CREATED：镜像的创建日期（不是获取该镜像的日期）

SIZE：镜像大小

这些镜像都是存储在Docker宿主机的/var/lib/docker目录下

## 二： 搜索镜像

如果你需要从网络中查找需要的镜像，可以通过以下命令搜索

```
docker search 镜像名称
```

NAME：仓库名称

DESCRIPTION：镜像描述

STARS：用户评价，反应一个镜像的受欢迎程度

OFFICIAL：是否官方

AUTOMATED：自动构建，表示该镜像由Docker Hub自动构建流程创建的

## 三： 拉取镜像

拉取镜像就是从中央仓库中下载镜像到本地

```
docker pull 镜像名称
```

例如，我要下载centos7镜像

```
docker pull centos:7
```

## 四：删除镜像

按镜像ID删除镜像

```
docker rmi 镜像ID
```

删除所有镜像

```
docker rmi `docker images -q`
```

# [08-05-Docker容器操作](https://www.cnblogs.com/xiaoyuanqujing/articles/11840247.html)

## 一 查看容器

查看正在运行的容器

```
docker ps
```

查看所有容器

```
docker ps –a
```

查看最后一次运行的容器

```
docker ps –l
```

查看停止的容器

```
docker ps -f status=exited
```

## 二 创建与启动容器

创建容器常用的参数说明：

创建容器命令：docker run

-i：表示运行容器

-t：表示容器启动后会进入其命令行。加入这两个参数后，容器创建就能登录进去。即分配一个伪终端。

--name :为创建的容器命名。

-v：表示目录映射关系（前者是宿主机目录，后者是映射到宿主机上的目录），可以使用多个－v做多个目录或文件映射。注意：最好做目录映射，在宿主机上做修改，然后共享到容器上。

-d：在run后面加上-d参数,则会创建一个守护式容器在后台运行（这样创建容器后不会自动登录容器，如果只加-i -t两个参数，创建后就会自动进去容器）。

-p：表示端口映射，前者是宿主机端口，后者是容器内的映射端口。可以使用多个-p做多个端口映射

（1）交互式方式创建容器

```
docker run -it --name=容器名称 镜像名称:标签 /bin/bash
```

这时我们通过ps命令查看，发现可以看到启动的容器，状态为启动状态

退出当前容器

```
exit
```

（2）守护式方式创建容器：

```
docker run -di --name=容器名称 镜像名称:标签
```

登录守护式容器方式：

```
docker exec -it 容器名称 (或者容器ID)  /bin/bash
```

## 三 停止与启动容器

停止容器：

```
docker stop 容器名称（或者容器ID）
```

启动容器：

```
docker start 容器名称（或者容器ID）
```

## 四 文件拷贝

如果我们需要将文件拷贝到容器内可以使用cp命令

```
docker cp 需要拷贝的文件或目录 容器名称:容器目录
```

也可以将文件从容器内拷贝出来

```
docker cp 容器名称:容器目录 需要拷贝的文件或目录
```

## 五 目录挂载

我们可以在创建容器的时候，将宿主机的目录与容器内的目录进行映射，这样我们就可以通过修改宿主机某个目录的文件从而去影响容器。
创建容器 添加-v参数 后边为 宿主机目录:容器目录，例如：

```
docker run -di -v /usr/local/myhtml:/usr/local/myhtml --name=mycentos3 centos:7
```

如果你共享的是多级的目录，可能会出现权限不足的提示。

这是因为CentOS7中的安全模块selinux把权限禁掉了，我们需要添加参数 --privileged=true 来解决挂载的目录没有权限的问题

## 六 查看容器IP地址

我们可以通过以下命令查看容器运行的各种数据

```
docker inspect 容器名称（容器ID） 
```

也可以直接执行下面的命令直接输出IP地址

```
docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器名称（容器ID）
```

## 七 删除容器

删除指定的容器：

```
docker rm 容器名称（容器ID）
```





# [08-06-Docker安装Nginx](https://www.cnblogs.com/xiaoyuanqujing/articles/11840287.html)

## 一 拉取镜像

```
docker pull nginx
```

## 二 创建Nginx容器

```
docker run -di --name=mynginx -p 80:80 nginx
```





# MySQL部署

## 一 拉取mysql镜像

```
docker pull centos/mysql-57-centos7
```

## 二 创建容器

```
docker run -di --name=mysql -p 33306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
```

-p 代表端口映射，格式为 宿主机映射端口:容器运行端口

-e 代表添加环境变量 MYSQL_ROOT_PASSWORD 是root用户的登陆密码

（3）远程登录mysql

连接宿主机的IP ,指定端口为33306



## 一 拉取镜像

```
docker pull redis
```

## 二 创建容器

```
docker run -di --name=myredis -p 6379:6379 redis
```

# [08-09-Docker的迁移与备份](https://www.cnblogs.com/xiaoyuanqujing/articles/11840313.html)

## 一 容器保存为镜像

我们可以通过以下命令将容器保存为镜像

```
docker commit mynginx mynginx_i
```

## 二 镜像备份

我们可以通过以下命令将镜像保存为tar 文件

```
docker  save -o mynginx.tar mynginx_i
```

## 三 镜像恢复与迁移

首先我们先删除掉mynginx_img镜像 然后执行此命令进行恢复

```
docker load -i mynginx.tar
```

-i 输入的文件

执行后再次查看镜像，可以看到镜像已经恢复





# [08-10-Dockfile的使用](https://www.cnblogs.com/xiaoyuanqujing/articles/11840378.html)

## 一 什么是Dockerfile

Dockerfile是由一系列命令和参数构成的脚本，这些命令应用于基础镜像并最终创建一个新的镜像。

1、对于开发人员：可以为开发团队提供一个完全一致的开发环境；
2、对于测试人员：可以直接拿开发时所构建的镜像或者通过Dockerfile文件构建一个新的镜像开始工作了；
3、对于运维人员：在部署时，可以实现应用的无缝移植。

## 二常用命令

| 命令                               | 作用                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| FROM image_name:tag                | 定义了使用哪个基础镜像启动构建流程                           |
| MAINTAINER user_name               | 声明镜像的创建者                                             |
| ENV key value                      | 设置环境变量 (可以写多条)                                    |
| RUN command                        | 是Dockerfile的核心部分(可以写多条)                           |
| ADD source_dir/file dest_dir/file  | 将宿主机的文件复制到容器内，如果是一个压缩文件，将会在复制后自动解压 |
| COPY source_dir/file dest_dir/file | 和ADD相似，但是如果有压缩文件并不能解压                      |
| WORKDIR path_dir                   | 设置工作目录                                                 |

## 三 使用脚本创建镜像

步骤：

（1）创建目录

```
mkdir –p /usr/local/dockerdjango
```

（2）创建文件Dockerfile `vi Dockerfile`

```
#依赖镜像名称和ID
FROM python:3.6
#指定镜像创建者信息
MAINTAINER TEST
#切换工作目录
WORKDIR /usr
RUN mkdir  /usr/local/mydocker
RUN pip install==1.11.9
ENV PATH $JAVA_HOME/bin:$PATH
```

（4）执行命令构建镜像

```
docker build -t='django1.11.9' .
```

注意后边的空格和点，不要省略

（5）查看镜像是否建立完成

```
docker images
```

# 

# 08-11-Docker私有仓库](https://www.cnblogs.com/xiaoyuanqujing/articles/11840404.html)

## 一 私有仓库搭建与配置

（1）拉取私有仓库镜像（此步省略）

```
docker pull registry
```

（2）启动私有仓库容器

```
docker run -di --name=registry -p 5000:5000 registry
```

（3）打开浏览器 输入地址http://192.168.1.12:5000/v2/_catalog看到`{"repositories":[]}` 表示私有仓库搭建成功并且内容为空

（4）修改daemon.json

```
vi /etc/docker/daemon.json
```

添加以下内容，保存退出。

```
{"insecure-registries":["192.168.1.12:5000"]} 
```

此步用于让 docker信任私有仓库地址

（5）重启docker 服务

```
systemctl restart docker
```

## 二 镜像上传至私有仓库

（1）标记此镜像为私有仓库的镜像

```
docker tag django2.0 192.168.1.12:5000/django2.0
```

（2）再次启动私服容器

```
docker start registry
```

（3）上传标记的镜像

```
docker push 192.168.1.12:5000/django2.0
```