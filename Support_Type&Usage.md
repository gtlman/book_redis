# <center>支持的数据类型以及对应的应用场景</center>

## <center>键命令</center>
所有数据类型共有的键命令

```
keys pattern （key *) # 根据正则批量获取键名
exists key [key ...]  # 查看是否存在键
type key              # 查看键类型
expire key seconds    # 设置key过期时间
ttl key               # 查看有效时间/s
```

## <center>String 字符串</center>
存储的是最简单的K-V格式，一般用作短信验证码，配置信息等

字符串类型是Redis中最为基础的数据存储类型，它在Redis中是二进制安全的，这便意味着该类型可以接受任何格式的数据，如JPEG图像数据或Json对象描述信息等。

在Redis中字符串类型的Value最多可以容纳的数据长度是512M。

### 保存 ＆ 修改
如果设置的键不存在则为添加，如果设置的键已经存在则修改

```
set key value
setex key seconds value
append key value
setnx key value       # 只有键不存在，才会执行保存操作
incr key              # 当value是数值类型的时候，递增+1，不是数值类型的时候会报错
decr key              # 当value是数值类型的时候，递减-1
incrby key increment  # 增加increment
decrby key decrement  # 减少decrement
del key1 key2         # 删除键
```

### 获取
```
get key
mget key1 key2 ...
```

## <center>Hash哈希</center>
```
hash⽤于存储对象，对象的结构为属性、值
值的类型为string
```

存储Key为ID这类唯一标志，并且Value是一组K-V的数据 Key-(K-V，K-V...），一般用作商品详情，新闻详情，购物车等

### 保存
```
hset key field value
hmset key field1 value1 field2 value2 ...
hsetnx key value            # 只有键不存在，才会执行保存操作
hincrby key field increment # 属性值增加increment
```

### 获取
```
hkeys key         # 获取key的所有属性名
hget key field    # 获取key某个属性的值
hmget key field1 field2 ...
hvals key         # 获取key所有属性的值
```

### 删除
```
hdel key field1 field2 ...
```

## <center>List列表</center>
```
列表的元素类型为string
按照插⼊顺序排序
```

存储的是Key-List格式，常作为双向队列轻易实现“最新/最旧”功能。一般用作消息中间件的任务队列，有限长度的浏览记等。

### 插入
```
lpush key value1 value2 ...   #左侧插入
rpush key value1 value2 ...   #右侧
linsert key BEFORE|AFTER pivot value  `>>lpush l1 1 2 3 4 5，>>linsert l1 BEFORE 3 10`
```

### 获取
```
lrange key start stop         #start、stop为元素的下标索引，[start, stop]是一个闭区间
lset key index value          #设置指定索引位置的元素值
```

### 删除
```
lrem key count value            #删除指定数目的指定元素
```

## <center>Set集合</center>
```
⽆序集合
元素为string类型
元素具有唯⼀性，不重复
说明：对于集合没有修改操作
```

存储的是Key-Set格式，Set保证数据不重复，提供了交集，并集，差集的功能，但无法根据顺序或序号查询。一般用作实现例如共同好友等用到集合的功能。

### 插入
```
sadd key member1 member2 ...
```

### 获取
```
smembers key                     # 返回集合所有元素
```

### 删除
```
srem key member [member ...]     # 删除集合
```

## <center>Zset有序集合</center>
```
sorted set，有序集合
元素为string类型
元素具有唯⼀性，不重复
每个元素都会关联⼀个double类型的score，表示权重，通过权重将元素从⼩到⼤排序
说明：没有修改操作
```

在Set的基础上加上了score（分数），根据score进行排序，一般用来实现**实时**TopN这类排名功能。

### 插入
```
zadd key score1 member1 score2 member2 ...
zincrby key increment member                #增加成员的score
```

### 获取
```
zrange key start stop       #根据索引下标查询，start、stop为元素的下标索引，[start, stop]是一个闭区间
zrangebyscore key min max   #根据score查询
zscore key member           #返回成员的score值
```

### 删除
```
zrem key member1 member2 ...    #根据名字，删除成员
zremrangebyscore key min max    #根据分数，删除成员
```

## <center>事务</center>
1. DISCARD: 取消事务，放弃执行事务块内的所有命令。
2. EXEC: 执行所有事务块内的命令。
3. MULTI: 标记一个事务块的开始。
4. UNWATCH: 取消 WATCH 命令对所有 key 的监视。
5. WATCH key [key ...]: 监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

redis的事务本质就是将命令打包在一起，在EXEC的时候一起发送出去