# 1 Redis 持久化
redis持久化，RDB快照和AOF日志，前者一段时间存一下，后者是写之前存；前者持久的数据少、但是恢复快；后者持久的数据全、但是恢复的慢；生产环境二者结合，出了问题先用RDB、再用AOF；
## RDB（Redis DataBase）（snapshotting 快照）
- RDB优点：使用单独子进程来进行持久化，主进程不会进行任何IO操作，保证了redis的高性能
- RDB缺点：间隔一段时间进行持久化，如果持久化之间redis发生故障，会发生数据丢失，更适合数据要求不严谨的时候
- 可以通过配置来自己确RDB，配置redis在n秒内如果超过m个key被修改这执行一次RDB操作（通常叫做snapshots）
- snapshot触发的时机，是有“间隔时间”和“变更次数”共同决定，二者同时符合即触发；否则变更次数累加到下一个变更时间
- snapshot过程中并不阻塞客户端请求；snapshot首先将数据写入临时文件，当成功结束后，将临时文件重名为dump.rdb
- RDB文件的载入工作是在服务器启动时自动执行的，并没有专门的命令；但是由于AOF的优先级更高，因此当AOF开启时，
- Redis会优先载入AOF文件来恢复数据；只有当AOF关闭时，才会在Redis服务器启动时检测RDB文件，并自动载入
- 服务器载入RDB文件期间处于阻塞状态，直到载入完成为止
- Redis载入RDB文件时，会对RDB文件进行校验，如果文件损坏，则日志中会打印错误，Redis启动失败
- 可以配置 redis 在 n 秒内如果超过 m 个 key 被修改就自动做快照
- 1 redis fork一个子进程出来，并将此刻数据库内容做快照
- 2 父进程继续处理client请求，子进程负责将内存内容写入临时文件
- 3 子进程将快照写入临时文件完毕后，替换原来的快照文件，然后子进程退出
- （client也可以通过save或者bgsave命令通知redis做一次快照，前者会阻塞client请求，后者是非阻塞的）
- （快照都是将内存数据完整写入到磁盘一次，并不是增量的只同步变更数据，如果数据量大的话，而且写操作比较多， 必然会引起大量的磁盘 io 操作，可能会严重影响性能）
``` 
dbfilename dump.rdb              # 持久化数据存储在本地的文件
dir ./                           # 默认是redis-cli的src目录下
save 300 10                      # snapshot触发的时机，save <seconds> <changes> 更改了10个key300秒存储
stop-writes-on-bgsave-error yes  # 当snapshot时，因为磁盘已满、磁盘故障、OS异常，是否阻塞客服端变更操作
rdbcompression yes               # 是否压缩，压缩意味着额外的CPU消耗，但也是更少的网络传输
```
### redis rdb 手动持久化
```
./redis-cli bgsave               # client和server在同一个机器，可以使用简化的命令
./redis-cli -h ip -p port save   # 前台存储，阻塞主进程，客户端无法连接redis，等SAVE完成后，主进程才开始工作，客户端可以连接
./redis-cli -h ip -p port bgsave # 后台存储，是fork一个save的子进程，在执行save过程中，不影响主进程，客户端可以正常链接redis，等子进程fork执行save完成后，通知主进程，子进程关闭
补充：在主从复制场景下，如果从节点执行全量复制操作，则主节点会执行bgsave命令，并将rdb文件发送给从节点，执行shutdown命令时，自动执行rdb持久化
```
## AOF（Append-only file）
- 将“操作 + 数据”以格式化指令的方式追加到操作日志文件的尾部，恢复的时候replay此日志文件
- 在append操作返回后(已经写入到文件或者即将写入)，才进行实际的数据变更
- 优点：可以保持更高的数据完整性；且如果日志写入不完整支持redis-check-aof来进行日志修复；
- 优点：AOF文件没被rewrite之前（文件过大时会对命令进行合并重写），可以删除其中的某些命令（比如误操作的flushall）
- 缺点：AOF文件比RDB文件大，且恢复速度慢
- 一条数据经过多次变更，将会产生多条AOF记录，持久化模式还伴生了“AOF rewrite”
- 入时突然server失效，有可能导致文件的最后一次记录是不完整，你可以通过手工或者程序的方式去检测并修正不完整的记录
- 如果设置的redis持久化手段中有aof，那么在server故障失效后再次启动前，需要检测aof文件的完整性
- 如果操作一条命令就写一次IO持久化，磁盘IO负担太重，就有了三种策略：always、everysec、no,默认为everysec 
- linux对文件操作采取延迟写入，先写临时文件，到达buffer值触发写入，这是linux对文件系统的优化
- AOF rewrite操作就是“压缩”AOF文件的过程（因为文件越来越大，需要进行重写，开辟新内存，可能导致内存暴涨）
- rewrite会fork出一个子进程，创建一个临时文件，遍历数据库，将每个key、value对输出到临时文件
- 输出格式就是Redis的命令，但是为了减小文件大小，会将多个key、value对集合起来用一条命令表达
- 在rewrite期间的写操作保存在内存的rewrite buffer中，rewrite成功后这些操作会复制到临时文件中，再代替AOF文件
- rewrite过程中，出现故障，将不会影响原AOF文件的正常工作，只有当rewrite完成之后才会切换文件
- 以上在AOF打开的情况下，如果AOF是关闭的，那么rewrite操作可以通过bgrewriteaof命令来进行
```
appendonly yes       # 默认是关闭的，需要手动打开 
appendfilename appendonly.aof 
appendfsync everysec # 同步策略有三种，always everysec no,默认为everysec；always立即同步；如果是no，操作系统可以根据buffer填充情况/通道空闲时间等择机触发同步 
no-appendfsync-on-rewrite no   # 在aof-rewrite期间，appendfsync是否暂缓文件同步，默认是no
auto-aof-rewrite-min-size 64mb # aof文件rewrite触发的最小文件尺寸(mb,gb),默认“64mb”，建议“512mb”  
···
##相对于“上一次”rewrite，本次rewrite触发时aof文件应该增长的百分比。  
##每一次rewrite之后，redis都会记录下此时“新aof”文件的大小(例如A)，那么当aof文件增长到A*(1 + p)之后  
##触发下一次rewrite，每一次aof记录的添加，都会检测当前aof文件的尺寸。  
auto-aof-rewrite-percentage 100 
```
### redis aof 手动持久化
```
redis-cli -h ip -p port bgrewriteaof # 通过fork一个子进程，并不会阻塞client操作
redis-cli-> config get appendonly    # 查看是否开启
redis-cli-> config set appendonly no # 手动关闭aof（OOM，全称“Out Of Memory”）
```
### 缓存会碰到的问题
- 缓存穿透：缓存和数据库中都没有，而用户不断发起请求，导致数据库压力过大； 1：接口层增加校验 2：将key-value对写为key-null，并设置过期时间
- 缓存击穿：(非常热点的数据)缓存中没有但数据库中有，引起数据库压力瞬间增大； 1：优化热点数据的过期时间 2：加互斥锁，加锁去数据库取数据，没有拿到锁的请求可以sleep一下再尝试
- 缓存雪崩：缓存中数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机； 1：设置随机过期时间 2：热点数据分布式部署 3：热点数据永不过期
### redis分布式锁
先拿setnx来争抢锁，抢到之后，再用expire给锁加一个过期时间防止锁忘记了释放；在setnx之后执行expire之前进程意外crash，可以把setnx和expire合成一条指
- 使用keys指令可以扫出指定模式的key列表，但由于redis是单线程的，会导致阻塞
- scan指令可以无阻塞的提取出指定模式的key列表，但是花费时间长，会有一定的重复概率，在客户端做一次去重就可以了（增量迭代过程中，键可能被修改）
### redis异步队列
- 私用list结构作为队列，rpush生产消息，lpop消费消息。当lpop没有消息的时候，要适当sleep一会再重试
- 不使用sleep，list还有个指令叫blpop，在没有消息的时候，它会阻塞住直到消息到来
- 生产一次消费多次：使用pub/sub主题订阅者模式，可以实现 1:N 的消息队列（可能会出现消息丢失，使用更专业的RocketMQ等）
### redis做延时队列
- 使用sortedset，拿时间戳作为score，消息内容作为key调用zadd来生产消息，消费者用zrangebyscore指令获取N秒之前的数据轮询进行处理
### redis之pipeline（管道）
- Redis其实是一个基于TCP协议的CS架构的内存数据库。所有的操作都是一个request一个response的同步操作。
- redis每接收到一个命令就会处理一个命令，并同步返回结果。问题是，一个命令就会产生一次RTT（Round Time Trip），这样的话必然会消耗大量的网络IO。
- 通过pipeline提高redis的读写能力，对于多个命令执行，不再同步等待每个命令的返回结果，在统一时间点来获取response；解决多RTT的问题
- 将多次IO往返的时间缩减为一次，前提是pipeline执行的指令之间没有因果相关性；
### Redis的同步机制
- 主从同步。第一次同步时，主节点做一次bgsave，并同时将后续修改操作记录到内存buffer，待完成后将RDB文件全量同步到复制节点
- 复制节点接受完成后将RDB镜像加载到内存。加载完成后，再通知主节点将期间修改的操作记录同步到复制节点进行重放就完成了同步过程
- 后续的增量数据通过AOF日志同步即可，有点类似数据库的binlog。
### 与Memcache比较
- mc使用多线程异步IO方式，可以合理利用多核优势
- key不能超过 250 个字节、value 不能超过 1M 字节、key 的最大失效时间是 30 天、只支持 K-V 结构，不提供持久化和主从同步功能
### redis 事务
- Redis 提供的不是严格的事务，Redis 只保证串行执行命令，并且能保证全部执行，但是执行命令失败时并不会回滚，而是会继续执行下去
### 基础类型、高级类型
- list/set/hash/set/sortedset
- Bitmap/HyperLogLog/Geospatial/pub sub/Pipeline/Lua(写脚本判断秒杀库存，原子性)
### 最经典的缓存+数据库读写的模式，就是 Cache Aside Pattern
- 读的时候，先读缓存，缓存没有的话，就读数据库，然后取出数据后放入缓存，同时返回响应
- 更新的时候，先更新数据库，然后再删除缓存
- 为什么是删除缓存，而不是更新缓存: 在复杂点的缓存场景，缓存值可能是多表多字段计算的值
### URL动态化
- 就连写代码的人都不知道，通过MD5之类的加密算法加密随机的字符串去做url，然后通过前端代码获取url后台校验才能通过
### strings 类型及操作
string类型是二进制安全的，可包含任何数据，如二进制图片或序列化的对象；内部是一个byte数组，上限1G；
```
set name qiyuan 设置 key 对应的值为 string 类型的 value。
setnx key value 如果 key 已存在，返回 0，nx 是 not exist
setex key seconds value 指定过期时间
setrange key offset value 指定替换，从下标offset开始并包含
mset key value [key value ...] 一次设置多个 key 的值
msetnx key value [key value ...] 失败返回0，都不执行（回滚）
···
get key  获取 key 对应的 string 值,如果 key 不存在返回 nil
getset key value 返回旧值；没有旧值返回nil，新值照样设定
getrange key start end 获取子串（切片），-1 从后开始，-7 -1
mget key [key ...] 一次获取多个 key 的值,没有的安装nil返回
···
incr key 对 key 的值做加加操作,并返回新的值;不是int会返回错误；key不存在则设置为1
incrby key increment 加指定数值;不存在则先设置在增加，默认为0
decr key 一个不存在 key，则设置 key 为-1
decrby key decrement 与incrby一个负值效果一样
append key value 字符串值追加 value,返回新字符串值的长度
strlen key 取指定 key 的 value 值的长度
```
### hashes 类型及操作
Redishash是一个string类型的field和value的映射表，添加、删除都是O（1）（平均），
特别适合用于存储对象，内部使用zipmap（又称small hash）来存储，节省了hash需要的元数据开销，但是如果field或value超过一定大小，会将zipmap替换为正常的hash实现，可以通过配置文件制定；
```
hash-max-zipmap-entries 64 #配置字段最多 64 个 
hash-max-zipmap-value 512  #配置 value 最大为 512 字节
···
hset key field value 设置 field 为指定值，如果 key 不存在，则先创建
hsetnx key field value 存在返回0；不存在先创建，返回1
hmset key field value [field value ...]
hget key field 没有返回nil
hmget key field [field ...]
hincrby key field increment  指定的 hash filed 加减上给定值
hexists key field 存在返回1，不存在返回0
hlen key 返回field个数
hdel key field [field ...]
hkeys key 返回所有field
hvals key
hgetall key 获取全部的filed和value
```
### lists 类型及操作
list 是一个链表结构, key 为链表的名字，每个元素都是string，最大长度2的32次方；也可以作为栈、队列使用；
[lr]pop 为非阻塞，如果 list 是空，或者不存在，会立即返回 nil
b[lr]pop 为**阻塞操作**，可以加超时时间，超时后也会返回 nil；如果用list实现工作队列，阻塞避免了轮询检查是否有有任务存在；
```
lpush key value [value ...]
lrange key start stop
rpush key value [value ...]
linsert key BEFORE|AFTER pivot value  在 key 对应 list 的特定位置之前或之后添加字符串元素
lset key index value 设置 list 中指定下标的元素值(下标从 0 开始)
lrem key count value 从 key 对应 list 中删除 count 个和 value 相同的元素,count>0 时，按从头到尾的顺序删除,< 0 从尾开始
ltrim key start stop 保留指定 key 的值范围内的数据；第一个是0，最后一个是-1
rpop key 从 list 的尾部删除元素，并返回删除元素
rpoplpush source destination 从第一个 list 的尾部移除元素并添加到第二个 list 的头部,最后返回被移除的元素值，整个操 作是原子的.如果第一个 list 是空或者不存在返回 nil
lindex key index  返回名称为 key 的 list 中 index 位置的元素
llen key 
```
### sets 类型及操作
set 是集合,可以添加删除元素，可以求交并差等操作，key是集合的名字
set 是 string 类型的无序集合，最多可以包含2的32次方个元素
set 的是通过 hash table 实现的，所以添加、删除和查找的复杂度都是 O(1)。hash table 会随 着添加或者删除自动的调整大小
调整 hash table 大小时候需要同步(获取写 锁)会阻塞其他读写操作，可能不久后就会改用跳表(skip list)来实现，跳表已经在 sorted set 中使用了
```
sadd key member [member ...] 重复元素不在添加
srem key member [member ...] 成功返回1，不存在失败返回0
spop key [count] 随机返回并删除名称为 key 的 set 中元素
smembers key  查看集合内元素
sdiff key [key ...] 返回所有给定 key 与第一个 key 的差集
sdiffstore destination key [key ...] 返回所有给定 key 与第一个 key 的差集，并将结果存为另一个 key
sinter key [key ...] 返回所有给定 key 的交集
sinterstore destination key [key ...]
sunion key [key ...]  返回所有给定 key 的并集
sunionstore destination key [key ...]
smove source destination member 从第一个 key 对应的 set 中移除 member 并添加到第二个对应 set 中
scard key 返回名称为 key 的 set 的元素个数 
sismember key member 测试 member 是否为key 的元素,是返回1，不是返回0
srandmember key [count] 随机返回元素，但是不删除元素
```
### sorted sets 类型及操作
zset也是string类型元素的集合，每个元素都关联一个double类型的score
在 set 的基础上增加了一个顺序属性,修改元素后zset会自动重新按新的值调整顺序
可以理解为两列mysql表，一列存value，一列存顺序
sorted set 的实现是 skip list 和 hash table 的混合体
zset常用作索引来使用，要排序字段做score，对象id做元素存储
```
zadd key [NX|XX] [CH] [INCR] score member [score member ...] 向名称为 key 的 zset 中添加元素 member，score 用于排序;如果key存在则按照score更新元素的顺序
zrange key start stop [WITHSCORES]                  # zrange jrtt_child_agents_v2_T100117 0 -1 
zrem key member [member ...] 删除
zincrby key increment member 元素的 score 增加 increment,否则新增
zrank key member 返回名称为 key 的 zset 中 member 元素的下标
zrevrank key member
zrevrange key start stop [WITHSCORES]
zrangebyscore key min max [WITHSCORES] [LIMIT offset count] 返回集合中 score 在给定区间的元素
zcount key min max 返回集合中 score 在给定区间的数量       # zcount jrtt_child_agents_v2_T100117 0 inf
zcard key 返回集合中元素个数                              # zcard jrtt_child_agents_v2_T100117
zscore key member 返回给定元素对应的 score
zremrangebyrank key start stop 删除集合中排名在给定区间的元素
zremrangebyscore key min max
```
### 键值相关命令
```
keys pattern  # keys * 返回满足给定 pattern 的所有 key;用表达式*，代表取出所有的 key
exists key [key ...]
del key [key ...]
expire key seconds
ttl key  # 获取过期时间,-1 说明此值已过期
select index # 选则数据库 
move key db # 将当前数据库中key移动到其他数据库
persist key  移除给定 key 的过期时间
randomkey # 随机返回 一个 key
rename key newkey
type key  # 返回值的类型
select index # Redis 数据库编号从 0~15,可以选择任意一个数据库来进行数据的存取
dbsize 返回当前数据库中 key 的数目
info 获取服务器的信息和统计
monitor 实时转储收到的请求
config get [dir] 获取服务器配置信息
flushdb 删除当前选择数据库中的所有 key
flushall 删除所有数据库中的所有 key
requirepass foobared  requirepass qiyuan 设置连接口令,auth qiyuan 或者 redis-cli -a qiyuan 
multi ... ... exec  简单事务控制，discard 命令清空事务的命令队列并退出事务上下文，也就是事务回滚
```
### 发布及订阅消息 (pub/sub)
pub/sub 不仅仅解决发布者和订阅者直接 代码级别耦合也解决两者在物理部署上的耦合
redis 作为一个 pub/sub 的 server，在订阅者 和发布者之间起到了消息路由的功能
- 消息类型称为通道(channel)
- 发布者通过 publish 命令向 redis server 发送特定类型的消息
- 订阅者可以通过 subscribe 和 psubscribe 命令向 redis server 订阅自己感兴趣的消息类型(psubscribe tv* 批量订阅)
- publish channel message
- subscribe channel [channel ...]
# 26 再次理解生产者和消费者
```
import random
import time
from Queue import Queue
from threading import Thread

queue = Queue(10)

class Producer(Thread):
    def run(self):
        while True:
            elem = random.randrange(9)
            queue.put(elem)
            print "厨师 {} 做了 {} 饭 --- 还剩 {} 饭没卖完".format(self.name, elem, queue.qsize())
            time.sleep(random.random())

class Consumer(Thread):
    def run(self):
        while True:
            elem = queue.get()
            print "吃货{} 吃了 {} 饭 --- 还有 {} 饭可以吃".format(self.name, elem, queue.qsize())
            time.sleep(random.random())

def main():
    for i in range(3):
        p = Producer()
        p.start()
    for i in range(2):
        c = Consumer()
        c.start()

if __name__ == '__main__':
    main()

```
Redis 消费者模式: blpop获取队列数据，如果队列没有数据则阻塞等待，也就是监听
```
import redis

class Task(object):
    def __init__(self):
        self.rcon = redis.StrictRedis(host='localhost', db=5)
        self.queue = 'task:prodcons:queue'

    def listen_task(self):
        while True:
            task = self.rcon.blpop(self.queue, 0)[1]
            print "Task get", task

if __name__ == '__main__':
    print 'listen task queue'
    Task().listen_task()
```
Redis 发布订阅模式: ubsub功能，订阅者订阅频道，发布者发布消息到频道了，频道就是一个消息队列
```
mport redis


class Task(object):

    def __init__(self):
        self.rcon = redis.StrictRedis(host='localhost', db=5)
        self.ps = self.rcon.pubsub()
        self.ps.subscribe('task:pubsub:channel')

    def listen_task(self):
        for i in self.ps.listen():
            if i['type'] == 'message':
                print "Task get", i['data']

if __name__ == '__main__':
    print 'listen task channel'
    Task().listen_task()
```
# redis db0-db15
- 默认生成16个db，并连接db0；可以通过配置文件redis.conf修改 databases 16，重启后生效
- 获取当前的db数：> CONFIG GET databases，通过select 切换
- 查看数据库db相关的统计信息：> INFO KEYSPACE
- brew search redis 查看redis服务
- brew services list 查看正在运行的服务列表
- brew services start redis 启动

# redis stream
xadd memberMessage * user wei msg yes
XREAD streams memberMessage 0

# redis-cli  info 查看版本等信息

# 找出拖慢 Redis 的罪魁祸首
由于 Redis 没有非常详细的日志，要想知道在 Redis 实例内部都做了些什么是非常困难的。幸运的是 Redis 提供了一个下面这样的命令统计工具：
127.0.0.1:6379> INFO commandstats# Commandstatscmdstat_get:calls=78,usec=608,usec_per_call=7.79 cmdstat_setex:calls=5,usec=71,usec_per_call=14.20 cmdstat_keys:calls=2,usec=42,usec_per_call=21.00 cmdstat_info:calls=10,usec=1931,usec_per_call=193.10
通过这个工具可以查看所有命令统计的快照，比如命令执行了多少次，执行命令所耗费的毫秒数(每个命令的总时间和平均时间)
只需要简单地执行 CONFIG RESETSTAT 命令就可以重置，这样你就可以得到一个全新的统计结果。

# MySQL自增列ID--AUTO_INCREMENT
在一张表中，里面有ID自增主键，当insert了17条记录之后，删除了第15,16,17条记录，再把MySQL重启，再Insert一条记录，
这条记录的ID是18还是15？
在MySQL 8.0之前，对于MyISAM和InnoDB的表，因为存储引擎对于自增列的实现机制不同，ID值也会有所不同，
对于InnoDB存储引擎的表，ID是按照max(id)+1的算法来计算的。在MySQL 8.0之后，InnoDB的自增列信息写入了共享表空间中，
所以服务重启之后，还是可以继续追溯这个自增列的ID变化情况的。
所以，对于MyISAM表来说，若数据库重启后，则ID值为18；
对于InnoDB表来说，若数据库重启后，则对于MySQL 8.0来说，ID值为18，
对于MySQL 8.0之前的数据库来说，则数据库重启后，ID值为15。
MySQL [(none)]> use lhrdb;
Database changed
MySQL [lhrdb]> select @@version;
```
在MySQL 8.0之前：

    1）如果是MyISAM表，则数据库重启后，ID值为18

    2）如果是InnoDB表，则数据库重启后，ID值为15

在MySQL 8.0开始，

    1）如果是MyISAM表，则数据库重启后，ID值为18

    2）如果是InnoDB表，则数据库重启后，ID值为18
```