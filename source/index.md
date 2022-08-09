# Redis笔记01

> 本笔记参考 [小林coding](https://github.com/xiaolincoder/CS-Base)
>
> [命令查询手册](http://doc.redisfans.com)
>
> [尚硅谷redis入门到精通](https://www.bilibili.com/video/BV1Rv41177Af?spm_id_from=333.337.search-card.all.click)
>
> Redis实战

## Redis基础

### 了解NoSql

NoSql（not only sql), 泛指非关系型数据库。

NoSql不依赖业务逻辑存储，以 key-value的形式存贮。

特点：

- 不遵循sql标准
- 不支持ACID。原子性（**A**tomicity，或称不可分割性）、一致性（**C**onsistency）、隔离性（**I**solation，又称独立性）、持久性（**D**urability）。
- 远超SQL的性能。

> 适用场景

- 对数据高并发读写
- 海量数据的读写
- 对数据高可扩展性的

> 不适用场景

- 需要事务支持
- 基于sql的结构化存储

> 常见的NOSql数据库

- Memcache
- Redis
- MongoDB

### 连接redis

shell使用 redis-cli 

> 通过docker 使用容器内redis-cli连接现有容器的redis-server

```shell
# 现有redis-server容器
docker run --name test_redis -v $PWD/develop/redis:/data:rw -p 6379:6379 \
    -d redis redis-server --save 60 1 --loglevel warning
# 查看 redis-server端的网络地址
docker network inspect bridge
# 找到具体地址
# 连接server
docker run -it  --rm redis redis-cli -h 172.17.0.3
```

### Redis 常用操作及数据类型

#### key操作

```shell
172.17.0.3:6379> keys * # 查看所有键
1) "k1"
172.17.0.3:6379> exists k1 # 是否存在
(integer) 1
172.17.0.3:6379> type k1  # 数据类型
string
172.17.0.3:6379> del k1 # 删除
(integer) 1
172.17.0.3:6379> set k2 11 # 设置k2 string
OK
172.17.0.3:6379> unlink k2 # 非阻塞删除
(integer) 1
172.17.0.3:6379> expire k2 10 # 设置 数据存活时间
(integer) 0
172.17.0.3:6379> ttl k2 # 查看存活时间
(integer) -2
172.17.0.3:6379> select 1 # 选择数据库
OK
172.17.0.3:6379[1]> dbsize  # 当前数据库数据数量
(integer) 0
172.17.0.3:6379[1]> select 0 # 选择数据库
OK

172.17.0.3:6379> flushdb # 清空当前数据库
OK
172.17.0.3:6379> flushall # 清空所有
OK


```



#### String

字符串value的最大是512M。

> 常用命令

```shell
172.17.0.3:6379> set k1 "a"  
OK
172.17.0.3:6379> set k1 "a" NX # 存在失败
(nil)
172.17.0.3:6379> set k1 "a" XX # 不存在失败
OK
172.17.0.3:6379> get k1
"a"
172.17.0.3:6379> set k1  "b" EX 20 # 20 秒超时
OK
172.17.0.3:6379> ttl k1
(integer) 15
172.17.0.3:6379> set k1  "b" PX 20 # 20毫秒超时
OK
172.17.0.3:6379> ttl k1
(integer) -2
172.17.0.3:6379> get k1
"bb"
172.17.0.3:6379> append k1 "cc" # 在字符串后追加
(integer) 4
172.17.0.3:6379> get k1
"bbcc"
172.17.0.3:6379> strlen k1  # 长度
(integer) 4
172.17.0.3:6379> setnx k1 "dd" # 同 NX 参数
(integer) 0
172.17.0.3:6379> setnx k2 "dd"
(integer) 1
172.17.0.3:6379> set k3 1 
OK
172.17.0.3:6379> incr k3 # +1
(integer) 2
172.17.0.3:6379> get k3
"2"
172.17.0.3:6379> incrby k3 20 # 按数字加
(integer) 22
172.17.0.3:6379> get k3
"22"
172.17.0.3:6379> decr k3  # -1
(integer) 21
172.17.0.3:6379> decrby k3 10 # 按数字减
(integer) 11
172.17.0.3:6379> mset k1 "a" k2 "b" k3 "c" # 批量
OK
172.17.0.3:6379> mget k1 k2 k3
1) "a"
2) "b"
3) "c"
172.17.0.3:6379> msetnx k3 "d" k4 "e" # 批量 同参数NX，一个失败都失败
(integer) 0
172.17.0.3:6379> mget k4
1) (nil)
172.17.0.3:6379> set kstr "abcd" 
OK
172.17.0.3:6379> getrange kstr 0 -1 # 按长度打印
"abcd"
172.17.0.3:6379> getrange kstr 0 -2
"abc"
172.17.0.3:6379> setrange kstr 4 "eee" # 按下标位置添加
(integer) 7
172.17.0.3:6379> get kstr
"abcdeee"
172.17.0.3:6379> setex kstr 10 k5 # 同参数EX 
OK
172.17.0.3:6379> ttl kstr
(integer) 0
172.17.0.3:6379> ttl kstr
(integer) -2
172.17.0.3:6379> getset k1 "ddd" # 返回旧值，修改新值
"a"
172.17.0.3:6379> get k1
"ddd"

```

> 底层数据结构

动态字符串

#### List

> 常用命令

```shell
172.17.0.3:6379> lpush l1 a b c d # 从左添加
(integer) 4
172.17.0.3:6379> lpop l1 # 从左移除
"d"
172.17.0.3:6379> rpush l1 d # 从右添加
(integer) 4 
172.17.0.3:6379> rpop l1 # 从右移出
"d"
172.17.0.3:6379> rpoplpush l1 l1 # 右边出，左边进
"a"
172.17.0.3:6379> lrange l1 0 -1 # 打印
1) "a"
2) "c"
3) "b"
172.17.0.3:6379> llen l1 # 长度
(integer) 3
172.17.0.3:6379> linsert l1 before c e # 一个值前、后添加
(integer) 4
172.17.0.3:6379> lrange l1 0 -1 # 
1) "a"
2) "e"
3) "c"
4) "b"
172.17.0.3:6379> lrem l1 2 c # 删除2个c
(integer) 1
172.17.0.3:6379> lrange l1 0 -1
1) "a"
2) "e"
3) "b"
172.17.0.3:6379> lset l1 1 d # 下标为1 值改成d
OK
172.17.0.3:6379> lrange l1 0 -1
1) "a"
2) "d"
3) "b"

```

> 数据结构



#### Set（集合）

> 常用命令

```shell
172.17.0.3:6379> sadd s1 1 2 3 4 5 3 4 # 添加
(integer) 5
172.17.0.3:6379> smembers s1 # 列出成员
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
172.17.0.3:6379> sismember s1 4 # 是否包含成员4
(integer) 1
172.17.0.3:6379> sismember s1 6
(integer) 0
172.17.0.3:6379> scard s1 # 长度
(integer) 5
172.17.0.3:6379> srem s1 1 2 # 移除 1 2
(integer) 2
172.17.0.3:6379> spop s1 2 # 随机移除2个
1) "4"
2) "5"
172.17.0.3:6379> srandmember s1 2 # 随机两个，但不移出
1) "3"
172.17.0.3:6379> smove s1 s2 3 # 将s1 中的3 移出至 s2
(integer) 1
172.17.0.3:6379> smembers s2 
1) "3"
172.17.0.3:6379> sinter s1 s2  # s1 和 s2 交集
(empty array)
172.17.0.3:6379> sunion s1 s2 # 并集
1) "3"
172.17.0.3:6379> smembers s1
(empty array)
172.17.0.3:6379> sadd s1 3 2 
(integer) 2
172.17.0.3:6379> sadd s2 4 5
(integer) 2
172.17.0.3:6379> sdiff s1 s2 # 差集 s1有 s2 没有
1) "2"

```

> 数据结构

哈希表

#### Hash

适合存储对象。

> 常用命令

```shell
172.17.0.3:6379> hset h1 name "ll" age 20 # 添置
(integer) 2 
172.17.0.3:6379> hget h1 name # 获取
"ll"
172.17.0.3:6379> hget h1 age
"20"
172.17.0.3:6379> hmset h2 name 'ff' age '30'  # 批量
OK
172.17.0.3:6379> hexists h1 name # 是否存在
(integer) 1
172.17.0.3:6379> hkeys h1  # 所有键
1) "name"
2) "age"
172.17.0.3:6379> hvals h1 # 所有值
1) "ll"
2) "20"
172.17.0.3:6379> hincrby h1 age 11 # 加11
(integer) 31
172.17.0.3:6379> hsetnx h1 sex 'nan' # 无此键，设置成功
(integer) 1
172.17.0.3:6379> hsetnx h1 sex 'nv' # 有此键，失败
(integer) 0
172.17.0.3:6379> hincrby h1 age -10 # 减 10
(integer) 21
172.17.0.3:6379> 

```

> 数据结构

ziplist hashtable，短的时候是ziplist

#### Zset

根据score 评分来排序的有序集合。

> 常用命令

```shell
172.17.0.3:6379> zadd z1 10 "apple" 11 "orange" 100 "banana" # 添加
(integer) 0
172.17.0.3:6379> zrangebyscore z1 10 20 # 取 分数在 10 - 20 之间的，从低到高
1) "apple"
2) "orange"

172.17.0.3:6379> zrange z1 1 2  # 按下标取
1) "orange"
2) "banana"
172.17.0.3:6379> zrevrangebyscore z1 1001 11 # 分数 11 - 1001 从高到底
1) "banana"
2) "orange"
172.17.0.3:6379> zincrby z1 11 apple # 成员分数加 11
"21"
172.17.0.3:6379> zrem z1 apple # 删除 成员
(integer) 1
172.17.0.3:6379> zcount z1 1 100 # 统计 1 -100 分数的成员数
(integer) 2
172.17.0.3:6379> zrank z1 orange  # 查看排名
(integer) 0
172.17.0.3:6379> zrank z1 banana
(integer) 1

```

> 使用场景



#### Bitmaps

位操作， 例如日活用户

> 常用命令

```shell
172.17.0.3:6379> setbit s1 1 1 
(integer) 0
172.17.0.3:6379> setbit s1 6 1 
(integer) 0
172.17.0.3:6379> setbit s1 11 1 
(integer) 0
172.17.0.3:6379> setbit s1 12 1 
(integer) 0
172.17.0.3:6379> 
172.17.0.3:6379> getbit s1 1
(integer) 1
172.17.0.3:6379> getbit s1 6
(integer) 1
172.17.0.3:6379> getbit s1 8
(integer) 0
172.17.0.3:6379> bitcount s1 0 -1 # 按范围计数
(integer) 4
172.17.0.3:6379> setbit s2 1 1 
(integer) 0
172.17.0.3:6379> setbit s2 2  1 
(integer) 0
172.17.0.3:6379> setbit s2 10  1 
(integer) 0
172.17.0.3:6379> setbit s2 12  1 
(integer) 0
172.17.0.3:6379> bitop and s1 s2 # 取交集
(integer) 2

```



#### HyperLogLog

计算基数统计（不重复元素）

```shell
172.17.0.3:6379> pfadd p1 1 2 3 4 5 2 3
(integer) 1
172.17.0.3:6379> pfcount p1
(integer) 5
172.17.0.3:6379> pfadd p2 2 7 8 
(integer) 1
172.17.0.3:6379> pfmerge p3 p1 p2 # 合并
OK
172.17.0.3:6379> pfcount p3
(integer) 7

```

#### Geospatial

地理位置信息

```shell
172.17.0.3:6379> geoadd china:city 106.50 29.53 chongqing 114.05 22.52 shenzhen 116.38 39.90 beijing 121.47 31.23 shanghai
(integer) 4
172.17.0.3:6379> geopos china:city beijing # 获取坐标
1) 1) "116.38000041246414185"
   2) "39.90000009167092543"
172.17.0.3:6379> geopos china:city shanghai 
1) 1) "121.47000163793563843"
   2) "31.22999903975783553"
172.17.0.3:6379> geodist china:city beijing shanghai # 两点距离
"1068153.5181"
172.17.0.3:6379> geodist china:city beijing shanghai km
"1068.1535"
172.17.0.3:6379> georadius china:city 100 30 1000 km # 根据经纬度 1000 km 范围内 的城市
1) "chongqing"

```



### Redis 配置文件

/etc/redis/redis.conf

- Units单位

```shell
# Note on units: when memory size is needed, it is possible to specify
# it in the usual form of 1k 5GB 4M and so forth:
#
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive so 1GB 1Gb 1gB are all the same.
```

​	只支持bytes不支持bit， 大小写不敏感

- INCLUDE包含

- 网络配置

```shell
注释掉可以远程访问
# bind 192.168.1.100 10.0.0.1
# bind 127.0.0.1 ::1
保护模式， no 可远程访问
protected-mode yes
tcp 连接的值
tcp-backlog=511
超时时间
timeout 0
检测心跳的间隔
tcp-keepalive 300
日志级别
loglevel notice
日志路径
logfile
最大终端连接数
# maxclients 10000
```

- 设置密码

  ```shell
  config get requirepass
  config set requirepass "123456"
  auth 123456
  
  ```



### 发布和订阅

订阅者：

```shell
172.17.0.3:6379> subscribe channel1
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "channel1"
3) (integer) 1
1) "message"
2) "channel1"
3) "hello"


```

发布者：

```shell
172.17.0.3:6379> publish channel1 hello
(integer) 1

```



### Redis in python

```python
import redis

```



### Redis事务和锁

#### 事务

> 事务的应用



> 事务的特性

#### 锁的使用

悲观锁、乐观锁



#### 秒杀系统

### Redis持久化

#### RDB（redis database）

#### AOF（append only file）

### 主从复制

#### 一主多从

#### 复制原理

#### 一主二仆

#### 薪火相传

#### 反客为主

#### 哨兵模式

### Redis集群

#### 集群的搭建

#### 集群操作与故障恢复

### 应用问题解决

#### 缓存穿透

命中率低

#### 缓存击穿

某个key过期，然后出现大量访问数据库的情况

#### 缓存雪崩

大量key同时过期，导致整个web应用服务器环境崩塌（数据库，应用服务器）

### Redis分布式锁

实现