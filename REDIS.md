# redis
- redis可以用来做存储，具有持久化的功能
- redis数据结构丰富，redis可以用来存储字符串，链表，哈希结构，集合，有序集合

## 安装
- wget 
- 解压源码，并进入目录
- tar zxcf xxxx
- cd 解压目录下
- 直接 make   
- make test
- yum install tcl
- make test
- make PREFIX=/usr/local/redis install
- cd /usr/local/redis
- redis-benchmark 性能测试工具
- redis-check-aof 检查aof日志的工具
- redis-check-dump 检查rbd日志的工具
- redis-cli 客户端
- redis-server 服务器
- 从源代码目录下，复制一份配置文件
- cp /usr/local/src/redis-2.6.16/redis.conf ./
- 启动
- ./bin/redis-server ./redis.conf
- ./bin/redis-cli
- 修改配置文件，可以后台运行
- vim redis.conf
- daemonize no  ->  yse 即可
- ./bin/redis-server ./redis.conf
- ps aux | grep redis

## 命令
- get
- set
- keys *  获取所有的键名
- keys s*
- keys sit[ey]  -- keys site 或者 keys sity
- keys si?e
- * 统配任意字符
- ? 统配单个字符
- [] 统配括号内的某一个字符
- randomkey  返回随机的key
- type keyName 看key是什么类型的
- exists keyName 判断某个key是否存在
- del keyName 删除
- rename keyName newName    修改key名
- renamenx keyName newName    如果新key存在，不能改，不存在可以改
- redis.conf  databases 16 默认开启了16个数据库
- 默认使用的0号数据库
- select 1 跳转到1号数据库
- move keyName  1 将keyName移动到1号数据库
- ttl keyName  查询有效期  -1 表示永久有效
- 不存在的key也返回-1 可以先用exists 是否存在
- V2.8 不存在 返回-2
- expire key 整型值  设置有效期  单位 秒
- expire key  10
- pexpire    毫秒单位的有效期
- pttl 查看有效期
- persist key 设置为永久有效

## string
- set key value ex 秒数 px 毫秒数  nx/xx
- ex px 不要同时写 两个都写，以后面的有效期为准
- pttl key 查询剩余时间
- nx 表示key不存在的时候执行操作
- xx 表示key存在时，执行操作
- flushdb 清理数据库
- mset key1 value1 key2 value2 ...
- mget key1 key2 ...
- setrange key 2 ??  hello => he??o
- 如果超过了最大的长度，中间的会用0x00 来补充
- setrange key 6 ！  hello => hello\x00!
- append 追加
- getrange  key [start stop] 获取区间 闭区间
- stop 为 负，表示从后面数 从-1开始计算
- 如果start 大于 length  返回空
- 如果stop >= length, 返回全部
- 如果start 在 stop的右边，返回空
- getset key  newvalue  返回旧值，设置新值
- incr key  增加1
- decr key  减少1
- incrby key  value 增加
- decrby key  value 减少
- incrbyfloat key value 浮点数
- setbit  key offset value
- 如果offset过大，则会在中间填充0  最大 2^32 -1 512M
- setbit key 2 1   A=>a
- setbit key 2 0   a=>A
- bitop op dest key1 key2
- op 可以是 AND OR NOT XOR

## list 链表
- lpush char a  -- left  push
- rpush char b  -- right push
- rpush char c
- lpush char 0
- lrange char 1 3
- lrange 0 -1 所有元素
- rpop key 右边弹出1个
- lpop key 左边弹出1个
- lrem key count value 删除指定的值，count 删除几个  正值count 表示从头部删  负值count表示从尾部删
- ltrim char 2 5   char链表的值就是原链表2-5范围内的元素
- lindex char index   取到索引上的值
- llen 链表的长度
- linsert num before 3  2  在3的前面插入2
- linsert num after 9  10  在10的后面插入10
- 找不到就插入失败
- 只插入一次，就算找到多个符合条件只会在第一处插入
- rpoplpush 两个链表的操作
- rpoplpush task job  原子性操作

```javascript
业务逻辑
1. rpoplpush task back
2. 接收返回值，并做业务处理
3. 如果成功，rpop back清除任务，如果不成功，下次从bak表里取任务
```
- brpop, blpop key  timeout
- 等待弹出key的尾部/头部元素
- timeout为等待时间
- 如果timeout为0，则一直等待
- 场景: 长轮休，Ajax,在线聊天时，能够用到
- brpop job  20  等待20秒

## 位图法统计活跃用户
- 1. 1亿用户，有频繁登录的，有不频繁登录的
- 2. 如何来记录用户的登录信息
- 3. 如何来查询胡活跃用户【1周登录3次的】

```javascript
Setbit 的实际应用

场景: 1亿个用户, 每个用户 登陆/做任意操作  ,记为 今天活跃,否则记为不活跃

每周评出: 有奖活跃用户: 连续7天活动
每月评,等等...

思路: 

Userid   dt  active
1        2013-07-27  1
1       2013-0726   1

如果是放在表中, 1:表急剧增大,2:要用group ,sum运算,计算较慢


用: 位图法 bit-map
Log0721:  ‘011001...............0’

......
log0726 :   ‘011001...............0’
Log0727 :  ‘0110000.............1’


1: 记录用户登陆:
每天按日期生成一个位图, 用户登陆后,把user_id位上的bit值置为1

2: 把1周的位图  and 计算, 
位上为1的,即是连续登陆的用户


redis 127.0.0.1:6379> setbit mon 100000000 0    - mon 第 100000000 设置为0 相当于 100000000全部为0
(integer) 0
redis 127.0.0.1:6379> setbit mon 3 1
(integer) 0
redis 127.0.0.1:6379> setbit mon 5 1
(integer) 0
redis 127.0.0.1:6379> setbit mon 7 1
(integer) 0
redis 127.0.0.1:6379> setbit thur 100000000 0
(integer) 0
redis 127.0.0.1:6379> setbit thur 3 1
(integer) 0
redis 127.0.0.1:6379> setbit thur 5 1
(integer) 0
redis 127.0.0.1:6379> setbit thur 8 1
(integer) 0
redis 127.0.0.1:6379> setbit wen 100000000 0
(integer) 0
redis 127.0.0.1:6379> setbit wen 3 1
(integer) 0
redis 127.0.0.1:6379> setbit wen 4 1
(integer) 0
redis 127.0.0.1:6379> setbit wen 6 1
(integer) 0
redis 127.0.0.1:6379> bitop and  res mon feb wen
(integer) 12500001


如上例,优点:
1: 节约空间, 1亿人每天的登陆情况,用1亿bit,约1200WByte,约10M 的字符就能表示
2: 计算方便

```

##  set
- 无序性
- 唯一性  不存在重复的值
- 确定行

- sadd key value1 value2
- 具有唯一性，重复的值只能增加一次
- smembers key 获取集合的所有元素
- srem key value1 value2 移除元素
- spop key 返回并删除集合中1个随机元素（抽签场景）
- srandmember 随机得到一个元素 该元素不会被删除
- sismember key value  该元素是否存在集合中  0 不存在 1 存在
- scard  key  集合中元素中的个数
- smove key1 key2 value
- 将key1集合中的元素移动到key2中，key1的值会被删除
- sinter key1 key2 key3 求交集
- sunion key1 key2 key3 求并集
- sdiff  key1 key2 差集  key1有，key2没有的元素
- sinterstore  keyResult key1 key2 存放到keyResult中

## sorted set
- zadd key score1 value1 score2 value2
- zrange key rank1 rank2  按排名查
- zrangebyscore key  score1 score2 按分数查
- zrangebyscore key score1 score2 limit 1 2
- limit 1 2 跳过1个取2个
- WITHSCORES 连分数一起取
- zrevrange
- zrank key value  返回排名
- zrevrank key value 
- zremrangebyscore key min max 删除[min,max]之间的元素
- zremrangebyrank key start end  按排名删   从0  计数
- zrem key value
- zcard key
- zcount key  score1 score2 区间有几个元素
- zinterstore key3 numberkeys key2 key1
- zinterstore key3 numberkeys key2 key1 weights weight1 weight2 aggregate sum|min|max  默认sum 
- 把key2 key1 的数据 求和 把结果放到key3中
- zunionstore key3 numberkeys key2 key1 weights weight1 weight2 aggregate sum|min|max

## hash
- hset key filed value
- hgetall key 
- hmset key field1 value field vaule2 ...
- hmget key field1 field2
- hdel key field
- hlen key 查询长度
- hexists key field
- hincby key field value
- hincbyfloat key field value
- hkeys key
- hvals key

## 事务
- multi
- decrby key1 100
- incrby key2 100
- exec
- multi 之后并不会执行
- 而是放在了队列里
- exec 集中执行
- 执行的时候，会执行正确的语句，并跳过不正确的语句 
- discard  实际就是清理队列 
- 即使是exec中出错了，exec中正确执行的语句也不会被取消


- 乐观锁
- watch 监控，如果watch以后，在执行事务的时候出现了变化，事务会执行失败 exec 返回nil
- set ticket 1
- set lisi 300
- set  wang 300
- watch ticket
- multi
- decr ticket 
- decrby lisii 100
- exec  
- watch 可以监视多个key，任意一个key 有变，事务取消
- unwatch  
- 取消监控

## pub/sub
- publish news 'today is sunshine'
- subscribe news 
- psubscribe new*
- pubsub channels 列出当前活动的收听频道 如果没有指定，会列出所有的
- 群聊，系统广告

## rdb快照持久化
- 把数据存储于断电后不会丢失的设备中，通常是硬盘
- 通过从服务器保存和持久化，如mongodb的replication sets 配置
- 操作生成相关日志，并通过日志来恢复数据
- rdb的工作原理
- 每隔N分钟或者N次操作以后，从内存中dump数据，形成rdb文件
- 压缩
- 放在备份目录
- redis.conf
- save 900 1
- save 300 10
- save 60 10000
- 上面的3个配置都注释掉，则不再导出rdb文件
- 60秒内有10000次操作或者变化进行保存
- 300秒内超过10次变化，进行保存
- 900秒内有超过1次变化，进行保存
- stop-writes-on-bgsave-error yes
- 当后台导出rdb出错的时候，停止redis写入
- rdbcompression yes
- 是否压缩
- rdbchecksum yes
- 重启redis服务器从rdb导入数据到内存，要检测数据是否完整
- dbfilename dump.rdb
- dir ./
- redis-benchmark -n 10000  性能测试
- 两个保存点之间断电，将会丢失1-N分钟的数据
- 出于对持久化更精细的要求，redis增加了atof方式 append only file
- redis-server -> if(触发条件) {redis-dump进程}  
- 因为导出的是整块的内存，恢复速度是比较快的

## aof日志持久化
- appendonly no  开关
- appendfsync always 每一个命令都立即同步到aof，安全，速度慢
- appendfsync everysec 每秒1次
- appendfsync no 写入工作交给操作系统，由操作系统判断缓冲区的大小 统一写到aof，同步频率低，速度快
- no-appendfsync-on-rewrite yes 正在导出rdb快照的过程中，要不要停止同步aof
- 在dump rdb 的过程中，aof如果停止爱同步，会不会丢失？
- 不会，所有的操作都会缓存在内部的队列里，dump完成后，统一操作
- appendfilename appendonly.aof

- auto-aof-rewrite-precentage 100  
- aof文件大小比起上次重写时的大小，增长率100% 重写
- auto-aof-rewrite-size 64m 文件至少64M才重写
- bgrewriteaof 重写aof文件

- aof重写指什么？
- aof重写指把内存中的数据，逆化成命令，写到aof中，以解决aof过大的问题
- 优先使用aof恢复数据，恢复的时候，可能仅用了aof，而没有用rdb
- 两种可以同时使用，推荐这么做
- rdb恢复数据更快，rdb是数据直接载入到内存， 二进制数据，aof需要逐条执行命令

## redis服务器集群
- 主从备份，防止主机宕机
- 读写分离，分担master的任务
- 任务分离，如从服分别分担备份工作与计算工作
- 星型集群 直线型集群

### 主从同步过程
- slave -> aync -> master
- master ->dump -> rdb -> slave
- master -缓冲aof -> salve
- master -> replicationFeedSlaves -> salve 

### 部署
- cp redis.conf redis6380.conf
- cp redis.conf redis6381.conf
- 6380配置
- pidfile /var/run/redis6380.pid
- port 6380
- dbfilename  dump6380.rdb
- salveof localhost 6379  6380是6379的salve
- slave-read-only yes  只读选项

- 6381配置
- pidfile /var/run/redis6381.pid
- port 6381
- dbfilename  dump6380.rdb  这里rdb就不再产生了，aof也不产生了
- salveof localhost 6379  6381是6379的salve
- slave-read-only yes  只读选项

- 6379 配置
- 禁止rdb即可
- aof可要可不要（主服务器打开比较好，信息比较全）


- 启动
- redis-server redis.conf
- redis-server redis6380.conf
- redis-server redis6381.conf

- 一个salve 做rdb
- master  做 aof
- 内网就不要用密码了

- 密码
- 主服务器 requirepass 123456
- auth 密码
- 配了密码以后，从服务器也连不上了
- 从配置密码
- masterauth 123456

- 主从复制缺陷
- 每次salve断开后，无论是主动断开，还是网络故障
- 再连接master
- 都master全部dump出来rdb，再aof
- 即同步的过程都要重新执行一遍
- 多台salve不要一下都启动起来，否则master可能io飙升








    



   





























 



















