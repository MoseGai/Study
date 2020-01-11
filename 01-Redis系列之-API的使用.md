## 一 通用命令

### 1.1 通用命令

```python
####1-keys 
#打印出所有key
keys * 
#打印出所有以he开头的key
keys he*
#打印出所有以he开头，第三个字母是h到l的范围
keys he[h-l]
#三位长度，以he开头，？表示任意一位
keys he？
#keys命令一般不在生产环境中使用，生产环境key很多，时间复杂度为o(n),用scan命令

####2-dbsize   计算key的总数
dbsize #redis内置了计数器，插入删除值该计数器会更改，所以可以在生产环境使用，时间复杂度是o(1)

###3-exists key 时间复杂度o(1)
#设置a
set a b
#查看a是否存在
exists a
(integer) 1
#存在返回1 不存在返回0
###4-del key  时间复杂度o(1)
删除成功返回1，key不存在返回0
###5-expire key seconds  时间复杂度o(1)
expire name 3 #3s 过期
ttl name  #查看name还有多长时间过期
persist name #去掉name的过期时间
###6-type key  时间复杂度o(1)
type name #查看name类型，返回string
```

### 1.2 数据结构和内部编码

![image-20191224110401405](https://tva1.sinaimg.cn/large/006tNbRwgy1ga7mt5tzjnj30q20pgn5n.jpg)

### 1.3 单线程架构

####1.3.1 单线程架构，

一个瞬间只会执行一条命令

![image-20191224111010657](https://tva1.sinaimg.cn/large/006tNbRwgy1ga7mzj7rpoj30wu0egq5t.jpg)

####1.3.2 单线程为什么这么快

1 纯内存

2 非阻塞IO （epoll），自身实现了事件处理，不在网络io上浪费过多时间

3 避免线程间切换和竞态消耗

#### 1.3.3 注意

1 一次只运行一条命令

2 拒绝长慢命令

​	-keys，flushall,flushdb,慢的lua脚本，mutil/exec，operate，big value

3 其实不是单线程（在做持久化是另外的线程）

​	-fysnc file descriptor

​	-close file descriptor

## 二 字符串类型

### 2.1 字符串键值结构

```python
key          value
hello        world      可以很复杂，如json格式字符串
counter      1          数字类型
bits         10101010   二进制（位图）
#字符串value不能大于512m，一般建议100k以内
#用于缓存，计数器，分布式锁...
```

### 2.2 常用命令

```python
###1---基本使用get，set，del
get name       #时间复杂度 o(1)
set name lqz   #时间复杂度 o(1)
del name       #时间复杂度 o(1)
###2---其他使用incr,decr,incrby,decrby
incr age  #对age这个key的value值自增1
decr age  #对age这个key的value值自减1
incrby age 10  #对age这个key的value值增加10
decrby age 10  #对age这个key的value值减10
#统计网站访问量（单线程无竞争，天然适合做计数器）
#缓存mysql的信息（json格式）
#分布式id生成（多个机器同时并发着生成，不会重复）
###3---set，setnx，setxx
set name lqz  #不管key是否存在，都设置 
setnx name lqz #key不存在时才设置（新增操作）
set name lqz nx #同上
set name lqz xx #key存在，才设置（更新操作）
###4---mget mset
mget key1 key2 key3     #批量获取key1，key2.。。时间复杂度o(n)
mset key1 value1 key2 value2 key3 value3    #批量设置时间复杂度o(n)
#n次get和mget的区别
#n次get时间=n次命令时间+n次网络时间
#mget时间=1次网络时间+n次命令时间
###5---其他：getset，append，strlen
getset name lqznb #设置新值并返回旧值 时间复杂度o(1)
append name 666 #将value追加到旧的value 时间复杂度o(1)
strlen name  #计算字符串长度(注意中文)  时间复杂度o(1)
###6---其他：incrybyfloat,getrange,setrange
increbyfloat age 3.5  #为age自增3.5，传负值表示自减 时间复杂度o(1)
getrange key start end #获取字符串制定下标所有的值  时间复杂度o(1)
setrange key index value #从指定index开始设置value值  时间复杂度o(1)
```



## 三 哈希类型

###3.1 哈希值结构

![image-20191224121323414](https://tva1.sinaimg.cn/large/006tNbRwgy1ga7otagi3lj30p00f2q8l.jpg)



###3.2 重要api

```python
###1---hget,hset,hdel
hget key field  #获取hash key对应的field的value 时间复杂度为 o(1)
hset key field value #设置hash key对应的field的value值 时间复杂度为 o(1)
hdel key field #删除hash key对应的field的值 时间复杂度为 o(1)
#测试
hset user:1:info age 23
hget user:1:info ag
hset user:1:info name lqz
hgetall user:1:info
hdel user:1:info age
###2---hexists,hlen
hexists key field  #判断hash key 是否存在field 时间复杂度为 o(1)
hlen key   #获取hash key field的数量  时间复杂度为 o(1)
hexists user:1:info name
hlen user:1:info  #返回数量
        
###3---hmget，hmset
hmget key field1 field2 ...fieldN  #批量获取hash key 的一批field对应的值  时间复杂度是o(n)
hmset key field1 value1 field2 value2  #批量设置hash key的一批field value 时间复杂度是o(n)

###4--hgetall,hvals，hkeys
hgetall key  #返回hash key 对应的所有field和value  时间复杂度是o(n)
hvals key   #返回hash key 对应的所有field的value  时间复杂度是o(n)
hkeys key   #返回hash key对应的所有field  时间复杂度是o(n)
###小心使用hgetall
##1 计算网站每个用户主页的访问量
hincrby user:1:info pageview count
##2 缓存mysql的信息，直接设置hash格式


```



###3.3 hash vs string

####3.3.1相似的api

| get                     | hget         |
| ----------------------- | ------------ |
| set /sentnx             | hset  hsetnx |
| del                     | hdel         |
| incr incrby dear decrby | hincrby      |
| mset                    | hmset        |
| mget                    | hmget        |

#### 3.3.2 缓存三种方案

直接json格式字符串

每个字段一个key

使用hash操作

###3.4 其他操作

```python
##其他操作 hsetnx，hincrby，hincrbyfloat
hestnx key field value #设置hash key对应field的value（如果field已存在，则失败），时间复杂度o(1)
hincrby key field intCounter #hash key 对英的field的value自增intCounter 时间复杂度o(1)
hincrbyfloat key field floatCounter #hincrby 浮点数 时间复杂度o(1)

```

## 四 列表类型

### 4.1 列表特点

有序队列，可以从左侧添加，右侧添加，可以重复，可以从左右两边弹出

#### 4.2 API操作

#####4.2.1 插入操作

```python
#rpush 从右侧插入
rpush key value1 value2 ...valueN  #时间复杂度为o(1~n)
#lpush 从左侧插入
#linsert
linsert key befort|after value newValue   #从元素value的前或后插入newValue 时间复杂度o(n) ，需要便利列表
linsert listkey before b java
linsert listkey after b php
```

#####4.2.2 删除操作

```python
lpop key #从列表左侧弹出一个item 时间复杂度o(1)

rpop key #从列表右侧弹出一个item 时间复杂度o(1)

lrem key count value
#根据count值，从列表中删除所有value相同的项 时间复杂度o(n)
1 count>0 从左到右，删除最多count个value相等的项
2 count<0 从右向左，删除最多 Math.abs(count)个value相等的项
3 count=0 删除所有value相等的项
lrem listkey 0 a #删除列表中所有值a
lrem listkey -1 c #从右侧删除1个c

ltrim key start end #按照索引范围修剪列表 o(n)
ltrim listkey 1 4 #只保留下表1--4的元素
```

#### 4.2.3 查询操作

```python
lrange key start end #包含end获取列表指定索引范围所有item  o(n)
lrange listkey 0 2
lrange listkey 1 -1 #获取第一个位置到倒数第一个位置的元素

lindex key index #获取列表指定索引的item  o(n)
lindex listkey 0
lindex listkey -1

llen key #获取列表长度
```

#### 4.2.3 修改操作

```python
lset key index newValue #设置列表指定索引值为newValue o(n)
lset listkey 2 ppp #把第二个位置设为ppp
```



### 4.3 实战

实现timeLine功能，时间轴，微博关注的人，按时间轴排列，在列表中放入关注人的微博的即可

### 4.4 其他操作

```python
blpop key timeout #lpop的阻塞版，timeout是阻塞超时时间，timeout=0为拥有不阻塞 o(1)
brpop key timeout #rpop的阻塞版，timeout是阻塞超时时间，timeout=0为拥有不阻塞 o(1)

#要实现栈的功能
lpush+lpop
#实现队列功能
lpush+rpop
#固定大小的列表
lpush+ltrim
#消息队列
lpush+brpop
```

## 五 集合类型

### 5.1 特点

无序，无重复，集合间操作（交叉并补） 

### 5.2 API操作

```python
sadd key element #向集合key添加element（如果element存在，添加失败） o(1)

srem key element #从集合中的element移除掉 o(1)

scard key #计算集合大小

sismember key element #判断element是否在集合中

srandmember key count #从集合中随机取出count个元素，不会破坏集合中的元素

spop key #从集合中随机弹出一个元素

smembers key #获取集合中所有元素 ，无序，小心使用，会阻塞住 

sdiff user:1:follow user:2:follow  #计算user:1:follow和user:2:follow的差集

sinter user:1:follow user:2:follow  #计算user:1:follow和user:2:follow的交集
          
sunion user:1:follow user:2:follow  #计算user:1:follow和user:2:follow的并集
                
sdiff|sinter|suion + store destkey... #将差集，交集，并集结果保存在destkey集合中
```

### 5.3 实战

抽奖系统 ：通过spop来弹出用户的id，活动取消，直接删除

点赞，点猜，喜欢等，用户如果点了赞，就把用户id放到该条记录的集合中

标签：给用户/文章等添加标签，sadd user:1:tags 标签1 标签2 标签3

给标签添加用户，关注该标签的人有哪些

共同好友：集合间的操作

#### 5.4 总结

sadd:可以做标签相关

spop/srandmember:可以做随机数相关

sadd/sinter：社交相关

## 六 有序集合类型

### 6.1 特点

```python
#有一个分值字段，来保证顺序
key                  score                value
user:ranking           1                   lqz
user:ranking           99                  lqz2
user:ranking           88                  lqz3
#集合有序集合
集合：无重复元素，无序，element
有序集合：无重复元素，有序，element+score
#列表和有序集合
列表：可以重复，有序，element
有序集合：无重复元素，有序，element+score
```

### 6.2 API使用

```python
zadd key score element #score可以重复，可以多个同时添加，element不能重复 o(logN) 

zrem key element #删除元素，可以多个同时删除 o(1)

zscore key element #获取元素的分数 o(1)

zincrby key increScore element #增加或减少元素的分数  o(1)

zcard key #返回元素总个数 o(1)

zrank key element #返回element元素的排名（从小到大排）

zrange key 0 -1 #返回排名，不带分数  o(log(n)+m) n是元素个数，m是要获取的值
zrange player:rank 0 -1 withscores #返回排名，带分数

zrangebyscore key minScore maxScore #返回指定分数范围内的升序元素 o(log(n)+m) n是元素个数，m是要获取的值
zrangebyscore user:1:ranking 90 210 withscores #获取90分到210分的元素

zcount key minScore maxScore #返回有序集合内在指定分数范围内的个数 o(log(n)+m)

zremrangebyrank key start end #删除指定排名内的升序元素 o(log(n)+m)
zremrangebyrank user:1:rangking 1 2 #删除升序排名中1到2的元素
        
zremrangebyscore key minScore maxScore #删除指定分数内的升序元素 o(log(n)+m)
zremrangebyscore user:1:ranking 90 210 #删除分数90到210之间的元素

```

### 6.3 实战

排行榜：音乐排行榜，销售榜，关注榜，游戏排行榜

### 6.4 其他操作

```python
zrevrank #从高到低排序
zrevrange #从高到低排序取一定范围
zrevrangebyscore #返回指定分数范围内的降序元素
zinterstore #对两个有序集合交集
zunionstore #对两个有序集合求并集
```

###6.5 总结

| 操作类型 | 命令                                              |
| -------- | ------------------------------------------------- |
| 基本操作 | zadd/  zrem/  zcard/  zincrby/  zscore            |
| 范围操作 | zrange/  zrangebyscore/  zcount/  zremrangebyrank |
| 集合操作 | zunionstore/  zinterstore                         |















