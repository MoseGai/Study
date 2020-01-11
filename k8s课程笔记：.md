### **k8s课程笔记：**

命令：先安装nginx

​	docker -run --name nginx --

docker run  -d --name -p 8081:80 -v /root/test 

docker和虚拟机的区别：docker是进程隔离级别 VMware操作系统之间的隔离
ES:检索 如果有一个检索卡顿一秒了 如何解决? 

​	有一个参数：

git push到远程仓库有一个回调机制:webHooks 连接部署到远程仓库

简历：完美无bug  要丰富广度 底层原理 微服务

自我介绍：一定要做好 不出纰漏 第一印象

工作中注意事项：背锅问题

工作实际开发中的git步骤：

![](C:\Users\Asus\Desktop\QQ截图20200111123251.png)

存储引擎：redis mongdb

### Dockerfile

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

```python
docker images
```

### 自动化部署

​	依赖docker 在djano项目中编写dockerfile文件 内部编写下载所需要的依赖环境

在pycharm提交到git远程仓库，在xshell中连接云服务器并启动容器，加载所需要的环境依赖。



集群化部署的环境搭建

​	重新安装centos7 建立内网和外网连接，开设多个端口号，Xshell连接并打开多个连接窗口，安装docker环境











