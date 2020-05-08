#Redis 深度历险

## 开篇

一、为什么要使用redis，redis使用的场景有哪些

首先使用redis是为了实现传统的关系型数据库不易实现的东西，或者读取数据库比较耗费资源的问题。常见的场景如下：

- 一些需要在页面上频繁展示，但是数据表动不大的场景，诸如一些固定不变的配置
- 一些关系型数据库不易实现的东西，如排行榜
- 实现一些计数操作、API请求限流。
- 好友关系，共同好友
- session 共享机制。
- 简单的消息队列。
- 分布式锁



## 基本数据结构

### string (字符串)

一、知识点

- value 值是有多个字节做成的，每个字节，8位。依然区分值的类型是int 还是字符串。
- 空间分配采用的是冗余分配的方法，和golang中的类似，都是caption/len的格式。1M以下，长度不足时，增加一倍。1M以上时，每次增加1M。
- 字符串的最大长度是512M。

二、基本命令

键值对

```redis
set key value [EX seconds] [PX milliseconds] [NX|XX]
get key
mset key value [key value ...]
mget key [key ...]
exists key [key ...]		//只要有一个存在时，即返回1
del key [key ...]		//只要有一个存在时，即返回1
setnx key value		//key 不存在是设置成功
```

过期时间

```
expire key seconds
setex key seconds value
```

计数

```
incr key		//自增1
incrby key increment	//自增指定步长
```



### list (链表)

一、知识点

- list是一个链表的结构，故插入和删除都非常快，时间复杂度为O(1)。个人理解，因为只是从一次插入或者删除数据，无需其他操作，故时间复杂度为O(1)。
- list的索引定位比较慢，是一个O(n)的操作。
- 使用 list 一侧进一侧处，可以达到队列的效果。同侧进同侧出可以实现栈的效果。
- 当list 弹出最后一个元素的时候，数据结构自动删除，内存被回收。
- 当 list 中的数据比较少的时候，使用一块连续的内存来存储，数据格式ziplist。当数据量大的时候，使用链表和 ziplist 形成 quicklist 的数据格式

二、基本命令

插入删除数据

```
	rpush key value [value ...]	//插入元素
	lpop key		//弹出元素
	lrange key start stop	//查看list
```

慢操作

```
lindex key index
ltrim key start stop	//截断list,得到[start, stop]的闭区间元素
```



### hash (哈希)

一、知识点

- hash 为无序字典
- hash 只能存储字符串
- 什么叫做rehash：若最开始hash结果使用的内存空间为M1，当数据量变大时分配新的空间M2，此时需要将M1空间内的数据重写入到M2，此过程称为rehash。
- rehash策略：Redis为了高兴能，不堵塞服务采用了渐进式rehash策略。在进行rehash的时候保留两个hash结构，查询时新旧两个结果都会查询。当hash移动完最后一个元素的时候，数据会被删除，内存会被回收。

二、基本命令

```
hset key field value	//第一次设置变量返回 (int)1，更新时返回0
hmset key field value [field value ...]	//返回ok
hget key field
hgetall key
hincrby key field increment	//对字典值进行自增操作
hdel key field [field ...]	//删除hash 中的key
```



###set (集合)

一、知识点

- 内部值无序唯一
- 特殊的字典值，所有key的值为null

二、基本命令

```
sadd key member [member ...]
scard key		//获取当前集合的长度
smembers key	//获取集合数据
sismember key member	//查询是否为集合数据
spop key [count]	//移除元素 count 个
```



### zset (有序集合)

一、知识点

- zset 有集合特征，保证了那不value的唯一性
- zset 可以为每一个value分配权重即score
- zset的数据结构为跳跃列表

二、基本命令

```
zadd books 10 php //添加元素
zrem books php 	//移除元素
zcard books 		//count(*)
zrange books 0 -1	//获取全部元素，根据分数正序排列
zrevrange books 0 -1 //获取全部元素，根据分数倒序排序
zrange books 0 -1 withscores //带分数展示
zscore books php	//获取分数
zrangebyscore books 0 9 [withscores]	//根据分数范围展示

```



### 容器型数据类型通用规则

list/hash/set/zset 都属于容器型数据类型，有如下特点：

- create if not exists
- drop if no elements

### 过期时间

- 所有数据类型都可以设置过期时间
- 字符串类型重新修改后过期时间失效
- ttl key 可用用来查查剩余多长时间过期



## 应用

### 分布式锁

一、为什么需要分布式锁

当存在多个线程需要读取、修改redis中的数据时，因为读取和修改不是原子操作，此时就会出现并发问题，如下图所示：

![avatar](/Users/iamcyan/Desktop/iamcyan/note/redis/redis 深度历险/分布式锁.png)

原子操作：不会被线程调度机制打断的操作。才做一旦开始执行，一直运行到结束，不会有 context switch 线程切换。

二、分布式锁

方案1：开始时加锁，结束时释放锁

```
setnx lock_something true
...do something
del lock_something
```

问题：如果在执行中，线程挂掉，那么就会造成死锁的现象。

解决办法：给变量设置一个过期时间



方案二、加锁后设置过期时间

```
setnx lock_something true
expire lock_somethins 5
...do something
del lock_something
```

问题：因为枷锁和设置过期时间属于两个指令，当setnx 执行完成之后，如果线程断掉，那么同样会造成死锁

解决方法：让setnx 和 expire 操作同时完成，并且属于原子操作。



方案三

```
set lock_something true ex 5 nx	//设置的同步设置过期时间，该操作为原子操作
... do something
del lock_something
```

问题：如果业务逻辑执行时间超时，那么在key 会自动过期。其他线程会进行加锁

解决方法：在将lock_something 的值设置为一个随机数字，在del 前验证随机数字



方案四：设置随机数字

1、Redis 的分布是锁不用于执行耗时较长的任务，如果偶发，手动修复。

2、一个稍微安全的方案：为 set 指令的 value 参数设置一个随机数，释放锁是匹配随机数

```
set lock_something avg(1) ex 5 nx	//设置的同步设置过期时间，该操作为原子操作
... do something
get lock_something

del lock_something
```

问题：因为验证随机数和删除key 并不是在一个原子操作内完成的，如果需要支持这样的操作，还得需要使用诸如lua的脚本来处理。因为lua可以保证多个连续指令的原子操作。



### 延时队列

一、redis队列可以达到一个什么样的水平

- 如果对于只有一个消费者的队列，可以使用redis 进行队列实现。
- redis 队列不是专业的队列，没有ack支持，对队列有极致要求的不应该使用redis 作为队列。

二、异步消息队列

方案1：队列的写入读取

```
lpush delay_list value1
rpop 
```

问题：如果队列空了，rpop会使队列陷入循环，拉高CPU使用、QPS。



方案2：队列为空时sleep(1)

```
lpush delay_list value1
rpop 
if empty sleep(1)
```

问题：此处sleep(1)，有可能会对消息的延迟增加1s。多个线程或进程的时候，因为sleep会岔开，延时会有降低。



方案3：使用 brpop 阻塞读取

```
lpush delay_list value1
brpop 
```

注意：空闲断开连接的时候，会产生异常，代码编写时需要捕获异常。



三、延时队列

redis 在分布式锁加锁失败的情况下，几种处理方法
1. 直接抛出异常，客户稍后重试
2. sleep 后重试
3. 将请求转移到延时队列，稍后重试

延时队列实现

延时队列可以通过 Redis 的 zset(有序列表) 来实现。我们将消息序列化成一个字符串作为 zset 的`value`，这个消息的到期处理时间作为`score`，然后用多个线程轮询 zset 获取到期的任务进行处理，多个线程是为了保障可用性，万一挂了一个线程还有其它线程可以继续处理。因为有多个线程，所以需要考虑并发争抢任务，确保任务不能被多次执行

```python
def delay(msg):
    msg.id = str(uuid.uuid4())  # 保证 value 值唯一
    value = json.dumps(msg)
    retry_ts = time.time() + 5  # 5 秒后重试
    redis.zadd("delay-queue", retry_ts, value)


def loop():
    while True:
        # 最多取 1 条
        values = redis.zrangebyscore("delay-queue", 0, time.time(), start=0, num=1)
        if not values:
            time.sleep(1)  # 延时队列空的，休息 1s
            continue
        value = values[0]  # 拿第一条，也只有一条
        success = redis.zrem("delay-queue", value)  # 从消息队列中移除该消息
        if success:  # 因为有多进程并发的可能，最终只会有一个进程可以抢到消息
            msg = json.loads(value)
            handle_msg(msg)			
```

Redis 的 zrem 方法是多线程多进程争抢任务的关键，它的返回值决定了当前实例有没有抢到任务，因为 loop 方法可能会被多个线程、多个进程调用，同一个任务可能会被多个进程线程抢到，通过 zrem 来决定唯一的属主。同时，我们要注意一定要对 handle_msg 进行异常捕获，避免因为个别任务处理问题导致循环异常退出



### 位图

一、位图是什么，什么场景下可以使用位图，能达到什么效果？

- 位图不是特殊的数据格式，值为普通的字符串，也就是普通的byte数组。可以使用get/set，当然也可以使用getbit/setbit。
- 当我们需要记录用户登录、签到等true/false场景的时候，可以使用位图。
- 举例：记录一年内用户登录状态是，如果使用key/value的格式，一个用户需要365个，如果用户量有1亿，耗费空间太大。如果使用位图，一个用户使用365个位，大概46个字节即可。

二、基本命令

字符串是已ASCII码表示的，字符串a的ASCII = 01101000，高位到低位，赋值的时候，是低位到高位的标识。

```
> setbit s 1 1
(integer) 0
> setbit s 2 1
(integer) 0
> setbit s 4 1
(integer) 0
> get s
"h"

> getbit s 1
(integer) 1
> getbit s 2
(integer) 1
> getbit s 4
(integer) 1
```



三、统计 查找

```
//bitcount key [start end] ，其中start end 指的是字节。
127.0.0.1:6379> bitcount s
(integer) 3
127.0.0.1:6379> bitcount s 0 1
(integer) 3

//bitpos key bit [start end]	查找bit位为1的位置，start end 指的是bit的索引位置
127.0.0.1:6379> bitpos s 1 0 2
(integer) 1
```



四、魔术指令bitfield

指令：`bitfield key [GET type offset] [SET type offset value] [INCRBY type offset increment] [OVERFLOW WRAP|SAT|FAIL]`

```
> bitfield s get u8 0	//从第0位开始，获取无符号的8位整数。
1) (integer) 104

> bitfield i set u8 0 104	//从第0位开始，使用8bit标识无符号整数104，即"h"
1) (integer) 0
127.0.0.1:6379> get i
"h"

127.0.0.1:6379> bitfield i incrby u8 0 1	//从第0位开始，对接下来的8位无符号整数+1
1) (integer) 105
127.0.0.1:6379> get i
"i"


```



###HyperLogLog

一、什么是HyperLogLog，哪些场景可以使用，可以解决什么问题，达到什么样的效果？

- hyperloglog是redis提供的一种高级数据结构，去重保存数据，一定程度上和set相似。但是计数结果会有0.8%的误差
- 常见的使用场景是统计网站、页面的UV
- 当需要存储上千万个元素到set中时，页面有几百个，那么耗费的空间太大。hyperloglog最大的占用空间为12K。

二、基本操作 `pfadd`、`pfcount`

```
127.0.0.1:6379> pfadd v 1
(integer) 1
127.0.0.1:6379> pfadd v 2
(integer) 1
127.0.0.1:6379> pfcount v
(integer) 2
```



### 布隆过滤器

一、什么是布隆过滤器，使用场景和解决的问题，能达到什么样的效果？

- 布隆过滤器是redis4.0之后提供的一个插件，提供一个接近set去重的功能，但是稍有误差。
- 例子：给客户端用户推送消息时，用户已经看过的消息不再推送。如果使用set，那么耗费的空间过大，故可以使用布隆过滤器。节省的空间大约在90%左右。

二、基本使用

````
bf.add key1 value1
bf.exists keys value1
bf.madd key1 value2 value3
bf.exists key1 value1 value2 value3

bf.reserve key error_data initial_size
注：
	1、对于已经存在的key，使用bf.reserve 会报错
	2、error_data 错误率，值越小需要的空间越大，默认0.01
	3、initial_size 容量，初始值100，超过初始值后错误率提升
````



### 简单限流

一、什么是简单限流，使用场景和能解决的问题，能达到什么样的效果？

- 限流：a、当系统处理能力有限的时候，需要有计划的阻止外部请求。b、用户的某些行为在规定的时间内有次数限制，超过限制即视为非法请求。
- 例子：用户的发帖、点赞等。

二、简单的Redis限流策略

```
策略的要求：可以调用方指定在指定时间内，用户可以操作的次数
实现逻辑：
	1、要求中提到的指定时间，可以知道这是一个时间窗口的概念，也就是说从现在看开始，之前的一段时间。用户的行为是唯一的，所以可以使用zset数据格式。
	2、一个用户的同一种形式使用一个key来记录，score的话可以使用毫秒级的时间戳。
	3、对于冷用户，zset中的值为空，可以删除对应的key.
	4、只需要窗口时间内的数据，其余数据可删除	

代码实现：省略
```



### 漏斗过滤



### GeoHash

一、什么是GeoHash，用于什么场景，可以解决什么问题，效果如何？

- Geohash是业界计算计算两个经纬度之间距离的通用算法。原理是将二维数据点计算成一维数据格式，根据此算法，二维平面上离的近的点，计算得到后在一维上离的也近。



二、基本使用

```
geoadd key longitude latitude member [longitude latitude member ...]
//geo在存储结构上使用的是zse,删除是使用zrem即可。

//计算两个点之间的举例
geodist key member1 member2 [unit]	//单位可选有m km ml ft (米 千米 英里 尺)

//获取一个点的经纬度
geopos key member [member ...]

//获取元素的hash值
geohash key member [member ...]

//附近的人
# 范围 20 公里以内最多 3 个元素按距离正排，它不会排除自身
127.0.0.1:6379> georadiusbymember company ireader 20 km count 3 asc
1) "ireader"
2) "juejin"
3) "meituan"
# 范围 20 公里以内最多 3 个元素按距离倒排
127.0.0.1:6379> georadiusbymember company ireader 20 km count 3 desc
1) "jd"
2) "meituan"
3) "juejin"
# 三个可选参数 withcoord withdist withhash 用来携带附加参数
# withdist 很有用，它可以用来显示距离
127.0.0.1:6379> georadiusbymember company ireader 20 km withcoord withdist withhash count 3 asc
1) 1) "ireader"
   2) "0.0000"
   3) (integer) 4069886008361398
   4) 1) "116.5142020583152771"
      2) "39.90540918662494363"
2) 1) "juejin"
   2) "10.5501"
   3) (integer) 4069887154388167
   4) 1) "116.48104995489120483"
      2) "39.99679348858259686"
3) 1) "meituan"
   2) "11.5748"
   3) (integer) 4069887179083478
   4) 1) "116.48903220891952515"
      2) "40.007669977

//根据坐标值查询附近
127.0.0.1:6379> georadius company 116.514202 39.905409 20 km withdist count 3 asc
1) 1) "ireader"
   2) "0.0000"
2) 1) "juejin"
   2) "10.5501"
3) 1) "meituan"
   2) "11.5748"
```

三、注意事项

- Geo的值如果比较多的情况下，不宜使用集群部署，建议使用但实例redis部署。原因是集群中的节点可能会进行迁移，如果geo中的数据很大的情况下，在迁移的的过程中，会造成redis卡顿的现象。
- 如果geo中的数据过大，可以进行分开存储，如按照省市区县来分开存储。



### Scan

一、什么是scan，使用场景是什么，解决什么样的问题，效果如何？

- scan 是一个可以在大量keys中方便找到key的命令。
- keys 可以通过正则匹配的方法来获取到需要的keys，其中的一个缺点是结果会一次返回，如果结果有百万级别的话，这样的返回时没有意义的。数据量太大的の时候会造成卡顿。
- scan命令有如下的特点：

​	复杂和keys一样都是O(n)，但是可以通过游标来控制访问，不会阻塞线程

​	提供一个limit参数，可以控制返回量，但是此limti不是限制数量，可以理解为一个hint，限制的是卡槽。

​	同keys一样，提供模式匹配

​	服务器不需要保存游标状态，游标的唯一状态就是返给客户的整数

​	返回的结果会有重复，需要客户端去重

​	遍历的过程中如果有数据修改，能不能遍历到不确定

​	单次遍历结果为空，不能认为遍历结束，需要看游标状态是不是0



二、基本使用

```
scan cursor [MATCH pattern] [COUNT count]

scan 0 match 1*8 count 10

更多的scan命令：
hscan zscan sscan
```



三、遍历顺序

- 遍历顺序采用的是高位进位加法来遍历
- 采用此方法的原因是避免字典在扩容缩容的时候，造成遍历的重复和遗漏



## 原理

### 线程的I/O模型

一、redis 单线程为什么这么快？

redis  所有的操作都在内存中完成，故速度快。

因为redis是单线程，所以对于时间复杂度为O(n)的命令慎用，有可能会造成Redis卡顿。



二、redis的单线程如何处理多个客户端的并发连接。

在这之前需要对下面几个知识点有所了解：

- 什么是I/O
- 什么是阻塞I/O，什么是非阻塞I/O
- 什么是多路复用技术

1）什么是I/O:

主存和外部设备（磁盘驱动器、网络、终端）之间数据复制的过程

2）什么是阻塞I/O，什么是非阻塞I/O

阻塞I/O：当一个线程去读取数据的时候，如果没有读取到，那么就会等待数据可读取或者链接关闭。这对于单线程的redis来说，会造成后续命令的等待。

非阻塞I/O：当前成读取数据的时候，如果没有读到，那么会返回一个错误。

3）什么是I/O多路复用

I/O多路复用就通过一种机制，可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。



三、redis 中是怎么使用多路复用技术的

常用的多路复用API：select epoll

select 是怎么一回事，（图来自网络，不保证正确）

![img](/Users/iamcyan/Desktop/iamcyan/note/redis/redis 深度历险/IO多路复用-select.png)





### 主从同步

一、分布式系统的的基石-CAP原理

- **C** - Consistent ，一致性
- **A** - Availability ，可用性
- **P** - Partition tolerance ，分区容忍性

分布式系统的节点往往都是分布在不同的机器上进行网络隔离开的，这意味着必然会有网络断开的风险，这个网络断开的场景的专业词汇叫着「**网络分区**」。当网路分区发生的时候，一致性和可用性很难都能保证。



二、redis 的主从同步

redis 的主从同步是异步进行的，所以redis 不满足一致性的要求。当网络分区发生的的时候，主节点依然可以对外提供服务，满足可用性的要求。redis 会在网络连接正常的时候，从节点继续从主节点同步数据，以达到最终一致性。



三、Redis 同步数据 的几种方式

增量同步：

主节点将对数据有影响的指令存储在一个buffer的环形数组当中，从节点从buffer中同步数据并告知主节点目前同步位置。因为buffer是有一定长度的，如果在一定时间内从节点未能完成数据同步，那么buffer中的数据会从起始位置进行覆盖，从节点同步数据缺失。



快照同步：

主节点进行一次bgsave，将内存中的全部数据存储到磁盘中，然后将数据一次性传输给从节点，从节点接收完成后，执行一次全量加载。

隐患点：如果数据量过大或者网络延迟等问题，从节点同步完成数据的时候，buffer中的内存已经进行了覆盖更新，那么很有可能陷入死循环的状态。

增加从节点的时候，必须进行快照同步，然后再进行增量同步。

所以buffer的大小设置很关键。



无盘复制：

主节点通过socket的方式，将生成的快照文件发送给从节点，从节点将数据存储到磁盘完成之后，进行一次全量加载。



备注：

redis 如果只是用来作为缓存，那么也就无所谓主从复制，挂掉重启即可。如果使用了redis的持久化的功能，那么必须重视复制的问题，这个是系统数据安全的保证。



##其他

一、redis 持久化的两种方式、比较

RDB(快照) 

- RDB 保存某个时间点的快照数据到磁盘。
- Redis 调用fork，创建子进程，子进程将快照数据写入到临时的RDB文件，写入完成之后替换原来的RDB 文件。
- RDB 的格式很快能够问几个持久化。
- 如果redis 故障，那么从上一次RDB到故障之间的数据丢失。
- redis 生成RDB快照的两个命令：save(同步，阻塞)、bgsave(fork 子进程)



ACF(追加文件)

- ACF 的方式是服务器会记录收到的每一个写命令，服务器重启时命令会挨个执行从而重建数据，以追加文件的形式存储。
- 比RDB 的数据准确，可以指定不同的fsync策略，1s/1min 都可以，可以控制数据丢失的。
- ACF文件通常比RDB 文件大。





