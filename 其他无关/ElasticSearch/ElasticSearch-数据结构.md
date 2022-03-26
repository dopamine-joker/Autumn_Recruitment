# ElasticSearch 数据结构

> dopamine-joker整理归纳，原文http://dopaminer.xyz/2021/08/05/ElasticSearch-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/#more

Elasticsearch(简称ES)是基于Lucene库实现的开源搜索引擎，其是作为Elastic Stack的核心的分布式搜索和分析引擎，可对所有数据提供近实时的搜索和分析。

## 1. 索引

ES里的索引(index)和MYSQL的索引是不一样的，ES官方对索引的释义如下

> An Elasticsearch index is a collection of documents that are related to each other. Elasticsearch stores data as JSON documents.

即ES中索引是一个相互关联的文档(documents)的集合，会以JSON文档格式来存储数据。而文档(document)是存储在ES索引中的JSON对象，是最基本的存储单位。在ES中是索引通过`倒排索引(Inverted Index)`来实现的，这种结构可以用来快速地进行全文搜索。那有倒排索引，自然就有`正向索引(forward Index)`。首先看看什么是正向索引。

### (1) 正向索引(forward Index)

正向索引其实就是一种文档(documents)映射(map)到单词(term)的数据结构，通过正向索引可以直接通过文档然后获取其所包含的单词。

举个例子，假如现在有三个文档

> doc1: hello world
>
> doc2: hello golang
>
> doc2: ElasticSearch World

那么对其进行分词后就可以建立如下的正向索引

| Document | term                  |
| -------- | --------------------- |
| doc1     | hello, world          |
| doc2     | hello, golang         |
| doc3     | ElasticSearch , world |

### (2) 倒排索引(Inverted Index)

在正向索引中是从文档映射到单词，那么将这种映射关系反过来再稍作修改那就可以得到倒排索引。

倒排索引其实就是一种单词映射到文档的数据结构，我们可以通过某个单词来得到包含了这个单词的文档列表（可以是一个或多个文档）。

同样是上述三个文档，可以建立如下的倒排索引

| term          | freq(频率) | Document   |
| ------------- | ---------- | ---------- |
| hello         | 2          | doc1, doc2 |
| world         | 2          | doc1, doc3 |
| golang        | 1          | doc2       |
| ElasticSearch | 1          | doc3       |

> 注意在倒排索引中不会出现重复的term.

(1) **Term Dictionary**

将文档写入ES时，ES会对其进行分词，这些分出来的一个个分词（这里可能包含一些专有名词，它并不会被分词器分离，比如'Hong Kong'或者自定义的）汇总起来叫做`Term Dictionary`.

(2) **Posting Lists**

上面提到ES是基于Lucene库实现的，在ES中索引中的数据会被分成多个分片，而ES**分片(shared)**的底层是Lucene索引，即分片本质上是个**Lucene索引(Lucene Index)**。Lucene中索引文件会被拆分多个子文件，每个子文件是一个**段(segment)**，段都是独立可被搜索的数据集，里面存储着文档，倒排索引等信息，并且会定期合并成一个新的大段(段不变性导致合并，[具体见这](https://fdv.github.io/running-elasticsearch-fun-profit/003-about-lucene/003-about-lucene.html))。在每个段中最多可以存储$2^{31}$个文档，并且每个文档都用一个范围在 [0,段中文档数(最大$2^{31}-1$)] 的数字来唯一标识，即文档ID。

![Inside an Elasticsearch index](https://fdv.github.io/running-elasticsearch-fun-profit/003-about-lucene/images/image2.svg)

![img](https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/image/613455-20160229144719611-1983692652.png)

在倒排索引中每个词都对应着一个文档列表，而文档又有其对应的文档ID。每个分词对应的**文档ID列表**叫做`Posting Lists`.建立了倒排索引后，当需要查询某个单词的时候，就可以直接在**排序**后的term decitionary中**二分查找**，然后再从它的posting list找到想要的文档。这里在posting list中为了快速查找文档ID，posting list中的ID是有序存储的，并且lucene底层采用了**跳表（skiplist）**数据结构进行维护。在联合查询时，就可以遍历多个term的posting list然后利用跳表进行并集操作。

(3) **Term Index**

在ES中，若要提高检索速度，就需要把term dicionary搬入内存中，可是当词条数量很多时，将其全部搬进内存将会耗费太多空间。因此ES中对term dicionary建立`term index`词条索引放入内存中，term index会存储单词的**部分前缀**，利用term index可以快速定位到term dictionary中的对应磁盘block位置，然后再顺序查找。内存中存储term index的数据结构则是**FST(Finite State Transducers) 有限状态转换器**

![index-arch.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/20/171981d0b44a8c44~tplv-t2oaga2asx-image.image)

## 2. Finite State Transducers(FST)

FST是一种有限状态机，可以将term(字节序)映射到任意输出，即当它读取读取输入时可以产生对应的输出。它的结构如下:

![img](https://2.bp.blogspot.com/_4pUbN9gxhUI/TPk21wErb9I/AAAAAAAAAFM/dhPcsyo3KV4/s400/FSTExample.png)

在上图中的这个FST中，它将**排序**后的单词`mop`,`moth`,`pop`,`stat`,`stop`和`top`分别映射到序数0,1,2,3,4,5。当沿着途中箭头遍历时，就可以将路径上的数字进行求和，从而得到对应的序数。举例，当我们遍历stop时，在`s`箭头中得到3，在`o`箭头中得到1，而其他字母没有标识数字，即0。这样就可以得到stop的排序结果$3+0+1+0=4$​​​。

利用这种数据结构，就可以以O(len(key))的时间复杂度去查找一个term对应的值，实现term-value映射。上面提到，这种结构只保存了词典中每个词条的前缀。实际查找时是根据这个前缀快速定位到磁盘block，然后再在这个block查找posting list。

[[FST演示工具]](http://examples.mikemccandless.com/fst.py?terms=&cmd=Build+it%21)

## 3. Frame Of Reference（FOR）

posting list中的ID都是有序保存的，这种有序结构可以在ES查询时方便对posting lists进行交集并集查询（比如同时查询包含term1和term2的文档ID)，而这种有序结构正好可以使用**增量编码(delta-encoding)**来对postling lists进行压缩。增量压缩即是保存除了第一个数字之外其他数字减去前一个数字的结果。

比如现在有个posting lists是[73, 300, 302, 332, 343, 372]，那么其增量压缩后得到的列表为[73,227,2,30,11,29]。在这里所有增量的大小都在 0~255范围内，这就意味着可以我们只用1 byte来存储每个数字。

在ES中（也可说是在Lucene中），首先会将posting lists先增量编码，然后分成多个块(block)，每个块有256个文档ID。分别在每一个块中计算需要多少个位(bit)来存储里面的每一个数，这个数字由块中最大的数字所需要的位数决定，之后将这个数字保存到每个块的块头。最终就可以用这个头部的位数信息决定要采用多少个位来保存每一个增量数字。这种编码方法就是`Frame Of Reference`。

假设posting lists每个块只有3个文档ID，则过程如下

![img](https://api.contentstack.io/v2/assets/575e4d88d8edd48f7693888c/download?uid=blt42bcd2cef05d6634?uid=blt42bcd2cef05d6634)

当然这样只是为了能够在磁盘中用更少的空间来保存posting list,实际上使用时还是需要将其恢复成原样。

## 3. Roaring bitmaps

在ES中查询时经常需要对posting list进行交集并集，即对多个posting list进行交并集计算，同时为了优化查询，在ES中，可以使用过滤器（filter）来进行文档匹配，这种匹配不会影响文档查询的评分(query会)，并且查询结果可以缓存。

为了优化查询，在ES中，可以使用过滤器来进行文档匹配，这种匹配不会影响文档查询的评分(query会)。常用的过滤器查询会被缓存起来放入内存，即过滤器缓存(filter cache)，过滤器缓存会将（filter,segment）映射到其所匹配的**文档列表**，但每个列表可能包含很多个文档ID，若过滤器将文档ID列表（从FOR复原后）全部搬到内存中存储也是一笔不小的空间开销。由于过滤器是缓存在内存中并且是常用的，因此对过滤器缓存进行解压缩时的速度不能太慢，其压缩结构不与加压缩比较费时的FOR相同。

`Roaring bitmaps`结构能满足对缓存过滤器中的文档ID列表的快速解压缩，这是一种将integer array(整数数组)和bitmap结合起来的结构。首先看一下integer array和bitmap。

(1) **integer array**

顾名思义，就是一个保存文档ID的整型(int)数组。其好处就是可以直接通过下标访问，迭代十分方便。但是这种结构却需要用4个字节来保存每一个文档ID，这使得密集的posting list得耗费大量内存，若需要保存100M个文档，则其需要400MB的内存空间。

(2) **bitmap**

在位图中将会使用一个位(bit)来代表一个文档ID是否存在。想知道位图中是否包含某个文档ID，只需要找到该ID对应的位置，并判断这个位的值，0代表不存在，1代表存在。对比上述同样100M个文档，若这100M个文档ID都是递增的，我们就只需要100M个bit=12.5MB存储即可。

[0,1,2,...,1000000] => [1,1,1,1,.....,1,1,1,1,1]

但bitmap也不一定是最佳选择，比如当列表ID只有两个文档[1,1000000]时，使用bitmap存储大小还是12.5MB。而使用integer array只需要8B

(3) **roaring bitmaps**

`roaring bitmaps`则将`integer array`和`bitmap`结合起来。它会在posting list中的数据按照最高的16个位分块。例如，第一个块中文档ID包含[0-65535] (前16个位均为0),第二个块包含[65536-131071] (前15个位为0最后一个位为1).这样划分后在每一个块中只需要独立编码最低的16个位就足够了，并且块内存储的数字也只会在范围[0-65535]中。

如果这个块中的ID个数小于4096则使用integer array来存储，否则使用bitmap来存储。

![img](https://api.contentstack.io/v2/assets/575e4d8843e9adc5387164fa/download?uid=bltec702aef46d747d4?uid=bltec702aef46d747d4)

为什么是以4096为分界线，具体原因如下:

由于每个ID的前16个位已经被划分出去了，我们只需要关注最低的16个位，即存储每一个ID只需要两个字节。假设现在一个块内有n个文档ID，这样若使用integer array来保存，则空间大小2n B。则对于bitmap，无论块内有多少个文档ID其都是固定大小（65536个bit）：$ \frac{65536}{8}= 8192$B。因此当n等于4096时，二者所需要耗费的空间相等，小于4096则integer array占用空间小，大于4096则bitmap占用空间小。利用这种思路，`roaring bitmaps`可以用更少空间缓存过滤器。

![img](https://api.contentstack.io/v2/assets/575e4d8843e9adc538716505/download?uid=blt8932f5963a5a42fc?uid=blt8932f5963a5a42fc)



## 4. References

https://www.elastic.co/cn/blog/frame-of-reference-and-roaring-bitmaps

https://learnku.com/elasticsearch/t/46888

https://xiaoming.net.cn/2020/11/25/Elasticsearch%20%E5%80%92%E6%8E%92%E7%B4%A2%E5%BC%95/

https://juejin.cn/post/6844904133162434574

https://segmentfault.com/a/1190000037658997

https://fdv.github.io/running-elasticsearch-fun-profit/003-about-lucene/003-about-lucene.html

https://www.cnblogs.com/richaaaard/p/5226334.html

