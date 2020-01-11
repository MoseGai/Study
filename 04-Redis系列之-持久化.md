

## 一 持久化的作用

### 1.1 什么是持久化

redis的所有数据保存在内存中，对数据的更新将异步的保存到硬盘上

### 1.2 持久化的实现方式

```python
快照：某时某刻数据的一个完成备份，
	-mysql的Dump
    -redis的RDB
写日志：任何操作记录日志，要恢复数据，只要把日志重新走一遍即可
	-mysql的 Binlog
    -Hhase的 HLog
    -Redis的 AOF
```

## 二 RDB

### 2.1 什么是RDB

![image-20191226120500154](https://tva1.sinaimg.cn/large/006tNbRwgy1ga9zt9svljj30oo0d644s.jpg)

### 2.2 触发机制-主要三种方式

```python

'''
save(同步)
1 客户端执行save命令----》redis服务端----》同步创建RDB二进制文件
2 会造成redis的阻塞（数据量非常大的时候）
3 文件策略：如果老的RDB存在，会替换老的
4 复杂度 o(n)
'''

'''
bgsave(异步，Backgroud saving started)

1 客户端执行save命令----》redis服务端----》异步创建RDB二进制文件（fork函数生成一个子进程（fork会阻塞reids），执行createRDB，执行成功，返回给reids消息）
2 此时访问redis，会正常响应客户端
3 文件策略：跟save相同，如果老的RDB存在，会替换老的
4 复杂度 o(n)
'''

'''
自动（通过配置）
配置   seconds   changes
save   900        1
save   300        10
save   60         10000
如果60s中改变了1w条数据，自动生成rdb
如果300s中改变了10条数据，自动生成rdb
如果900s中改变了1条数据，自动生成rdb

以上三条符合任意一条，就自动生成rdb，内部使用bgsave
'''

#配置：
save 900 1 #配置一条
save 300 10 #配置一条
save 60 10000 #配置一条
dbfilename dump.rdb  #rdb文件的名字，默认为dump.rdb
dir ./ #rdb文件存在当前目录

stop-writes-on-bgsave-error yes #如果bgsave出现错误，是否停止写入，默认为yes
rdbcompression yes #采用压缩格式
rdbchecksum yes #是否对rdb文件进行校验和检验

#最佳配置
save 900 1 
save 300 10 
save 60 10000 
dbfilename dump-${port}.rdb  #以端口号作为文件名，可能一台机器上很多reids，不会乱
dir /bigdiskpath #保存路径放到一个大硬盘位置目录
stop-writes-on-bgsave-error yes #出现错误停止
rdbcompression yes #压缩
rdbchecksum yes #校验
```

### 2.3 触发机制-不容忽略的方式

```python
1 全量复制 #没有执行save和bgsave没有添加rdb策略，还会生成rdb文件，如果开启主从复制，主会自动生成rdb
2 debug reload #debug级别的重启，不会将内存中的数据清空
3 shutdown save#关闭会出发rdb的生成
```

### 2.4 试验

```python

```

### 三 AOF

### 3.1 RDB问题

耗时，耗性能：

不可控，可能会丢失数据

### 3.2 AOF介绍

客户端每写入一条命令，都记录一条日志，放到日志文件中，如果出现宕机，可以将数据完全恢复

### 3.3 AOF的三种策略

日志不是直接写到硬盘上，而是先放在缓冲区，缓冲区根据一些策略，写到硬盘上

always：redis--》写命令刷新的缓冲区---》每条命令fsync到硬盘---》AOF文件

everysec（默认值）：redis——》写命令刷新的缓冲区---》每秒把缓冲区fsync到硬盘--》AOF文件

no:redis——》写命令刷新的缓冲区---》操作系统决定，缓冲区fsync到硬盘--》AOF文件

| 命令 | always                            | everysec                   | no     |
| ---- | --------------------------------- | -------------------------- | ------ |
| 优点 | 不丢失数据                        | 每秒一次fsync，丢失1秒数据 | 不用管 |
| 缺点 | IO开销大，一般的sata盘只有几百TPS | 丢1秒数据                  | 不可控 |

### 3.4 AOF 重写

随着命令的逐步写入，并发量的变大， AOF文件会越来越大，通过AOF重写来解决该问题

| 原生AOF                                                      | AOF重写                                                 |
| ------------------------------------------------------------ | ------------------------------------------------------- |
| set hello world<br/>set hello java<br/>set hello hehe<br/>incr counter<br/>incr counter<br/>rpush mylist a<br/>rpush mylist b<br/>rpush mylist c<br/>过期数据 | set hello hehe<br/>set counter 2<br/>rpush mylist a b c |

本质就是把过期的，无用的，重复的，可以优化的命令，来优化

这样可以减少磁盘占用量，加速恢复速度

#### 实现方式

bgrewriteaof：

客户端向服务端发送bgrewriteaof命令，服务端会起一个fork进程，完成AOF重写

#### AOF重写配置：

| 配置名                      | 含义                |
| --------------------------- | ------------------- |
| auto-aof-rewrite-min-size   | AOF文件重写需要尺寸 |
| auto-aof-rewrite-percentage | AOF文件增长率       |

| 统计名           | 含义                                  |
| ---------------- | ------------------------------------- |
| aof_current_size | AOF当前尺寸（单位：字节）             |
| aof_base_size    | AOF上次启动和重写的尺寸（单位：字节） |

自动触发时机（两个条件同时满足）：

aof_current_size>auto-aof-rewrite-min-size：当前尺寸大于重写需要尺寸

(aof_current_size-aof_base_size)/aof_base_size>auto-aof-rewrite-percentage:（增长率）当前尺寸减去上次重写的尺寸，除以上次重写的尺寸如果大于配置中的增长率

#### 重写流程

![image-20191229185839519](https://tva1.sinaimg.cn/large/006tNbRwgy1gadsmknx2sj30fy0hw78l.jpg)

#### 配置

```python
appendonly yes #将该选项设置为yes，打开
appendfilename "appendonly-${port}.aof" #文件保存的名字
appendfsync everysec #采用第二种策略
dir /bigdiskpath #存放的路径
no-appendfsync-on-rewrite yes #在aof重写的时候，是否要做aof的append操作，因为aof重写消耗性能，磁盘消耗，正常aof写磁盘有一定的冲突，这段期间的数据，允许丢失
```

### 3.5 AOF 重写演示

```python

```

## 四 RDB和AOF的选择

### 4.1 rdb和aof的比较

| 命令       | rdb    | aof                           |
| ---------- | ------ | ----------------------------- |
| 启动优先级 | 低     | 高(挂掉重启，会加载aof的数据) |
| 体积       | 小     | 大                            |
| 恢复速度   | 快     | 慢                            |
| 数据安全性 | 丢数据 | 根据策略决定                  |
| 轻重       | 重     | 轻                            |

### 4.2  rdb最佳策略

rdb关掉，主从操作时

集中管理：按天，按小时备份数据

主从配置，从节点打开

### 4.3 aof最佳策略

开：缓存和存储，大部分情况都打开，

aof重写集中管理

everysec：通过每秒刷新的策略

### 4.4 最佳策略

小分片：每个redis的最大内存为4g

缓存或存储：根据特性，使用不通策略

时时监控硬盘，内存，负载网络等

有足够内存





