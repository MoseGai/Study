## 一 子进程开销和优化

1 cpu

开销：rdb和aof文件生成，属于cpu密集型

优化：不做cpu绑定，不和cpu密集型的服务一起部署

2 内存

开销：fork内存开销，copy-on-write，

优化：单机部署尽量少重写

 3 硬盘

开销：aof和rdb写入，可以结合分析工具使用

优化：

1 不要和高硬盘负载的服务部署在一起：存储服务，消息队列

2 在aof重写期间，不要对aof进行追加：no-appendfsync-on-rewrite=yes

3 根据写入量决定磁盘类型：例如ssd

4 单机多实例持久化考虑分盘

## 二 fork操作

1 fork是同步操作

2 与内存量嘻嘻相关：内存越大，耗时越长，跟机型也有关系

3 info：latest_fok_usec:查看持久化执行时间

改善fork

1 有限使用无机或高效支持fork操作的虚拟化技术

2 控制redis实例最大可用内存：maxmemory

3 合理配置linux内存分配策略

4 降低fork频率，例如放宽aof重写自动触发时机，不必要的全量复制

## 三 aof追加阻塞

![image-20191229192629198](https://tva1.sinaimg.cn/large/006tNbRwgy1gadtfor4exj30hu0ke434.jpg)

aof阻塞：看日志定位

info Persistence：每次阻塞一次就会+1

