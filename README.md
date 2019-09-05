# <center>Introduction</center>
## 安装

到[redis官方文档](https://redis.io/ "redis官方文档")下载安装包`redis-xxx.tar.gz`文件, 或者像下面那样直接用wget去下载

```
wget http://download.redis.io/releases/redis-5.0.4.tar.gz
tar xzf redis-5.0.4.tar.gz
cd redis-5.0.4
make			//编译源文件，如果中途中断了，则运行make distclean复原一下
mkdir -m755 /usr/local/redis
make install PREFIX=/usr/local/redis	        //将安装文件存放在指定目录，会在Redis下创建bin目录
mkdir -m755 /usr/local/redis/conf               //创建conf目录存放配置文件
mkdir -m755 /usr/local/redis/log                //创建log目录存放配置文件
mkdir -m755 /usr/local/redis/data                //创建data目录存放配置文件
cp /opt/software/redis-5.0.4/redis.conf /usr/local/redis/conf/redis.conf    //拷贝模板到conf目录

vim redis.conf
    将daemonize no 改为yes                        //设置后台启动（也就是将用户进程转为守护进程，或系统服务）
    设置logfile为'/usr/local/redis/log/redis_6379.log'     //需要提前创建log目录）
    设置最大内存maxmemory为751619276，也就是0.75G           //这个按需分配
    设置持久化数据存放位置dir为/usr/local/redis/data/       //需要提前创建data目录)
    远程连接redis：修改允许访问的IP：bind IP1 IP2 ...或者直接注释掉->代表允许所有IP访问
	关闭保护模式：在配置文件中找到protected-mode参数，设置为no（保护模式限制了必须要bind ip或者使用密码访问）

redis-server /usr/local/redis/conf/redis.conf       //指定配置文件启动redis服务端
redis-cli：进入客户端	quit/shutdown
```