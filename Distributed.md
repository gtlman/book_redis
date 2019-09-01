# <center>分布式</center>
## 主从复制
1. 读写分离：通过给机器分配主从关系，主从各司其职，让master只负责写操作，让slave只负责读操作，然后扩展slave就可以高效应付当今分布式环境读多写少的情况。
2. 高可用性：当master出现故障时，slave允许上位，实现了容灾的快速恢复功能。
```
[root@centos7-redis redis]# bin/redis-cli     #进入redis客户端
127.0.0.1:6379> slaveof host port             #主动设置当前数据库属于host:port的从数据库
```

### 新增从机切入点
从级数据库会发送信号给主级数据库，主级数据库收到信号后执行快照并发送RDB文件给从级数据库加载，后续主级数据库收到写操作也会同步发送给从级数据库。

### 主机宕机
默认主从关系不发生变，可通过设置让Slave自动上位成为主机快速恢复，也可以在Slave处执行命令slaveof no one让它成为Master

### 从机宕机
需要重新获取RDB文件，重新建立主从关系

### 哨兵模式
哨兵是redis提供的一个分布式监控系统，它会监控redis的状况，并且定期PING其它哨兵以确认对方状态。如果本机发生宕机，哨兵可以向其它哨兵或者管理员发送信号，如果根据优先级，数据完整性，runid来选举新的master，如果后面原master恢复，则自动变为slave。

假设主服务器宕机，哨兵1先检测到这个结果，系统并不会马上进行failover过程，仅仅是哨兵1主观的认为主服务器不可用这个现象成为主观下线。哨兵之间会定期交换信息，其它哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之就会进行一次投票，投票的结果由一个哨兵发起，进行failover操作。切换成功后，就会通过发布订阅模式，让各个哨兵把己监控的从服务器实现切换主机，这个过程称为客观下线。

## 集群
把多个服务器当作一个整体来提供服务
### 技术目的
1. 存储数据分摊在可扩展的node上，解决存储空间不足问题以及实现了去中心化
2. master的可扩展性解决了读写分离没有解决的并发写操作太多时单个master无法处理的情况

### Redis集群的缺陷
1. 不支持多建操作（mset，mget...）
2. 不支持多键事务和lua脚本
3. 支持集群的版本较晚

### 最低配置
Redis集群模式设定了一个最低配置：最少6台机器分别组成3个Master-Slave集合

### Redis集群安装
集群需要安装Ruby环境：
```
yum install ruby
yum install rubygems
gem install redis（绑定redis的时候可能会出现yum源提供的ruby版本低于redis.gem要求的最低ruby版本，这时需要跟新ruby版本）
```
详情参考：https://blog.csdn.net/u010533511/article/details/89389906 + https://rvm.io/

### 启动Redis集群模式
1. 清空集群中各个node数据
2. 配置redis.conf
- cluster-enabled yes             #开启集群模式
- cluster-node-timeout 15000      #设置超时时间
3. 创建集群
`./redis-trib.rb create --replicas 1 ip1:port1 ip2:port2 ip3:port3....ipN:protN`

命令讲解：create参数创建集群，--replicas参数指定一个master有多少个slave，从左到右的IP:Port先master，再slave（也就是IP1，IP2，IP3...IPN/2是master）

### 哈希槽 hashslot
一个集群有16384个hashslot，集群使用CRC16(key)%16384来计算键key的存放位置，而这些hashslot则按顺序平均分配给集群中的master
```
CLUSTER KEYSLOT key                #返回key对应的slot数
CLUSTER COUNTKEYSINSLOT slotnum    #返回slot中key数目
```