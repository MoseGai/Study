## 一 什么是主从复制

机器故障；容量瓶颈；QPS瓶颈

一主一从，一主多从

做读写分离

做数据副本

扩展数据性能

一个maskter可以有多个slave

一个slave只能有一个master

数据流向是单向的，从master到slave

##  二 复制的 配置

### 2.1 slave 命令

```python
6380是从，6379是主

在6389上执行

slave of 127.0.0.1 6379 #异步
slaveof no one #取消复制，不会把之前的数据清除
```

### 2.2 配置文件

```python
slaveof ip port #配置从节点ip和端口
slave-read-only yes #从节点只读，因为可读可写，数据会乱

'''
mkdir -p redis1/conf redis1/data redis2/conf redis2/data redis3/conf redis3/data
vim redis.conf

daemonize no
pidfile redis.pid
bind 0.0.0.0
protected-mode no
port 6379
timeout 0
logfile redis.log
dbfilename dump.rdb
dir /data
slaveof 10.0.0.101 6379
slave-read-only yes


cp redis.conf /home/redis2/conf/


docker run -p 6379:6379 --name redis_6379 -v /home/redis1/conf/redis.conf:/etc/redis/redis.conf -v /home/redis1/data:/data -d redis redis-server /etc/redis/redis.conf

docker run -p 6378:6379 --name redis_6378 -v /home/redis2/conf/redis.conf:/etc/redis/redis.conf -v /home/redis2/data:/data -d redis redis-server /etc/redis/redis.conf

docker run -p 6377:6379 --name redis_6377 -v /home/redis3/conf/redis.conf:/etc/redis/redis.conf -v /home/redis3/data:/data -d redis redis-server /etc/redis/redis.conf

info replication

'''
```



## 四 故障处理

slave故障

master故障

## 五 复制常见问题

1 读写分离

读流量分摊到从节点

可能遇到问题：复制数据延迟，读到过期数据，从节点故障

2 主从配置不一致

maxmemory不一致：丢失数据

数据结构优化参数：主节点做了优化，从节点没有设置优化，会出现一些问题

3 规避全量复制

第一次全量复制，不可避免：小主节点，低峰(夜间)

节点运行id不匹配：主节点重启(运行id变化)

复制挤压缓冲区不足：增大复制缓冲区大小，rel_backlog_size

4 规避复制风暴

单主节点复制风暴，主节点重启，所有从节点复制