# Redis命令速查

## KEY操作

```shell
keys pattern # 根据给定模式查找键
scan cursor [MATCH pattern] [COUNT count] # 一个基于游标的迭代器
randomkey # 从数据库中随机返回一个键
sort key [BY pattern] [LIMIT start count] [GET pattern] [ASC|DESC] [ALPHA] [STORE dstkey] # 排序

dbsize # 当前数据库内键的数量
exists key [key ...] # 一个或者多个键是否存在
type key # 返回键的类型

rename key newkey # 键改名
renamenx key newkey # 键改名，新名字没有占用才成功
move key db # 当前数据库键移动到指定数据库
del key [key ...] # 删除一个或多个键
unlink key [key ...] # 非阻塞删除键
flushdb # 清空当前数据库
flushall # 清空所有数据库

expire key seconds [NX|XX|GT|LT] # 设置键过期时间 秒
pexpire key milliseconds [NX|XX|GT|LT] # 毫秒

ttl key # 还有多少秒过期
pttl key # 毫秒
persist key # 删除键的过期时间设置
```



## 字符串（string）

```shell
set key value [NX|XX] [GET] [EX seconds|PX milliseconds|EXAT uni # 设置键和过期时间
get key # 获取键的值
getset key value # 设置键的值，并返回旧值
setnx key value # 设置键的值，仅有没有此键时成功
setex key seconds value # 设置键的值，并设置过期时间（秒）
psetex key milliseconds value # 设置键的值，并设置过期时间（毫秒）

mset key value [key value ...] # 一次设置多个key value
mget key [key ...] # 一次获取多个key的值
msetnx key value [key value ...] # 批量设置多个key value，仅没有此key时成功

strlen key # 字符串长度
setrange key offset value # 从字符串指定索引开始修改
getrange key start end # 从start-end区间的字符串
append key value # 指定key后拼接value

incr key # 整数+1
decr key # 整数-1
incrby key increment # 加一个数
decrby key decrement # 减一个数
incrbyfloat key increment # 加浮点型

```



## 列表(list)

```shell
lpush key element [element ...] # 一个或者多个元素添加至列表，左进
lpushx key element [element ...] # 一个或者多个元素添加至列表,左进，仅列表存在成功
rpush key element [element ...] # 一个或者多个元素添加至列表，右进
rpushx key element [element ...] # 一个或者多个元素添加至列表,右进，仅列表存在成功

lpop list # 移除并返回左一
rpop list # 移除并返回右一
blpop key [key ...] timeout # 移除并返回左一,指定时间内
brpop key [key ...] timeout # 移除并返回右一,指定时间内

rpoplpush source destination # 从 source 右边移出一个向destination左边添加
brpoplpush source destination # 从 source 右边移出一个向destination左边添加,在指定时间内

lindex list index # 获取指定下标的元素
llen list # 长度
lrange list start end # start-end 获取
linsert key BEFORE|AFTER pivot element # 在 pivot 之前/之后插入元素
lrem list count element # 移除指定元素
lset key index element # 将下标的值改成目标值
ltrim list start end # 根据start end 裁剪list
```



## 字典(hash)

```shell
hset key field value [field value ...] # 设置一个后者多个值
hsetnx key field value # 设置键值对，存在设置失败
hget key field # 根据字段获取值
hmset key field value [field value ...] # 批量设置
hmget key field [field ...] # 批量获取

hincrby key field increment # 键的某字段增加一个数
hincrbyfloat key field increment # 键的某字段增加一个浮点数

hexists key field # 是否存在
hlen key # 键值对数量
hdel key field [field ...] # 删除一个或多个字段

hkeys key # 返回所有键
hvals key # 返回所有值
hgetall key # 返回所有键和值
hscan key cursor [MATCH pattern] [COUNT count] # 渐进式遍历 
```



## 集合(set)

```shell
sadd key member [member ...] # 添加一个或多个值
spop key [count] # 随机返回N个
smove source destination member # 移动成员从source到destination集合
srem key member [member ...] # 移除一个或者多个成员

scard key # 返回成员数量
sismember key member # 是否包含此成员
srandmember key [count] # 随机返回N个，不删除
smembers key # 返回所有成员
sscan key cursor [MATCH pattern] [COUNT count] # 渐进式遍历

sdiff key [key ...] # 集合差集,前者中有，后者中没有的元素
sdiffstore destination key [key ...] # 集合差集，存储至destination
sunion key [key ...] # 集合并集，多个集合中所有的元素
sunionstore destination key [key ...] # 集合并集，存储至destination
```



## 有序集合(sorted set)

```shell
zadd key [NX|XX] [GT|LT] [CH] [INCR] score member [score member ...] # 添加一个值多个值到集合
zincrby key increment member # 增加一个值
zscore key member # 返回指定成员的分数
zcard key # 成员数量
zrank key member # 返回指定成员按分数从小到大的排名
zrevrank key member # 返回指定成员按分数从大到小的排名 

zcount key min max # 分值在 [min, max]之间的成员
zrange key start stop [BYSCORE|BYLEX] [REV] [LIMIT offset count] [WITHSCORES] # 指定start-end，按分值从小到大返回成员
zrangebyscore key min max [WITHSCORES] [LIMIT offset count] # 分数在[min,max]之间，返回成员和分数
zscan key cursor [MATCH pattern] [COUNT count] # 以渐进的方式返回
zrem key member [member ...] # 删除成员
zremrangebyrank key start stop # 删除排名在 [start, stop] 的成员，按分数从小到大排名
zremrangebyscore key min max # 删除分数在[min, max]的成员

zinterstore destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX] # 将集合做交集，结果存储在destination 集合
zunionstore destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX] # 将集合做并集，结果存储在destination 集合

zlexcount key min max # 指定大小范围内的成员数量
zrangebylex key min max [LIMIT offset count] # 按从小打到的顺序返回成员
zremrangebylex key min max # 从集合里移除指定大小范围的成员
```



## 位图（bitmap）

```shell
setbit key offset value # 位图索引上设置0或者1
getbit key offset # 获取索引上的二进制值

bitcount key [start end [BYTE|BIT]] # 统计start-end 置1的数量 bit按位 byte按字节
bitop operation destkey key [key ...] # 对多个位图进行逻辑运算（and、or、xor、not），并将结果存放至 destkey
```



## HyperLogLog(基数)

```shell
pfadd key [element [element ...]] # 一个或者多个添加至hyperloglog里
pfcount key [key ...] # 一个或者多个hyperloglog里基数的个数
pfmerge destkey sourcekey [sourcekey ...] # 合并操作，将一个或者多个sourcekey合并至destkey 
```



## 地理位置（Geospatial）

```shell
geoadd key [NX|XX] [CH] longitude latitude member [longitude lat # 添加一个或者多个地理位置
geopos key member [member ...] # 获取其经纬度
geodist key member1 member2 [M|KM|FT|MI] # 两者之间的距离
georadiusbymember key member radius M|KM|FT|MI [WITHCOORD] [WITHDIST] [WITHHASH] [ASCIDESC] [COUNT count] # 返回该位置方圆多少距离的其他位置
geohash key member [member ...] # 计算 geo hash值
```



## 事务（transaction）

```shell
multi # 开始事务
exec # 执行事务
discard # 取消事务

watch key [key ...] # 监视一个或者多个键，（上乐观锁）
unwatch # 取消所有键的监视
```



## 脚本（script）

```shell
eval script numkeys [key [key ...]] [arg [arg ...]] # 执行lua脚本
evalsha sha1 numkeys [key [key ...]] [arg [arg ...]] # 执行校验并载入相应的脚本

script load script # 载入给定的lua脚本
script exists sha1 [sha1 ...] # 判断lua脚本是否载入
script kill  # 杀死正在执行的lua脚本
script flush # 移除所有lua脚本
```



## 发布和订阅(publish/subscribe)

```shell
publish channel message # 给某频道发布消息
subscribe channel [channel ...] # 订阅
psubscribe pattern [pattern ...] # 根据模式订阅

unsubscribe [channel [channel ...]] # 取消订阅
punsubscribe pattern [pattern ...] # 根据模式取消订阅，如果没有提供模式，则取消所有模式订阅

pubsub channels # 查看订阅信息
pubsub cahnnel # 返回该频道订阅者数量
pubsub numpat # 返回当前被订阅模式的数量
```

## Redis配置&其他

```shell
auth password # 使用密码连接redis
echo message # 打印
ping # 向服务器发送ping请求
qiut # 退出连接
select number # 选择数据库

client setname name # 给当前客户端起个名字
client getname # 获取名字
client list # 所有连接服务器的终端信息
client kill ip:port # 关闭指定客户端

# 持久化相关
save # rdb持久化操作，会阻塞客户端
bgsave # 启动子进程执行rdb持久化，不会阻塞
bgrewriteaof # aof文件重写
lastsave # 服务器最后一次持久化的时间戳

# 配置相关
config set option value # 给配置选项设定值
config get option # 获取某项信息
config rewrite # 重写配置，并持久化
config resetstat # 重置服务器的统计数据

info [section] # 返回服务器各项信息
time #当前服务器的unix时间戳
shutdown [save|nosave] # 关闭服务器

# 调试相关
slowlog get [number] #查看慢日志
slowlog len #慢日志条数
slowlog reset #清空慢日志
monitor #启动监视器，打印服务器执行的命令
debug segfault # 让redis段错误，使其崩溃
debug object key # 查看与有关键相关的调试信息
object refcount key #查看键的值被引用的次数
object encoding key # 查看建的值使用的内部表示
object idletime key #查看给定键的空闲时间
```



## 命令行

```shell
redis-server # redis server
redis-cli # client
redis-benchmark # 性能测试
redis-check-rdb  # 检查 RDB
redis-check-aof  # 检查 AOF
redis-sentinel   # 哨兵
```

