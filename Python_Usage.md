# <center>与Python交互</center>
主要涉及python以及django如何使用redis

## <center>与Python交互</center>
需要用到`redis-py`模块

### 安装
`pip install redis`

### 使用方式
几乎是完全模拟redis客户端的操作

redis-py3.0只支持bytes,str,number类型的数据，其它的数据redis-py都会抛出DataError异常

```
import redis
r = redis.Redis(host='localhost', port=6379, db=0)
r.set('foo', 'bar')         #插入成功返回True, 如果插入失败则会返回False
r.get('foo')                #返回'bar'，如果是空则啥返回None
r.mset({'s1':1,'s2':2})     #返回True
r.msetnx({'s1':1,'s2':2})   #返回False
r.zadd('zs1', {'s1':1,'s2':2}, nx=True, xx=False, ch=True, incr=False)      #返回2
#因为默认参数必须在后面，因此上述操作跟客户端在参数顺序有出入
其它zincrby啥的都跟原客户端用法一致
```

***注意事项***：
```
SELECT: 没有实现，无法切换数据库
DEL： del 与 python 的关键字冲突了，因此用 delete 替换----->r.delete('foo') #成功则返回1，失败则返回0
MULTI/EXEC: 默认实现为 Pipeline 管道类的一部分，可以通过transaction=False取消
```

### Locks锁
redis-py3.0不再支持基于管道的锁，现在只支持基于LUA的锁，原来的LuaLock已改名为Lock，也因此redis-py3.0要求服务器最低是2.6版本以上

通过上下文进行加锁操作，如果超过blocking_timeout的时间还没获取锁就会抛出LockError异常

```
try:
    with r.lock('my-lock-key', blocking_timeout=5) as lock:
        # code you want executed only after the lock has been acquired
except LockError:
    # the lock wasn't acquired
```

### Pipelines 管道
是redis类的子类，通过将命令缓存，然后一次性打包批量提交，来提高性能

```
r = redis.Redis(...)
r.set('bing', 'baz')
# 通过pipeline方法创建一个pipeline类
pipe = r.pipeline()
# 以下命令都会被缓存
pipe.set('foo', 'bar')
pipe.get('bing')
# 调用execute方法会将所有缓存的命令发送并返回结果
# 返回结果会组合成list并返回
pipe.execute()  #[True, None]

# 而且为了方便，pipeline类还支持链式
pipe.set('foo', 'bar').sadd('faz', 'baz').incr('auto_number').execute() #返回[True, 1, 1]
```

##  <center>与Django的交互</center>
[django-redis官方中文文档](https://django-redis-chs.readthedocs.io/zh_CN/latest/ "django-redis官方中文文档")

### 安装
pip install django-redis

所有版本的 django-redis 基于 redis-py >= 2.10.0，因此也需要安装上面那个redis-py

### Django配置
```
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/1",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
            "PASSWORD": "mysecret"
        }
    }
}
```

Django 会将上述redis库作为 session 储存后端

原生客户端
```
from django_redis import get_redis_connection
con = get_redis_connection("default")       #con就可以像上面的连接一样操作了
```
