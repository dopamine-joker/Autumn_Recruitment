## redis

## 1. 基本数据类型

string,list,hash,set,zset

(1) string

一个Key对应一个value,string是二进制安全的，redis中的string可以包含任何数据，如jpg图片或者序列化的对象。string类型最大能存储**512MB**

常用命令: `incr`(自增), `decr`(自减), `incrby`(加), `decrby`(减),`set`,`get`,`mget`(multiget，一次获取多个key的value)

```shell
127.0.0.1:6379> set string1 2
OK
127.0.0.1:6379> incr string1
(integer) 3
127.0.0.1:6379> decr string1
(integer) 2
127.0.0.1:6379> incrby string1 5
(integer) 7
127.0.0.1:6379> get string1
"7"
127.0.0.1:6379> 
```

(2) list

list列表是简单的字符串列表，按照插入顺序排序(队列)。可以从左边或者右边添加，pop元素

常用命令: `lpush`,`rpush`,`lpop`,`rpop`,`lrange`（遍历）, `rpoplpush`(从一个list的rpop出元素，并把元素lpush到另外一个list),`blpop`(阻塞pop),`lren`(移除元素)

```shell
127.0.0.1:6379> lrange my_list 0 100
1) "14"
2) "13"
127.0.0.1:6379> lrange other_list 0 100
1) "12"
127.0.0.1:6379> rpoplpush my_list other_list
"13"
127.0.0.1:6379> lrange my_list 0 100
1) "14"
127.0.0.1:6379> lrange other_list 0 100
1) "13"
2) "12"
```

(3) hash

哈希是键值对集合，形式一般为(key->(field->value))。redis是一个string类型的field和value的映射表，适合用来存储对象。

常用命令`hget`, `hset`, `hgetall`（获取某个key的全部(field,value)）,`hscan`

> hgetall实际业务中用hscan会好一点，因为redis是单线程的，当key中的field数量过多时会阻塞到其他查询

```shell
127.0.0.1:6379> hset hash1 field1 value1 field2 value2 field3 value3
(integer) 3
127.0.0.1:6379> hget hash1 field2
"value2"
127.0.0.1:6379> hgetall hash1
1) "field1"		## field1
2) "value1"		## value1
3) "field2"
4) "value2"
5) "field3"
6) "value3"
127.0.0.1:6379> hmget hash1 field1 field3
1) "value1"
2) "value3"
```

(4) set

set是string的无序去重集合，概念和数据中集合相似，可以交集并集差集等。set中元素没有顺序。添加删除查找复杂度都是O(1)。其底层实现是一个value永远为null的hashMap,这样就可以通过hash快速定位元素。

常用命令: `sadd`,`spop`,`smembers`(获取全部成员),`sunion`(并集),`sinter`(交集),`sdiff`(差集),`sscan`等

> 与hash同理，实际应用中使用sscan来代替smember

```shell
127.0.0.1:6379> smembers set1
1) "v1"
2) "v2"
127.0.0.1:6379> smembers set2
1) "v3"
2) "v2"
127.0.0.1:6379> sunion set1 set2
1) "v1"
2) "v3"
3) "v2"
127.0.0.1:6379> sinter set1 set2
1) "v2"
127.0.0.1:6379> sdiff set1 set2
1) "v1"
127.0.0.1:6379> sdiff set2 set1
1) "v3"
```

(5) zset

有序集合(sorted set)，zset和set一样是也是string类型的集合，不允许重复成员。

不过set中只保存单独一个元素(hashmap,value永远为null)，而zset中每个元素都会对应一个`double`类型的分数(score)。zset正是通过这个score来对元素进行排序。

zset中，元素是唯一的，但是元素对应的score是可以重复的。

常用操作:`zadd`, `zcart`(查询个数),`zrange`

```shell
127.0.0.1:6379> zadd zset 1 redis
(integer) 1
127.0.0.1:6379> zadd zset 2 mongodb
(integer) 1
127.0.0.1:6379> zadd zset 3 mysql
(integer) 1
127.0.0.1:6379> zadd zset 3 mysql
(integer) 0
127.0.0.1:6379> zadd zset 3 nosql
(integer) 1
127.0.0.1:6379> zadd zset 3 anosql
(integer) 1
127.0.0.1:6379> zrange zset 0 10 WITHSCORES
 1) "redis"
 2) "1"
 3) "mongodb"
 4) "2"
 5) "anosql"
 6) "3"
 7) "mysql"
 8) "3"
 9) "nosql"
10) "3"
```

## 2. 基本数据应用场景

| 类型                 | 简介                                                   | 特性                                                         | 场景                                                         |
| -------------------- | ------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| String(字符串)       | 二进制安全                                             | 可以包含任何数据,比如jpg图片或者序列化的对象,一个键最大能存储512M | \---                                                         |
| Hash(字典)           | 键值对集合,即编程语言中的Map类型                       | 适合存储对象,并且可以像数据库中update一个属性一样只修改某一项属性值(Memcached中需要取出整个字符串反序列化成对象修改完再序列化存回去) | 存储、读取、修改用户属性                                     |
| List(列表)           | 链表(双向链表)                                         | 增删快,提供了操作某一段元素的API                             | 1、最新消息排行等功能(比如朋友圈的时间线) 2、消息队列        |
| Set(集合)            | 哈希表实现,元素不重复                                  | 1、添加、删除、查找的复杂度都是O(1) 2、为集合提供了求交集、并集、差集等操作 | 1、共同好友 2、利用唯一性,统计访问网站的所有独立ip 3、好友推荐时,根据tag求交集,大于某个阈值就可以推荐 |
| Sorted Set(有序集合) | 将Set中的元素增加一个权重参数score,元素按score有序排列 | 数据插入集合时,已经进行天然排序                              | 1、排行榜 2、带权重的消息队列                                |

## 3.高级数据结构

1. HyperLogLog

    用来基数统计，比如用户流量统计。

    常用命令: `pfadd`, `pfcount`, `pfmerge`(合并两个统计)

    ```shell
    127.0.0.1:6379> pfadd user doper1	# (integer) 1
    127.0.0.1:6379> pfadd user doper1	# (integer) 0
    127.0.0.1:6379> pfadd user doper2	# (integer) 1
    127.0.0.1:6379> pfadd player doper1	# (integer) 1
    127.0.0.1:6379> pfadd player doper3	# (integer) 1
    127.0.0.1:6379> pfcount user player	# (integer) 3
    127.0.0.1:6379> pfcount user	# (integer) 2
    127.0.0.1:6379> pfcount player	# (integer) 2
    127.0.0.1:6379> pfmerge pv user player	# OK
    127.0.0.1:6379> pfcount pv	# (integer) 3
    ```

    使用pf开头的原因是因为这个数据结构的发明人叫Philippe Flajolet,



