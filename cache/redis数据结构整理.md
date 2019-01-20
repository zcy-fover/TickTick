## Redis

- 扁平化的特点，不存在数据库中列表查询一类的操作

- ### 数据结构

> - #### Value 对象的通用数据结构
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
> >- Redis 中的 String 可以表示字符串、整数、浮点数三种类型，由Redis完成相互间的自动转型
> >
> >- ##### 基本操作：
> >
> > > ![Redis-String操作](https://note.youdao.com/yws/api/personal/file/WEBda5af15f4bd8cfc917bf6764295d3d05?method=download&shareKey=c5e74b3ff893de9a0f9e53fffce19803)
>
> - #### List
>
> > - ##### 基本操作：
> >
> > > - RPUSH/LPUSH：RPUSH 将元素添加在列表尾，L 则是将元素添加在列表头部
> > > - RPOP/LPOP：取出给定 key 的列表的尾部或头部的元素并删除元素
> > > - LINDEX：去除给定的 key 对应列表索引的某个元素
> > > - LRANGE：取出给定 key 的索引范围内的元素，例如 LRANGE key 0, 3 即取出前四个元素
> > > - LTRIM：将给定 key 的列表索引范围的元素去除
> > > - BLPOP/BRPOP：BRPOP key1 key2 60，60秒内，key1 非空则从 key1 对应的列表中 pop 最右元素，否则从 key2 中 pop 最右元素；如果 60 秒内两个列表始终为空，则超时返回
> > > - BLPOPPUSH/BRPOPPUSH：BRPOPPUSH key1 key2 60 即 60 秒内如果 key1 对应的列表非空，则把 key1 列表的最右元素 pop，并且放到 key2 最后
>
> - #### Map：
>
> > - ##### 基本操作：
> >
> > > - HGET：返回给定 key， field 的值
> > > - HSET：设置给定 key，field 的值
> > > - HMGET：返回给定 key，field1、field2 ...的值
> > > - HMSET：设置给定 key，多个 field 的值
> > > - HGETALL：获取给定 key 的所有 field 和 value
> > > - HDEL：删除给定 key， field 元素
> > > - HKEYS：获取给定的 key 的所有 field 名字
> > > - HVALS：获取给定 key 的所有 value
> > > - HLEN：获取给定的 key 的字段数量
> > > - HINCRBY：HINCRBY key field increment，给定 key 的 field 元素 value 自增整数
> > > - HINCRBYFLOAT：HINCRBYFLOAT key field increment ，给定 key 的 field 元素 value 自增浮点数
> > > - HEXISTS：检查指定 key field 是否存在
> > > - HSETNX：HSETNX key field value 只有当 field 字段不存在时，设置该元素
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
> > - ZRANK：确定某个 KEY 值在本 sorted-set 内按照顺序排在第几位
> > - ZRANGE：例如ZRANGE key start stop，获取 sorted-set 中排名为 start 和 stop 间的数据
> > - ZRANGESCOPE：ZRANGEBYSCOPE key min max获取 sorted-set 中 scope 介于 min 和 max 之间的数据
> > - ZSCOPE：确定某个 key 值在本 sorted-set 内对应的 value
> > - ZINCRBY