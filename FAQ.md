# 为什么使用 Redis
主要从 **性能** 和 **并发**两个角度将 Redis 作为缓存数据库使用。

性能方面，Redis 读写性能优异（内存数据库），大大提高了响应速度，特别是用来存储 **耗时** 并且 **修改频率低** 的数据。

并发方面，在高并发情况下，请求直接到达数据库，数据库会因为连接数高而出现异常。Redis 通过缓存数据让请求先访问 Redis 来实现缓冲。  

# Redis 有什么缺点
存储空间受内存限制，无法用作海量数据的存储

# Redis 作为缓存数据库可能会遇到什么问题？
## 缓存和数据库双写一致性问题
三种策略：
1. 先更新数据库，再更新缓存
2. 先删除缓存，再更新数据库
3. 先更新数据库，再删除缓存
4. 为什么没有 先更新缓存再更新数据库？

### 先更新数据库，再更新缓存
问题：
1. 并发写操作的场景下：A,B两个请求都进行了写操作，可能会因为网络等原因导致 A1,B1,B2,A2 的错误操作顺序而导致**缓存脏数据**
2. 在写操作频繁，读操作较少的场景下（那为什么还用 Redis 缓存数据），Redis 的缓存都还没啥查询就被频繁更新，浪费性能

### 先删除缓存，再更新数据库
补充：查询失效的时候再更新缓存

问题：
并发读写的场景下会导致：
1. 请求 A 进行写操作，删除缓存
2. 请求 B 查询，发现缓存不存在（A 又还没更新数据库）
3. 请求 B 去数据库查询得到旧值(**查询脏数据**)
4. 请求 B 将旧值写入缓存（**缓存脏数据**）
5. 请求 A 将新值写入数据库
延时双删策略
- 删除缓存->更新数据库->等待（一个读数据业务逻辑的耗时基础上加几百ms，如果数据库使用了读写分离，那还需要再加上 主从同步 的几百ms）->再次删除缓存从而规避上述问题。
并发读写的时候查询结果是旧值。
- 可以将 等待以及二次删除 异步实现，从而降低每次请求的处理时间，从而提高吞吐量
- 但是如果第二次删除失败，又会导致长时间的脏数据，会有很严重的后果（哪一步操作失败都会会导致严重的后果吧，后面有讲）

### 先更新数据库，再删除缓存
问题：
1. 缓存刚好失效
2. 请求A查询数据库，得一个旧值(**查询脏数据**)
3. 请求B将新值写入数据库
4. 请求B删除缓存
5. 请求A将查到的旧值写入缓存(**缓存脏数据**)

不过这种概率非常低，因为这要求 3写操作 要比 2读操作 快（只有可能是正在写快写完了，读操作到了的情况），这个也可以用 延时双删 解决。

### 缓存删除失败怎么办？
重试机制：
将失败的 key 加入消息队列，后台线程不断消费循环删除
    
### **查询脏数据**
上述三种策略都表明，Redis 作缓存数据库无法避免**查询脏数据**的情况，因此如果对数据具有 **强一致性** 要求（例如银行转账），不能放在缓存中。

## 缓存雪崩问题
同一时间缓存大面积失效，从而导致大量请求直接到数据库引发的数据库连接异常

## 缓存击穿问题
一般是 无脑爬虫 和 恶意黑客，短时间内访问大量缓存中不存在的数据（甚至数据库都不存在的无效请求），从而从而导致大量请求直接到数据库引发的数据库连接异常

## 缓存的并发竞争问题
多个子系统需要同时 set 一个 key
1. 不要求顺序的情况下，可以使用分布式锁
2. 要求顺序的情况下，可以通过队列中间件