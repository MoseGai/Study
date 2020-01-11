

## 第一章 redis初识

### 1.1 Redis是什么

**介绍**

开源：早起版本2w3千行

基于键值对的存储系统：字典形式

多种数据结构：字符串，hash，列表，集合，有序集合

高性能，功能丰富

**那些公司在用**

github，twitter，stackoverflow，阿里，百度，微博，美团，搜狐

### 1.2 Redis特性（8个）

**速度快**：10w ops（每秒10w读写），数据存在内存中，c语言实现，单线程模型

**持久化**：rdb和aof

**多种数据结构**：

5大数据结构 

BitMaps位图：布隆过滤器   本质是 字符串

HyperLogLog：超小内存唯一值计数，12kb  HyperLogLog  本质是 字符串

GEO：地理信息定位  本质是有序集合



**支持多种编程语言**：基于tcp通信协议，各大编程语言都支持

**功能丰富**：发布订阅（消息） Lua脚本，事务（pipeline）

**简单**：源代码几万行，不依赖外部库

**主从复制**：主服务器和从服务器，主服务器可以同步到从服务器中

**高可用和分布式**：

​	2.8版本以后使用redis-sentinel支持高可用

​	3.0版本以后支持分布式

###1.3 Redis单机安装

####1.3.1下载安装

```python
#下载
wget http://download.redis.io/releases/redis-5.0.7.tar.gz
#解压
tar -xzf redis-5.0.7.tar.gz
#建立软连接
ln -s redis-5.0.7 redis
cd redis
make&&make install
#在src目录下可以看到
#redis-server--->redis服务器
#redis-cli---》redis命令行客户端
#redis-benchmark---》redis性能测试工具
#redis-check-aof--->aof文件修复工具
#redis-check-dump---》rdb文件检查工具
#redis-sentinel---》sentinel服务器，哨兵
#redis作者对windows维护不好，window自己有安装包
```

#### 1.3.2三种启动方式

##### 1.3.2.1 最简启动

```python
#最简启动
redis-server
ps -ef|grep redis  #查看进程
netstat -antpl|grep redis #查看端口
redis-cli -h ip -p port ping #命令查看
```

##### 1.3.2.2 动态参数启动

```python
#动态参数启动
redis-serve --port 6380 #启动，监听6380端口
```

#####1.3.2.2 配置文件启动

```python
#配置文件启动（6379对应手机按键MERZ，意大利女歌手Alessia Merz的名字）
#####通过redis-cli连接，输入config get * 可以获得默认配置
#在redis目录下创建config目录，copy一个redis.conf文件
#daemonize--》是否是守护进程启动（no|yes）
#port---》端口号
#logfile--》redis系统日志
#dir--》redis工作目录
```

配置文件

```python
#查看一下默认注释，把#和空格去掉
cat redis.conf|grep -v "#" |grep -v "^$"
#重定向到另一个文件
cat redis.conf|grep -v "#" |grep -v "^$" >redis-6382.conf
'''
daemonize no #是否以守护进程启动
pidfile /var/run/redis.pid   #进程号的位置，删除
port 6379    #端口号
dir "opt/soft/redis/data"  #工作目录
logfile “6379.log” #日志位置  
#其他全删掉
'''
#在redis目录下新建data目录，用来存放书籍
#启动redis
redis-server config/redis.conf
#查看进程
ps -ef |grep redis-server |grep 6379
#查看日志
cd data
cat 6379.log

```



#### 1.3.3 客户端连接

```python
###客户端连接###
redis-cli -h 127.0.0.1 -p 6379
ping #返回PONG
```

####1.3.4 redis返回值

```python
####redis返回值
状态回复：ping---》PONG
错误回复：hget hello field ---》(error)WRONGTYPE Operation against
整数回复：incr hello---》(integer) 1
字符串回复：get hello---》"world"
多行字符串回复：mget hello foo---》"world" "bar"
```





### 1.4 Redis典型使用场景

缓存系统：

计数器：网站访问量，转发量，评论数

消息队列：发布订阅，阻塞队列实现

排行榜：有序集合

社交网络：很多特性跟社交网络匹配，粉丝数，关注数

实时系统：垃圾邮件处理系统，布隆过滤器



