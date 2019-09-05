# <center>持久化</center>
## <center>快照 RDB</center>
快照是redis默认的持久化策略
###配置项
```
save 900 1                             #执行快照，after 900 sec (15 min) if at least 1 key changed
stop-writes-on-bgsave-error yes        #最后一次快照失败后，停止接受写操作
rdbcompression yes                     #执行快照的时候，对String进行压缩
rdbchecksum yes                        #校验快照，代价是存储和读取RDB文件时需要耗费10%的性能
dbfilename dump.rdb                    #RDB文件命名
dir /usr/local/redis/data/             #工作目录，rdb文件和aof文件都存放在该目录下
```

### 写时复刻
redis在执行快照的时候会fork一个子进程，利用了写时复刻技术，写时复刻就是当子进程和父进程的内存空间一致的时候会让他们共用同一段内存空间，直到出现差别。在这里就是当redis父进程没有操作的时候出现共用现象，提高了资源利用率。

## <center>追加 Append Only File->AOF</center>
redis默认不开启AOF，可以与RDB同时开启，启动时会以AOF为准（因为AOF的更新频率较高）
###配置项
```
appendonly no                       #默认不开启
appendfilename "appendonly.aof"     #AOF文件命名
appendfsync everysec                #三种策略always，everysec，no

auto-aof-rewrite-percentage 100     #当增长率为100%时，条件一
auto-aof-rewrite-min-size 64mb      #限制最小重写大小，条件二
上述条件一和条件二为重写策略，重写调用bgrewriteof，会删减重复的修改，压缩AOF文件
```

## <center>RDB vs AOF</center>
|RDB|AOF|
|:-:|:-:|
|节省磁盘，RBD会压缩String类型数据|虽然也有重写功能，但文件依然较大|
|直接读取数据文件，恢复速度快|需重新执行写操作，恢复速度慢|
|稳定|有BUG|
|数据文件人不可读|追加文件可读，可人工修改|
|备份频率相较低，容易丢失修改|追加的频率高，丢失修改的概率低|
|数据量大情况下执行快照耗费大量性能|只需追加写操作，耗费新能少|
