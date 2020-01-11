## 一 慢查询

### 1.1 生命周期

我们配置一个时间，如果查询时间超过了我们设置的时间，我们就认为这是一个慢查询.

慢查询发生在第三阶段

客户端超时不一定慢查询，但慢查询是客户端超时的一个可能因素



![image-20191225102102218](https://tva1.sinaimg.cn/large/006tNbRwgy1ga8r6p7tpej30z80k2afq.jpg)

### 1.2 两个配置

#### 1.2.1 slowlog-max-len

慢查询是一个先进先出的队列

固定长度

保存在内存中

#### 1.2.2 slowlog-max-len

慢查询阈值（单位：微秒）

slowlog-log-slower-than=0，记录所有命令

slowlog-log-slower-than <0,不记录任何命令

#### 1.2.3 配置方法

**1 默认配置**

config get slowlog-max-len=128

Config get slowly-log-slower-than=10000

**2 修改配置文件重启**

**3 动态配置**

```python
# 设置记录所有命令
config set slowlog-log-slower-than 0
# 最多记录100条
config set slowlog-max-len 100
# 持久化到本地配置文件
config rewrite

'''
config set slowlog-max-len 1000
config set slowlog-log-slower-than 1000
'''
```

### 1.3 三个命令

```python
slowlog get [n]  #获取慢查询队列
'''
日志由4个属性组成：
1）日志的标识id
2）发生的时间戳
3）命令耗时
4）执行的命令和参数
'''

slowlog len #获取慢查询队列长度

slowlog reset #清空慢查询队列

```

### 1.4 经验

```python
1 slowlog-max-len 不要设置过大，默认10ms，通常设置1ms
2 slowlog-log-slower-than不要设置过小，通常设置1000左右
3 理解命令生命周期
4 定期持久化慢查询
```

## 二 pipeline

### 2.1 什么是pipeline(管道)

Redis的pipeline(管道)功能在命令行中没有，但redis是支持pipeline的，而且在各个语言版的client中都有相应的实现



将一批命令，批量打包，在redis服务端批量计算(执行)，然后把结果批量返回

1次pipeline(n条命令)=1次网络时间+n次命令时间

```python
pipeline期间将“独占”链接，此期间将不能进行非“管道”类型的其他操作，直到pipeline关闭；如果你的pipeline的指令集很庞大，为了不干扰链接中的其他操作，你可以为pipeline操作新建Client链接，让pipeline和其他正常操作分离在2个client中。不过pipeline事实上所能容忍的操作个数，和socket-output缓冲区大小/返回结果的数据尺寸都有很大的关系；同时也意味着每个redis-server同时所能支撑的pipeline链接的个数，也是有限的，这将受限于server的物理内存或网络接口的缓冲能力
```

### 2.2 客户端实现

```python
import redis
pool = redis.ConnectionPool(host='10.211.55.4', port=6379)
r = redis.Redis(connection_pool=pool)
# pipe = r.pipeline(transaction=False)
#创建pipeline
pipe = r.pipeline(transaction=True)
#开启事务
pipe.multi()
pipe.set('name', 'lqz')
pipe.set('role', 'nb')
 
pipe.execute()
```

### 2.3 与原生操作对比

```python
通过pipeline提交的多次命令，在服务端执行的时候，可能会被拆成多次执行，而mget等操作，是一次性执行的，所以，pipeline执行的命令并非原子性的
```

### 2.4 使用建议

1 注意每次pipeline携带的数据量

2 pipeline每次只能作用在一个Redis的节点上

3 M(mset，mget....)操作和pipeline的区别



## 三 发布订阅

### 3.1 角色

**发布者/订阅者/频道**

发布者发布了消息，所有的订阅者都可以收到，就是生产者消费者模型（后订阅了，无法获取历史消息）

### 3.2 模型

![image-20191225163659941](https://tva1.sinaimg.cn/large/006tNbRwgy1ga923qyr2uj31xp0u0jwt.jpg)

### 3.3 API

```python
publish channel message #发布命令
publish souhu:tv "hello world" #在souhu:tv频道发布一条hello world  返回订阅者个数

subscribe [channel] #订阅命令，可以订阅一个或多个
subscribe sohu:tv  #订阅sohu:tv频道

unsubscribe [channel] #取消订阅一个或多个频道
unsubscribe sohu:tv  #取消订阅sohu:tv频道
    
psubscribe [pattern...] #订阅模式匹配
psubscribe c*  #订阅以c开头的频道

unpsubscribe [pattern...] #按模式退订指定频道

pubsub channels #列出至少有一个订阅者的频道,列出活跃的频道

pubsub numsub [channel...] #列出给定频道的订阅者数量

pubsub numpat #列出被订阅模式的数量
```



### 3.4 发布订阅和消息队列

发布订阅数全收到，消息队列有个抢的过程，只有一个抢到

## 四 Bitmap位图

### 4.1 位图是什么

下面是字符串big对应的二进制（b是98）

![image-20191225172053447](https://tva1.sinaimg.cn/large/006tNbRwgy1ga93bk259dj313y0isajp.jpg)



### 4.2 相关命令
```python
set hello big #放入key位hello 值为big的字符串
getbit hello 0 #取位图的第0个位置，返回0
getbit hello 1 #取位图的第1个位置，返回1 如上图

##我们可以直接操纵位
setbit key offset value #给位图指定索引设置值
setbit hello 7 1 #把hello的第7个位置设为1 这样，big就变成了cig

setbit test 50 1 #test不存在，在key为test的value的第50位设为1，那其他位都以0补

bitcount key [start end] #获取位图指定范围(start到end,单位为字节,注意按字节一个字节8个bit为，如果不指定就是获取全部)位值为1的个数

bitop op destkey key [key...] #做多个Bitmap的and(交集)/or(并集)/not(非)/xor(异或)，操作并将结果保存在destkey中 
bitop and after_lqz lqz lqz2 #把lqz和lqz2按位与操作，放到after_lqz中

bitpos key targetBit start end #计算位图指定范围(start到end，单位为字节，如果不指定是获取全部)第一个偏移量对应的值等于targetBit的位置
bitpos lqz 1 #big 对应位图中第一个1的位置，在第二个位置上，由于从0开始返回1
bitpos lqz 0 #big 对应位图中第一个0的位置，在第一个位置上，由于从0开始返回0
bitpos lqz 1 1 2 #返回9：返回从第一个字节到第二个字节之间 第一个1的位置，看上图，为9
```

![image-20191225172547661](https://tva1.sinaimg.cn/large/006tNbRwgy1ga93gnif6ej310q0iuwpy.jpg)

### 4.3 独立用户统计

1 使用set和Bitmap对比

2 1亿用户，5千万独立（1亿用户量，约5千万人访问，统计活跃用户数量）

| 数据类型 | 每个userid占用空间             | 需要存储用户量 | 全部内存量       |
| -------- | ------------------------------ | -------------- | ---------------- |
| set      | 32位(假设userid是整形，占32位) | 5千万          | 32位*5千万=200MB |
| bitmap   | 1位                            | 1亿            | 1位*1亿=12.5MB   |

假设有10万独立用户，使用位图还是占用12.5mb，使用set需要32位*1万=4MB

### 4.5 总结

1 位图类型是string类型，最大512M

2 使用setbit时偏移量如果过大，会有较大消耗

3 位图不是绝对好用，需要合理使用

## 五 HyperLogLog

### 5.1 介绍

基于HyperLogLog算法：绩效的空间完成独立数量统计

本质还是字符串

### 5.2 三个命令

```python
pfadd key element #向hyperloglog添加元素,可以同时添加多个
pfcount key #计算hyperloglog的独立总数
pfmerge destroy sourcekey1 sourcekey2#合并多个hyperloglog，把sourcekey1和sourcekey2合并为destroy

pfadd uuids "uuid1" "uuid2" "uuid3" "uuid4" #向uuids中添加4个uuid
pfcount uuids #返回4
pfadd uuids "uuid1" "uuid5"#有一个之前存在了，其实只把uuid5添加了
pfcount uuids #返回5

pfadd uuids1 "uuid1" "uuid2" "uuid3" "uuid4"
pfadd uuids2 "uuid3" "uuid4" "uuid5" "uuid6"
pfmerge uuidsall uuids1 uuids2 #合并
pfcount uuidsall #统计个数 返回6
```

### 5.3 内存消耗&总结

百万级别独立用户统计，百万条数据只占15k

错误率 0.81%

无法取出单条数据，只能统计个数

## 六 GEO

### 6.1 介绍

GEO（地理信息定位）：存储经纬度，计算两地距离，范围等

北京：116.28，39.55

天津：117.12，39.08

可以计算天津到北京的距离，天津周围50km的城市，外卖等

### 6.2 5个城市纬度
| 城市   | 经度   | 纬度  | 简称         |
| ------ | ------ | ----- | ------------ |
| 北京   | 116.28 | 39.55 | beijing      |
| 天津   | 117.12 | 39.08 | tianjin      |
| 石家庄 | 114.29 | 38.02 | shijiazhuang |
| 唐山   | 118.01 | 39.38 | tangshan     |
| 保定   | 115.29 | 38.51 | baoding      |

### 6.3 相关命令

```python
geoadd key longitude latitude member #增加地理位置信息
geoadd cities:locations 116.28 39.55 beijing #把北京地理信息天津到cities:locations中
geoadd cities:locations 117.12 39.08 tianjin
geoadd cities:locations 114.29 38.02 shijiazhuang
geoadd cities:locations 118.01 39.38 tangshan
geoadd cities:locations 115.29 38.51 baoding
    
geopos key member #获取地理位置信息
geopos cities:locations beijing #获取北京地理信息

geodist key member1 member2 [unit]#获取两个地理位置的距离 unit:m(米) km(千米) mi(英里) ft(尺)
geodist cities:locations beijing tianjin km #北京到天津的距离，89公里

georadius key logitude latitude radiusm|km|ft|mi [withcoord] [withdist] [withhash] [COUNT count] [asc|desc] [store key][storedist key]

georadiusbymember key member radiusm|km|ft|mi [withcoord] [withdist] [withhash] [COUNT count] [asc|desc] [store key][storedist key]
#获取指定位置范围内的地理位置信息集合
'''
withcoord:返回结果中包含经纬度
withdist：返回结果中包含距离中心节点位置
withhash：返回解雇中包含geohash
COUNT count：指定返回结果的数量
asc|desc：返回结果按照距离中心店的距离做升序/降序排列
store key：将返回结果的地理位置信息保存到指定键
storedist key：将返回结果距离中心点的距离保存到指定键
'''
georadiusbymember cities:locations beijing 150 km
'''
1) "beijing"
2) "tianjin"
3) "tangshan"
4) "baoding"
'''
```



### 6.4 总结

3.2以后版本才有

geo本质时zset类型

可以使用zset的删除，删除指定member：zrem 

```
  geoadd key longitude latitude member #增加地理位置信息
  geoadd cities:locations 116.28 39.55 beijing #把北京地理信息天津到cities:locations中
  geoadd cities:locations 117.12 39.08 tianjin
  geoadd cities:locations 114.29 38.02 shijiazhuang
  geoadd cities:locations 118.01 39.38 tangshan
  geoadd cities:locations 115.29 38.51 baoding
      
  geopos key member #获取地理位置信息
  geopos cities:locations beijing #获取北京地理信息
  
  geodist key member1 member2 [unit]#获取两个地理位置的距离 unit:m(米) km(千米) mi(英里) ft(尺)
  geodist cities:locations beijing tianjin km #北京到天津的距离，89公里
  
  georadius key logitude latitude radiusm|km|ft|mi [withcoord] [withdist] [withhash] [COUNT count] [asc|desc] [store key][storedist key]
  
  georadiusbymember key member radiusm|km|ft|mi [withcoord] [withdist] [withhash] [COUNT count] [asc|desc] [store key][storedist key]
  #获取指定位置范围内的地理位置信息集合
  '''
  withcoord:返回结果中包含经纬度
  withdist：返回结果中包含距离中心节点位置
  withhash：返回解雇中包含geohash
  COUNT count：指定返回结果的数量
  asc|desc：返回结果按照距离中心店的距离做升序/降序排列
  store key：将返回结果的地理位置信息保存到指定键
  storedist key：将返回结果距离中心点的距离保存到指定键
  '''
  georadiusbymember cities:locations beijing 150 km
  '''
  1) "beijing"
  2) "tianjin"
  3) "tangshan"
  4) "baoding"
  '''
```



### 6.4 总结

3.2以后版本才有

geo本质时zset类型

可以使用zset的删除，删除指定member：zrem 

```
    geoadd key longitude latitude member #增加地理位置信息
    geoadd cities:locations 116.28 39.55 beijing #把北京地理信息天津到cities:locations中
    geoadd cities:locations 117.12 39.08 tianjin
    geoadd cities:locations 114.29 38.02 shijiazhuang
    geoadd cities:locations 118.01 39.38 tangshan
    geoadd cities:locations 115.29 38.51 baoding
        
    geopos key member #获取地理位置信息
    geopos cities:locations beijing #获取北京地理信息
    
    geodist key member1 member2 [unit]#获取两个地理位置的距离 unit:m(米) km(千米) mi(英里) ft(尺)
    geodist cities:locations beijing tianjin km #北京到天津的距离，89公里
    
    georadius key logitude latitude radiusm|km|ft|mi [withcoord] [withdist] [withhash] [COUNT count] [asc|desc] [store key][storedist key]
    
    georadiusbymember key member radiusm|km|ft|mi [withcoord] [withdist] [withhash] [COUNT count] [asc|desc] [store key][storedist key]
    #获取指定位置范围内的地理位置信息集合
    '''
    withcoord:返回结果中包含经纬度
    withdist：返回结果中包含距离中心节点位置
    withhash：返回解雇中包含geohash
    COUNT count：指定返回结果的数量
    asc|desc：返回结果按照距离中心店的距离做升序/降序排列
    store key：将返回结果的地理位置信息保存到指定键
    storedist key：将返回结果距离中心点的距离保存到指定键
    '''
    georadiusbymember cities:locations beijing 150 km
    '''
    1) "beijing"
    2) "tianjin"
    3) "tangshan"
    4) "baoding"
    '''
    
```



### 6.4 总结

3.2以后版本才有

geo本质时zset类型

可以使用zset的删除，删除指定member：zrem cities:locations beijing





