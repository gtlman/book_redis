# <center>支持的数据类型以及对应的应用场景</center>
##String 字符串
存储的是最简单的K-V格式，一般用作短信验证码，配置信息等
##Hash哈希
存储Key为ID这类唯一标志，并且Value是一组K-V的数据 Key-(K-V，K-V...），一般用作商品详情，新闻详情，购物车等
##List列表
存储的是Key-List格式，常作为双向队列轻易实现“最新/最旧”功能。一般用作消息中间件的任务队列，有限长度的浏览记等。
##Set集合
存储的是Key-Set格式，Set保证数据不重复，提供了交集，并集，差集的功能，但无法根据顺序或序号查询。一般用作实现例如共同好友等用到集合的功能。
##Zset有序集合
在Set的基础上加上了score（分数），根据score进行排序，一般用来实现**实时**TopN这类排名功能。