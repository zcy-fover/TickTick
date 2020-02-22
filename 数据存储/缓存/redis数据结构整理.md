## Redis 数据结构

- 扁平化的特点，不存在数据库中列表查询一类的操作

- ### 数据结构

> - #### Value对象的通用数据结构
>
> > ```c
> > typedef struct redisObject {
> >     unsigned type:4;
> >     unsigned encoding:4;
> >     unsigned lru:REDIS_LRU_BITS;
> >     int refcount;
> >     void *ptr;
> > }robj;
> > ```
> >
> > - **type**：指String、List等结构化类型
> > - **encoding**：encoding指的是这些接口规划类型具体的实现（承载）方式，同一个类型可以有多个实现方式，例如String可以用int来承载，也可以用封装的cha[]来承载，List可以用ziplist或者链表来承载
> > - **lru**：表示本对象的空转时长，用于有限内存下长久不访问的对象的清理
> > - **refcount**：应用计数用于对象的垃圾回收
> > - **ptr**：指向的是以encoding方式实现这个对象的实际承载者的地址，例如String对象对应的sds地址
>
> - #### String
>
> >- Redis中的String可以表示字符串、整数、浮点数三种类型，由Redis完成相互间的自动转型
> >
> >- ##### 基本操作：
> >
> > > ![Redis-String操作](https://note.youdao.com/yws/api/personal/file/WEBda5af15f4bd8cfc917bf6764295d3d05?method=download&shareKey=c5e74b3ff893de9a0f9e53fffce19803)
>
> - #### List
>
> > - ##### 基本操作：
> >
> > > - RPUSH/LPUSH：RPUSH将元素添加在列表尾，L则是将元素添加在列表头部
> > > - RPOP/LPOP：取出给定key的列表的尾部或头部的元素并删除元素
> > > - LINDEX：去除给定的key对应列表索引的某个元素
> > > - LRANGE：取出给定key的索引范围内的元素，例如LRANGE key1 0, 3即取出前四个元素
> > > - LTRIM：将给定key的列表索引范围的元素去除
> > > - BLPOP/BRPOP：BRPOP key1 key2 60，60秒内，key1非空则从key1对应的列表中pop最右元素，否则从key2中pop最右元素；如果60秒内两个列表始终为空，则超时返回
> > > - BLPOPPUSH/BRPOPPUSH：BRPOPPUSH key1 key2 60即60秒内如果key1对应的列表非空，则把key1列表的最右元素pop，并且放到key2最后
>
> - #### Map：
>
> > - ##### 基本操作：
> >
> > > - HGET：返回给定key， field的值
> > > - HSET：设置给定key，field的值
> > > - HMGET：返回给定key，field1、field2...的值
> > > - HMSET：设置给定key，多个field的值
> > > - HGETALL：获取给定key的所有field和value
> > > - HDEL：删除给定key， field元素
> > > - HKEYS：获取给定的key的所有field名字
> > > - HVALS：获取给定key的所有value
> > > - HLEN：获取给定的key的字段数量
> > > - HINCRBY：HINCRBY key field increment，给定key的field元素value自增整数
> > > - HINCRBYFLOAT：HINCRBYFLOAT key field increment ，给定key的field元素value自增浮点数
> > > - HEXISTS：检查指定key field是否存在
> > > - HSETNX：HSETNX key field value只有当field字段不存在时，设置该元素
>
> - #### Set：
>
> > - ##### 基本操作：
> >
> > > - SADD/SREM/SISMEMBER：实现向SET中增加、删除元素，以及检查元素是否存在
> > > - SCAD/SMEMBERS/SRANDMEMBER：实现统计元素个数、列出所有元素、随机获取元素的操作
>
> - #### Sorted-Set：
>
> > - ZRANK：确定某个KEY值在本sorted-set内按照顺序排在第几位
> > - ZRANGE：例如ZRANGE key start stop，获取sorted-set中排名为start和stop间的数据
> > - ZRANGESCOPE：ZRANGEBYSCOPE key min max获取sorted-set中scope介于min和max之间的数据
> > - ZSCOPE：确定某个key值在本sorted-set内对应的value
> > - ZINCRBY

<div style="text-align:center;margin-top:50px;margin-bottom:50px;">
    <img src="https://note.youdao.com/yws/api/personal/file/C2C6FCFDC10942B6A3532E6F0928E455?method=download&shareKey=c554dacfc5193c29d4b35682aa1226d9" />
</div>
