##### 通用操作命令

>keys *  keys key? exists key rename key

>expire 过期时间 ttl 剩余过期时间 type key 类型

>move key table 将key移动到指定数据库

>multi 开启事物  exec 提交事物  discard 回滚

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

##### 集群搭建
>redis conf 配置修改：

    post修改、daemonise打开、pidfile修改、logfile修改、
    appendonly打开、cluster-enabled打开、
    cluster-config-file修改、cluster-node-timeout（失联时间）
    bind绑定去掉，protected mode也要关掉

>集群创建方式

+ ruby: redis-trib.rb create --replicas 1 ip:port*
- redis5新特性：redis-cli --cluster create ip:port* --cluster-replicas 1
+ 主/从 比率 = 1 cluster-replicas 1

>集群模式打开client:
    redis-cli -c -h host -p port

>集群扩容和删除

    扩容：redis-cli --cluster add nodes ip:port
    主节点：redis cli --cluster reshard ip:port
    从节点：客户端执行：cluster replicate redis节点ID

    删除：
    主节点：redis cli --cluster reshard ip:port
    再执行删除从节点的命令
    redis cli --cluster del-node ip:port 节点ID

>持久化的方式

    AOF 的优缺点
    优点：数据的完整性和一致性更高
    缺点：因为AOF记录的内容多，文件会越来越大，数据恢复也会越来越慢。

    RDB 的优缺点
    优点：
    1 适合大规模的数据恢复。
    2 如果业务对数据完整性和一致性要求不高，RDB是很好的选择。

    缺点：
    1 数据的完整性和一致性不高，因为RDB可能在最后一次备份时宕机了。
    2 备份时占用内存，因为Redis 在备份时会独立创建一个子进程，将数据写入到一个临时文件（此时内存中的数据是原来的两倍哦），最后再将临时文件替换之前的备份文件。
    所以Redis 的持久化和数据的恢复要选择在夜深人静的时候执行是比较合理的。



