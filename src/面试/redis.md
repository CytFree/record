##### 通用操作命令

>keys *  keys key? exists key rename key

>expire 过期时间 ttl 剩余过期时间 type key 类型

##### 字符串操作命令
>get set getset del

>incr decr incrby descby

##### HASH操作命令
>hset hget hdel

>hmset hmget

>hgetall hkeys hvals hlen

>hincrby hexists

##### LIST操作命令

>lpush lpop

>rpush rpop

>llen lrange

>lpushx(rpushx) key value 在头部插入元素

>lrem key count(大于0后左，大于0从右，等于0删除所有) value 移除多少个value

>lset linsert rpoplpush(消息队列)


##### SET操作命令

>sadd srem smembers sismember

>sdiff 差集 sinter 交集 sunion 并集

>sdiffstore 差集结果放入新的set

>scard 数量 srandmember 随机返回一个元素


##### sorted SET操作命令

>zadd zrem zrange zrevrange zcard

>zrangebyrank zrangebyscore

>zscore zcount


