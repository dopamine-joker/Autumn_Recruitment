# 1.空间配置器

## 1. new和delete

new中包含两个操作:(1) 调用`::operator new`配置内存 (2) 调用构造函数

delete: (1) 调用析构函数 (2) 调用`operator::delete`释放内存

stl将这两个阶段的步骤分开

内存分配和释放: alloc::allocate(), alloc::deallocate()

对象构造和析构: ::construct() ,::destory()

## 2. construct()和destory()

stl中的构造和析构

![image-20210908165031991](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908165031991.png)

## 3. std::alloc

stl中的内存分配和释放

二级配置器，理论上超过128b使用第一级配置器，否则第二级配置器。

实际由于采用哪个配置器是由`__USE_MALLOC`决定，并且SCI STL没有定义它。

所以默认都是第二级。

![image-20210908165557291](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908165557291.png)

# 2. Traits编程技法

http://dopaminer.xyz/2021/02/28/C++traits/

**迭代器的类型**

- Input Iterator: 只读 （opterator++）
- Output Iterator: 只写 （opterator++）
- Forward Iterator: 允许读写 （opterator++）
- Bidrectional Iterator: 可双向移动 (++ --)
- Random Access Iterator: 涵盖所有指针计算能力(p+n, p-n, p[n], p1-p2, p1<p2) (++ --)

# 3. [vector](http://www.cplusplus.com/reference/vector/vector/)

**线性连续空间**

1. **vector的迭代器是普通指针**

2. vector迭代器失效，因容器不足重新分配，导致原先迭代器指向的位置被回收

    引起内存空间重分配的情况:

    - reserve(重新分配空间capacity)
    - insert
    - resize(重新分配size)

3. 空间扩充原则

    原大小为0，则配置1，原大小不为0，则配置为原大小的两倍

4. 比较

    vector重载了`==` ,`!=`, `<`, `>`, `<=`, `>=`

```c++
// API
// 修改capacity或size
void resize (size_type n);	//change size
void resize (size_type n, const value_type& val);	//change size
size_type capacity();	//get capacity
void reserve (size_type n);	//change capacity
void shrink_to_fit();	//Requests the container to reduce its capacity to fit its size.

//allocator
allocator_type get_allocator() const noexcept;	//Get allocator
```

# 4. list

链表，SCI List是一个双向环状链表，**list在缺省构造函数中会初始化一个空节点，并让其前后指针都指向自己，这个节点作为end()来用**

- 没有capacity
- vector重载了`==` ,`!=`, `<`, `>`, `<=`, `>=`

![image-20210908193705837](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908193705837.png)

```c++
// modifiers api
template <class... Args> //(c++11)
void emplace_front (Args&&... args); //Construct and insert element at beginning
void push_back (value_type&& val);	//Add element at the end 
void pop_back();	//Delete last element

// Operations
void slice(...);	//把一个list移到另外一个list中，注意是移不是复制
void remove(const value_type& val);		//Remove elements with specific value
template <class Predicate>
void remove_if (Predicate pred);	//Remove elements fulfilling condition,移除满足条件的元素
void unique();		//移除相同的连续元素直至连续相同的元素只剩下一个
void merge (list&（&&） x);	// Merge sorted lists
void sort(Compare comp);
void reverse();		//反转
```

![image-20210908194756466](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908194756466.png)

# 5. deque

deque是双向开口的连续线程空间

![image-20210908195113373](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908195113373.png)

![image-20210908200526272](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908200526272.png)

![image-20210908200424303](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210908200424303.png)

```C++
// api
// 同vecotr类似，多了添加到头和尾的方法
void push_front (value_type&& val);
template <class... Args>
void emplace_front (Args&&... args);
```

1. **扩容策略**
    - 如果空间够，全部后挪或者前挪，不重新申请空间
    - 若前后空间不够，则重新申请map

# 6. queue

- 构造时可以指定容器，如`vector<int>`，` list<int>`等

```c++
// api
void push(value_type&& val);
template <class... Args> void emplace (Args&&... args);
void pop();
```

# 7. priority_queue

# 8. set

- **set**的**insert**会返回一个`pair<iterator,bool>`代表是否插入成功，成功返回结果迭代器指向插入位置，bool=true,失败返回指向已有元素的迭代器，bool=false。
- lower_bound和upper_bound的区别是前者返回**大于等于val**的，后者返回**大于val**的

```c++
// api
pair<iterator,bool> insert (value_type&& val);
iterator insert (const_iterator position, value_type&& val);
template <class InputIterator>
void insert (InputIterator first, InputIterator last);
void insert (initializer_list<value_type> il);

iterator find (const value_type& val);	//Get iterator to element
iterator lower_bound (const value_type& val);	//Return iterator to lower bound	返回等于或大于val的元素的迭代器
iterator upper_bound (const value_type& val);	//Return iterator to upper bound， 返回指向第一个比val大的元素的迭代器
pair<iterator,iterator> equal_range (const value_type& val);	//返回一个范围，这个范围内所有值等于val,但是由于set元素的独一性，因此这个范围只包含了一个值
```

# 9. multiset

**lookup**

find() count() lower_bound() upper_bound() equal_range()

可以重复元素，同set

# 10. unordered_set

**lookup api**

```c++
//lookup
iterator find ( const key_type& k );	//Get iterator to element
size_type count ( const key_type& k ) const;	//Count elements with a specific key
pair<iterator,iterator>
   equal_range ( const key_type& k );	//Get range of elements with a specific key
```



```c++
// api
// buckets
size_type bucket_count() const noexcept;	//返回hash表当前有多少个桶
size_type max_bucket_count() const noexcept;	//返回hash最多可以有多少个桶
size_type bucket_size ( size_type n ) const;	//返回i号桶当前有多少个元素
size_type bucket ( const key_type& k ) const;	//返回元素当前所在桶的序号
// hash policy
float load_factor() const noexcept;	//返回哈希表的负载因子
float max_load_factor() const noexcept;	//返回当前哈希表的最大负载因子
void max_load_factor ( float z );	//设置z为当前哈希表最大负载因子
void rehash ( size_type n );	//rehash,设置哈希表桶的数量大于等于n
//
hasher hash_function() const;	//返回hash所使用的hash函数
```

hash_function用法

```c++
// unordered_set::hash_function
#include <iostream>
#include <string>
#include <unordered_set>

typedef std::unordered_set<std::string> stringset;

int main ()
{
  stringset myset;

  stringset::hasher fn = myset.hash_function();

  std::cout << "that: " << fn ("that") << std::endl;
  std::cout << "than: " << fn ("than") << std::endl;

  return 0;
}

/*
that: 15843861542616104093
than: 18313131606624605886
*/
```

# 11. map

**lookup**

find() count() lower_bound() upper_bound() equal_range()

```c++
// api
mapped_type& at (const key_type& k);	// 返回Key对用的value值的引用，注意这里是左值

// Operations
iterator find (const key_type& k);	//搜索，搜不到返回end()
iterator lower_bound (const key_type& k);	//大于等于
iterator upper_bound (const key_type& k);	//大于
pair<iterator,iterator> equal_range (const key_type& k);
```

# 12. multi_map

可以重复元素，同map

**lookup**

find() count() lower_bound() upper_bound() equal_range()

# 13. unordered_map

**lookup**

```c++
iterator find ( const key_type& k );	//Get iterator to element
size_type count ( const key_type& k ) const;	//Count elements with a specific key
pair<iterator,iterator>
   equal_range ( const key_type& k );	//Get range of elements with specific key
```













