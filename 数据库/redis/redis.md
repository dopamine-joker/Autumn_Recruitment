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

zset中，**元素是唯一的**，但是元素对应的score是可以重复的。

常用操作:`zadd`, `zcard`(查询个数),`zrange`

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



## 4. 简单动态字符串SDS(simple dynamic string)

eg:

```shell
redis> SET msg "hello world"
OK
```

redis会创建一个键值对，其中键值对的键和值就是SDS对象。

![image-20210908161200703](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908161200703.png)

1. SDS优化字符扩展内存重分配策略

    (1) **空间预分配**

    - 修改后SDS长度小于1MB,则分配和其长度一样的未使用空间。如修改后len(SDS)=13Byte,则分配13Byte未使用空间。这时候实际的SDS buf数组长度为13+13+1('\0')Byte
    - 修改后大于1MB,则分配1MB未使用空间。如修改后len(SDS)为10MB,则分配1MB未使用空间，大小变为30MB+1MB+1Byte('\0') 

    (2) **惰性空间释放**

    - SDS API是缩短字符串时，多出来的空间不回立马回收，而是用free属性保存这些字节的数量。

2. SDS是二进制安全的，即所有SDS API都会以**处理二进制**的方式来处理SDS中buf数组数据。

3. SDS和C字符串的区别

    ![image-20210908161210091](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908161210091.png)

## 5. 链表

eg:

```shell
redis> LLEN integers
(integer) 1024
redis > LRANGE integers 0 2
1) "1"
2) "2"
3) "3"
```

1. 链表节点

![image-20210908161217230](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908161217230.png)

2. 链表

    ![image-20210908161227675](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908161227675.png)

    ![image-20210908161240821](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908161240821.png)

    3. 链表的特性

    - 双端: 链表节点有prev和next指针，访问节点前后节点时间复杂度O(1)
    - 无环: 表头节点的prev指针和表尾节点的next指针都为NULL，对链表访问以NULL为终点
    - 带有表头指针和表尾指针: 链表获取表头结点和表尾节点的复杂度为O(1) 
    - 有长度计数器，获取list长度复杂度为O(1)
    - 多态: 链表节点使用**void*来保存节点值**，可以通过list的dup,free,match为节点值**设置类型特定函数**，所以**链表可以用于保存各种不同类型的值**

## 6. 字典

eg:

```shell
redis> SET msg "hello world"
OK
```

redis会创建一个键为"msg",值为"hello world" SDS对象 的键值对，这个键值对就是保存在数据库的**字典**里面

dict字典保存了数据库中所有的键值对

eg:

hash中的键值对

```shell
redis> HLEN website
(integer) 10086

redis> HGETALL website
1) "Redis"
2) "Redis.io"
1) "MariaDB"
2) "MariaDB.org"
...
```

website键的底层实现就是一个字典，字典中包含了10086个键值对。

1. **字典底层实现**

    哈希表，一个哈希表里面有多个哈希节点，每个哈希节点就保存了字典中的一个键值对。

2. **哈希表**

    ![image-20210908161304955](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908161304955.png)

    

table是一个数组，数组中的每一个元素指向`dict.h/dictEntry`结构的指针，每一个`dictEntry结构`保存一个键值对。

![image-20210908161313171](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908161313171.png)

3. **哈希表节点**

![image-20210908161324797](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908161324797.png)

![image-20210908161331860](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908161331860.png)

4. **字典**

![image-20210908161757908](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908161757908.png)

![image-20210908161811546](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908161811546.png)

![image-20210908161817971](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908161817971.png)

5. **哈希算法**

    添加键值对(k0,v0)步骤

    ![image-20210908161841096](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908161841096.png)

6. **rehash**

    ![image-20210908161916513](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908161916513.png)
    
    ![image-20210908162025236](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908162025236.png)

## 7. 跳表

eg:

```shell
redis> ZRANGE fruit-price 0 2 WITHSCORES
1) "banana"
2) "5"
3) "cherry"
4) "6.5"
5) "apple"
6) "8"
```

fruit-price有序集合中的所有数据保存在一个**跳跃表**里面，其中每个跳跃表节点(node)都保存了水果价钱信息(分值)。

1. Redis用跳表的地方
    - 有序集合键
    - 集群节点中用作内部数据结构

2. 跳跃表节点

    ![image-20210908162108485](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908162108485.png)

    ![image-20210908162115464](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908162115464.png)

    > 虚线是遍历的走向

3. 跳跃表

    ![image-20210908162319363](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908162319363.png)

## 8. 整数集合

eg:

当我们创建一个只包含五个元素的集合键，并且集合中的所有元素都是**整数值**，那么这个集合键的底层实现会是**整数集合**

```shell
redis> SADD numbers 1 3 5 7 9
(integer) 5
redis> OBJECT ENCODING numbers
"intset"
```

## 9. 压缩列表

TODO

## 10. 对象

Redis使用对象表示数据库中的键和值，每次新建一个键值对时，至少会创建两个对象，一个对象用作键值对的键，另一个用作键值对的值（值对象）。

eg:

```shell
redis> SET msg "hello world"
OK
```

Redis每个对象都由redisObject结构表示。

![image-20210908162340155](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908162340155.png)

对于redis数据库保存的键值对，**键总是一个字符串对象**，而值可以是**字符串对象**，**列表对象**，**哈希对象**，**集合对象**或**有序集合对象**

对于type字段记录对象的类型，其中的值如下:

![image-20210908162348282](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908162348282.png)

对于TYPE命令返回的就是数值库**键**对应的**值对象的类型**，而不是键对象的类型。

![image-20210908162410466](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908162410466.png)

### 1. **字符串对象(string)**

字符串对象的编码可以是**int**,**raw**或者**embstr**

- int编码

    如果字符串对象保存的是整数值，并且整数值可以用long表示，那么字符串对象会将整个整数值保存在字符串对象的ptr属性里面(将void*转换成long)，并将字符串对象的编码设置为int

    ```sh
    redis> SET number 10086
    OK
    redis> OBJECT ENCODING number
    "int"
    ```

    ![image-20210908162503121](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908162503121.png)

- raw编码

    如果字符串对象长度大于**32字节**，则使用SDS保存，并设置编码为**raw**

    ```sh
    redis> SET story "Long, long ago there lived a king ..."
    OK
    redis> STRLEN story
    (integer)37
    redis> OBJECT ENCODING story
    "raw"
    ```

    ![image-20210908162530338](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908162530338.png)



- embstr编码

    如果字符串对象值长度小于等于32字节，则使用**embstr**编码方式保存

    embstr编码专门用于保存短字符串，这种编码和raw编码一样也是使用redisObject和sdshdr结构，但是embstr通过**一次内存分配**分配一块连续空间存储redisObject和sdshdr结构，而raw要调用两次内存分配来分别创建。

    ![image-20210908162611899](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908162611899.png)

    ![image-20210908162604809](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908162604809.png)

    ```sh
    redis> SET msg "hello"
    OK
    redis> OBJECT ENCODING msg
    "embstr"
    ```

    注意long double类型表示的浮点数也是使用字符串值来保存。

    ![image-20210908162648980](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908162648980.png)

### 2. 列表对象(list)

列表对象的编码可以是**ziplist**或**linkedlist**，但是在Redis3.2开始使用**quicklist**代替前面两者

- ziplist

    满足下列情况使用ziplist

    (1) 列表对象保存所有字符串的长度**都小于64字节**

    (2) 列表对象保存元素的数量**小于512个**

    ![image-20210908163300730](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908163300730.png)

- linkedlist

    不能同时满足ziplist使用条件的就使用linkedlist

    ![image-20210908163250017](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908163250017.png)

- **quicklist**

    quicklist 实际上是 zipList 和 linkedList 的混合体，它将 linkedList 按段切分，每一段使用 zipList 来紧凑存储，多个 zipList 之间使用双向指针串接起来

![img](https://hunter-image.oss-cn-beijing.aliyuncs.com/redis/quicklist/QuickList.png)

### 3. 哈希对象hash

哈希对象的编码可以是**ziplist**或者**hashtable**

- **ziplist**

    ziplist编码的哈希对象使用**压缩列表**作为底层实现时，每当有新的键值对要加入哈希表，程序就先将保存了键的压缩列表节点推入到压缩列表的表尾，然后再保存了值得压缩列表节点推入到压缩列表表尾，因此，

    - **保存了统一键值对得两个节点总是紧挨在一起，保存键得节点在前，保存值的节点在后**
    - **先添加到哈希对象的键值对会放在压缩列表的表头方向，而后来添加到哈希对象中的键值会放在压缩列表的表尾方向**

    ![image-20210908163555731](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908163555731.png)

    如果profile键创建的是ziplist编码对象

    ![image-20210908163529031](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908163529031.png)

    当满足下列情况，使用ziplist编码
    (1) 哈希对象保存的所有键值对的键和值的字符串长度都小于64字节

    (2) 哈希对象保存的键值对数量小于512

- **hashtable**

    hashtable编码的哈希对象使用<font color=red>**字典**</font>作为底层实现，哈希对象中的**每个键值对**都使用一个**字典键值**保存

    - 字典的每个键都是一个字符串对象，对象中保存了键值对的键
    - 字段的每个值都是一个字符串对象，对象中保存了键值对的值

    如果profile键创建的是hashtable对象

    ![image-20210908163609739](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908163609739.png)

    不满足使用ziplist编码条件的使用hashtable

### 4. 集合对象set

集合对象的编码可以是**intset**或**hashtable**

- intset

    intset编码的集合对象使用整数集合作为底层实现，集合对象包含的所有元素都保存在整数集合里面

    同时满足下面条件则使用Intset

    - 集合对象保存的所有元素都是整数值
    - 集合对象保存的元素数量不超过512个

- hashtable

    hashtable编码的集合对象使用<font color=red>**字典**</font>作为底层实现，字典的每个键都是一个字符串对象，每个字符串对象包含了一个集合元素，则**字典的值则全部被设置为NULL**

```shell
redis> SADD numbers 1 3 5
(integer)3
redis> SADD Dfruits "apple" "banana" "cherry"
(integer)3
```

![image-20210908163717907](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908163717907.png)

### 5. 有序集合对象zset

有序集合的编码可以是**ziplist**或**skiplist**

- **ziplist**

    ziplist编码底层使用**压缩列表**作为底层实现，每个集合元素使用两个紧挨在一起的压缩列表节点来保存，第一个节点保存元素的**成员(member)**，第二个元素则保存元素的**分值(score)**

    同时满足以下条件，使用ziplist编码

    - 有序集合保存的元素个数小于128个
    - 有序集合保存的所有元素成员的长度都小于64字节

```sh
redis> ZADD price 8.5 apple 5.0 banana 6.0 cherry
(integer) 3
```

如果price使用ziplist编码，则

![image-20210908163825995](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908163825995.png)

![image-20210908163818482](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908163818482.png)

- **skiplist**

    skiplist编码的有序集合底层使用**zset数据结构**实现，一个zset包含一个**字典**和一个**跳跃表**

    ```c
    typedef struct zset{
        zskiplist *zsl;
        dict *dict;
    } zset
    ```

    ![image-20210908163921572](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908163921572.png)

    如果上述price使用skiplist实现，则

    ![image-20210908164121022](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908164121022.png)

    值得注意的是，其实**zset中的跳跃表和字典都会通过指针来共享相同的元素的成员和分值**，不会造成额外空间浪费

    使用字典的原因是可以以O(1)复杂度查找某个成员的分值

    ![image-20210908164222515](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908164222515.png)
    
    ![image-20210908163906516](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908163906516.png)

### 6. 内存回收

使用引用计数



### 7. 对象共享

键A和键B都创建一个包含整数值100的字符串对象

![image-20210908164233920](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908164233920.png)

![image-20210908164246226](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908164246226.png)

redis的对象引用，只对value >= 0 && value < OBJ_SHARED_INTEGERS的数值类对象生效，除此之外的其他redis对象，都不会相互引用。OBJ_SHARED_INTEGERS 在系统中默认是10000。为了标记这种可被复用的对象，引用计数值会被标记成 INT_MAX

# 11. redis单线程是什么意思

原文链接: https://www.nowcoder.com/discuss/723854

讨论 这个问题前，先看下 Redis的版本中两个重要的节点：

1. Redisv4.0（引入多线程处理异步任务）
2. Redis 6.0（在网络模型中实现多线程 I/O ）

所以，网络上说的Redis是单线程，通常是指在Redis 6.0之前，其核心网络模型使用的是单线程。

且Redis6.0引入**多线程I/O**，只是用来**处理网络数据的读写和协议的解析**，而**执行命令依旧是单线程**。

> Redis在 v4.0 版本的时候就已经引入了的多线程来做一些异步操作，此举主要针对的是那些非常耗时的命令，通过将这些命令的执行进行异步化，避免阻塞单线程的事件循环。
>
> 在 Redisv4.0 之后增加了一些的非阻塞命令如 UNLINK、FLUSHALL ASYNC、FLUSHDB ASYNC。

**单线程速度快的原因**

(一)纯内存操作，避免大量访问数据库，减少直接读取磁盘数据，redis将数据储存在内存里面，读写数据的时候都不会受到硬盘 I/O 速度的限制，所以速度快；

(二)单线程操作，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗；

(三)采用了非阻塞I/O多路复用机制

**为什么单线程**

官方FAQ表示，因为Redis是基于内存的操作，CPU不是Redis的瓶颈，Redis的瓶颈最有可能是机器内存的大小或者网络带宽。既然单线程容易实现，而且CPU不会成为瓶颈，那就顺理成章地采用单线程的方案了（毕竟采用多线程会有很多麻烦！）。

# 12. [redis分布式锁](https://juejin.cn/post/6844903830442737671)

实现分布式锁有三种，1. 数据库锁 2. zookeeper分布式锁 3. redis分布式锁

**redis分布式锁实现**

**加锁**

1. **利用setnx+expire命令 (错误的做法)**

    非原子操作，容易出错，使得锁永远不过期，可以使用lua脚本来实现原子操作

    ```lua
    if redis.call('setnx',KEYS[1],ARGV[1]) == 1 then
        redis.call('expire',KEYS[1],ARGV[2]) 
        return 1 
    else 
        return 0 
    end
    ```

2. **使用set key value [EX seconds]\[PX milliseconds\][NX|XX]**

    ```sh
    SET key value[EX seconds][PX milliseconds][NX|XX]
    ```

    - EX seconds: 设定过期时间，单位为秒
    - PX milliseconds: 设定过期时间，单位为毫秒
    - NX: 仅当key不存在时设置值
    - XX: 仅当key存在时设置值

    其中value必须要是可唯一标识的，防止其他无关线程可以操作锁，或过期释放锁后原来的加锁线程可以操作锁。可以是UUID

**释放锁**

使用lua脚本

```lua
if redis.call("GET", KEYS[1]) == ARGV[1] then
	return redis.call("DEL", KEYS[1])
else
	return 0
end
```

同时要考虑redis集群问题。

golang中 redsync的实现

```go
func genValue() (string, error) {
	b := make([]byte, 16)
	_, err := rand.Read(b)
	if err != nil {
		return "", err
	}
	return base64.StdEncoding.EncodeToString(b), nil
}

// 加锁
// Lock locks m. In case it returns an error on failure, you may retry to acquire the lock by calling this method again.
func (m *Mutex) Lock() error {
    value, err := m.genValueFunc()	//调用genValue(),获得当前线程的唯一标识，防止被其他线程解锁
	if err != nil {
		return err
	}

    // 创建锁时指定加锁尝试次数
	for i := 0; i < m.tries; i++ {
		// 每次尝试的间隔
        if i != 0 {
			time.Sleep(m.delayFunc(i))
		}

		start := time.Now()

        // 异步尝试加锁,对每一个redis设置key
        // (aquire中的关键步骤) redis.String(conn.Do("SET", m.name, value, "NX", "PX", int(m.expiry/time.Millisecond)))，原子操作，并且已经存在的话则加锁失败
        // 这里n返回成功设置的redis池的个数
		n, err := m.actOnPoolsAsync(func(pool Pool) (bool, error) {
			return m.acquire(pool, value)
		})
		if n == 0 && err != nil {
			return err
		}

		now := time.Now()
		until := now.Add(m.expiry - now.Sub(start) - time.Duration(int64(float64(m.expiry)*m.factor)))
        // 这里quorum为连接池组中的一半数目+1,超过一半设置才算成功
        // quorum:       len(r.pools)/2 + 1,
		if n >= m.quorum && now.Before(until) {
			m.value = value
			m.until = until
			return nil
		}
        // 这里如果没有超过一半的连接池加锁，则释放掉已经操作过的连接池，判定失败
		m.actOnPoolsAsync(func(pool Pool) (bool, error) {
			return m.release(pool, value)
		})
	}

	return ErrFailed
}

// lua脚本会判定string中value的值是否为唯一标识的线程标识，如果一样的才允许删除,用Lua实现原子操作
var deleteScript = redis.NewScript(1, `
	if redis.call("GET", KEYS[1]) == ARGV[1] then
		return redis.call("DEL", KEYS[1])
	else
		return 0
	end
`)

// 释放锁，删除相关Key即可
func (m *Mutex) release(pool Pool, value string) (bool, error) {
	conn := pool.Get()
	defer conn.Close()
	status, err := redis.Int64(deleteScript.Do(conn, m.name, value))

	return err == nil && status != 0, err
}

// 解锁
// Unlock unlocks m and returns the status of unlock.
func (m *Mutex) Unlock() (bool, error) {
	n, err := m.actOnPoolsAsync(func(pool Pool) (bool, error) {
		return m.release(pool, m.value)
	})
	if n < m.quorum {
		return false, err
	}
	return true, nil
}
```

# 13. redis缓存穿透，缓存雪崩，缓存击穿

- **缓存穿透**：key对应的数据在数据源并不存在，每次针对此key的请求从缓存获取不到，请求都会到数据源，从而可能**压垮数据源**。比如用一个不存在的用户id获取用户信息，不论缓存还是数据库都没有，若黑客利用此漏洞进行攻击可能压垮数据库。（<font color=red>查询不存在的数据导致直接打到数据库</font>）
- **缓存击穿**：热点key对应的数据存在，但在redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候**大并发的重建缓存**可能会瞬间把后端DB压垮。（<font color=red>热点数据缓存失效导致直接打到数据库</font>）
- **缓存雪崩**：当缓存服务器重启或者大量缓存集中在某一个时间段失效（如宕机，key过期），这样在失效的时候，也会给后端系统(比如DB)带来很大压力。（<font color=red>缓存突然大面积失效</font>）

**<font size=7>如何解决</font>**

- **缓存穿透**:

1. 缓存空对象，这样就可以避免直接打到数据库

2. 使用**布隆过滤器**，把所有可能存在的key存储在布隆过滤器中，不存在的key查询会被过滤掉

- **缓存击穿**: 

1. 分布式互斥锁，只允许一个线程来重建这个热点缓存，之后恢复缓存获取数据
2. 设置热点数据永不过期

- **缓存雪崩**

1. 使用哨兵或者redis集群实现高可用
2. 采用多级缓存，本地进程作为一级缓存，redis作为二级缓存，不同级别的缓存设置的超时时间不同，即使某级缓存过期了，也有其他级别缓存兜底
3. 缓存过期时间设置随机值，减少大量key同一时间突然全部失效的概率

# 14. redis pipeline

# 15. redis事务

redis中事务是通过`MULTI`, `EXEC`, `WATCH`等命令来实现的。事务提供了一种将多个命令请求打包，然后一次性，按顺序地执行多个命令的机制，而且事务执行期间，服务器不会中断事务而改去执行其他客户端的命令请求，它会将事务中所有命令都执行完毕，然后才去处理其他客户端的命令请求。 

```shell

```

# 16. bgsave和save的比较

![https://pics2.baidu.com/feed/86d6277f9e2f07081c16c7ef407edb9fa801f2ab.jpeg?token=ea0c962fd6de8d119d2d6a262a8d974a](https://pics2.baidu.com/feed/86d6277f9e2f07081c16c7ef407edb9fa801f2ab.jpeg?token=ea0c962fd6de8d119d2d6a262a8d974a)
