# 1. return机制

return **并非原子操作**，会分为赋值，返回两个步骤

```go
func get() int {
    num := 1
    return num
}
```

上面return时会生成一个临时变量tmp,把num复制到tmp后，返回tmp

因此下面写法不会影响返回值

1. 多个defer的执行顺序为“后进先出”；
2. defer、return、返回值三者的执行逻辑应该是：return最先执行，return负责将结果写入要返回的临时变量中；接着defer开始执行一些收尾工作；最后函数携带当前返回值退出。

注意defer 语句是在返回前执行

```go
func get() int {
    num := 1
    defer func() {
        num++
    }()
    return num
}
// 步骤
// tmp = num
// num++
// return tmp
```

而下面写法会影响

```go
func get() (num int) {
    num = 1
    defer func() {
        num++
    }()
    return
}
// 步骤
// num = 1
// num++
// return
```

# 2. defer

```go
func deferTest() {
  var a = 1
  defer fmt.Println(a)
  
  a = 2
  return
}
// 1
```

这里输出1,因此参数在defer语句出现的时候就已经确定了

但是这样写就不一样了

```go
func deferTest() {
	var a = 1
	defer func() {
		fmt.Println(a)
	}()

	a = 2
	return
}
//2
```

注意多defer的顺序是后进先出的

# 3. goto

goto 不能跳转到其他函数或者内层代码

# 4. 单引号，双引号，反引号

go语言中不倾向使用单引号表示字符串，单引号用于表示Golang的一个特殊类型：**rune**，类似其他语言的byte但又不完全一样，是指：码点字面量（Unicode code point），不做任何转义的原始内容

双引号用来创建可解析的字符串字面量(支持转义，但不能用来引用多行)

反引号用来创建原生的字符串字面量，这些字符串可能由多行组成(不支持任何转义序列)，原生的字符串字面量多用于书写多行消息、HTML以及正则表达式

# 5. 字符大小

ASCII字符: 1字节

中文字符: 3字节

emoji: 4字节

# 6. array原理

数组占用的空间是连续的，由于内存连续，CPU能把正在使用的数据缓存得更久。**和slice不一样，这里确确实实就是一块连续空间，而不是共享数组的机制，因此函数传值是会对数组所有元素进行拷贝的，并且修改了形参不会影响到实参**

![image-20211009044544180](https://gitee.com/dopamine-joker/image-host/raw/master/202110090445324.png)

初始化方法

```go
// len(array) cap(array)输出一样
var array [5]int64{10, 20, 30, 40, 50}	//10,20,30,40,50
var array [5]int64{10, 20, 30}			//10,20,30,0,0
var array [...]int64{10,20,30}			//10,20,30
array := [5]int64{2:10, 1:20}			//0,20,10,0,0
var array [4][2]int
array := [2][2]int{{10, 11}, {12,13}}
array := [2][2]int{0:{10, 11}, 1:{12,13}}
array := [2][2]int{0:{0:10, 1:11}, 1:{0:12,1:13}}
```

```go
array := [5]*int{0: new(int), 1: new(int)}
*array[0] = 10
*array[1] = 20
```

![image-20211009045540555](https://gitee.com/dopamine-joker/image-host/raw/master/202110090455701.png)

- 数组变量的类型包括**<font color=red>数组长度</font>**和**<font color=red>每个元素的类型</font>**。只有两部分都相同的数组，才是类型相同的数组，才能相互赋值。

- 如果把一个指针数组赋值给另外一个，则两个指针指向同一个底层数组同样这两个指针类型要一样才可以

    ![image-20211009045843036](https://gitee.com/dopamine-joker/image-host/raw/master/202110090458252.png)

# 7. [slice原理](https://studygolang.com/articles/7118)

slice的数据结构其实就是一个指针，这个指针指向底层的一个共享数组

它包括三个部分

- 指向数组的指针ptr
- slice的长度len
- slice的容量cap

```go
type slice struct {
    array unsafe.Pointer
    len   int
    cap   int
}
```

![image-20211009050830589](https://gitee.com/dopamine-joker/image-host/raw/master/202110090508754.png)

![img](http://images2015.cnblogs.com/blog/496176/201605/496176-20160514133733937-1151272381.png)

![img](http://images2015.cnblogs.com/blog/496176/201605/496176-20160514133752702-1527122175.png)

**nil切片**

![image-20211009051005557](https://gitee.com/dopamine-joker/image-host/raw/master/202110090510730.png)

<font color=red>**注意nil切片也可以使用append**</font>

**空切片**

![image-20211009051211557](https://gitee.com/dopamine-joker/image-host/raw/master/202110090512753.png)

**<font color=red>nil切片和空切片的区别</font>**

```go
//slice底层结构
type SliceHeader struct {
    Data uintptr  //指向的引用数组地址
    Len  int     //切片长度
    Cap  int     //切片容量
}

var nilSlice []int64
nullSlice := make([]int64, 0)
fmt.Println(len(nilSlice), cap(nilSlice))
fmt.Println(len(nullSlice), cap(nullSlice))
fmt.Printf("%p\n", &nilSlice)
fmt.Printf("%p\n", &nullSlice)
fmt.Printf("%+v\n", *(*reflect.SliceHeader)(unsafe.Pointer(&nilSlice)))
fmt.Printf("%+v\n", *(*reflect.SliceHeader)(unsafe.Pointer(&nullSlice)))
```

```go
// output
0 0
0 0
0xc00000c030
0xc00000c048
{Data:0 Len:0 Cap:0}
{Data:5594960 Len:0 Cap:0}
```

可以看到nil切片底层中的data数据域为0

而空切片的data数据域有被分配，指向一块地址。



**<font color=red>切片赋值</font>**

类似下面这种操作，这是把老slice切块，然后新的slice把指针域指向这块切块的第一个元素，并修改对应的cap和size。**两个slice共享一个底层数组**。这时候修改newSlice会影响到slice

```go
slice := []int64{10,20,30,40,50}
newSlice := slice[1:3]
```

![image-20211009053431369](https://gitee.com/dopamine-joker/image-host/raw/master/202110090534580.png)

**!!!!这种情况对于string也同理**

```go
// string底层结构
type StringHeader struct {
	Data uintptr
	Len  int
}

str1 := "123"
str2 := str1
str3 := str1[1:2]
fmt.Printf("%+v\n", *(*reflect.StringHeader)(unsafe.Pointer(&str1)))
fmt.Printf("%+v\n", *(*reflect.StringHeader)(unsafe.Pointer(&str2)))
fmt.Printf("%+v\n", *(*reflect.StringHeader)(unsafe.Pointer(&str3)))
```

```go
///output
{Data:4806600 Len:3}
{Data:4806600 Len:3}
{Data:4806601 Len:1}
```

**切片第三个索引选项**

[a : b : c]

a: 从哪个下标开始（包含）

b: 从哪个下标结束（不包含）

c: 容量到达原切片哪个位置

```go
slice := make([]int64, 10, 15)
for i := 0; i < len(slice); i++ {
    slice[i] = int64(i)
}
slice2 := slice[3:10:15]
// slice2 := slice[3:10:17]		//报错，因为第三个项比原切片容量要大
fmt.Println(len(slice2), "\t", cap(slice2))
fmt.Println(slice2)
```

```go
///output
7        12
[3 4 5 6 7 8 9]
```

**多唯切片**

```go
slice := [][]int{{10}, {100, 200}}
```

![image-20211009181046394](https://gitee.com/dopamine-joker/image-host/raw/master/202110091810464.png)

append时 slice容量的变化规律

> 实际go在append的时候放大cap是有规律的。在 *cap* 小于1024的情况下是每次扩大到 *2 \* cap* ，当大于1024之后就每次扩大到 *1.25 \* cap* 。所以上面的测试中cap变化是 1, 2, 4, 8

**参数为slice的情况**

首先要明确**golang中都是传值，不是引用传递**，因此形参和形参确实不一样，只是他们指向的底层数组是同一个地方，因此在方法中修改slice,实际上是修改了指针所指向的数组，因此用实参访问时会被修改。

但问题出现了，由于slice存在扩容机制(扩容后会在新地址操作数据，老地址就不管了)，因此我们还要在方法代码中判断slice是否扩容。若在方法中使用了append导致slice扩容，则此时方法的操作不会影响到实参。如果方法中没有使slice扩容，则修改会影响到原来的实参。

```go
func appendSLice(fSLice []int64) {
    fSLice[0] = 100						//这里无论传s还是s1,都还没扩容，因此操作在原来的底层数组，会影响到实参
    fSLice = append(fSLice, 10)			//若实参是s,fSLice在这里不扩容，因为此时size<cap, 若实参是s1,fSLice在这里不扩容在这里要发生扩容
    fSLice[1] = 200						//若实参是s,fSLice不扩容，此时fSlice指向旧空间，操作在原来的底层数组，会影响到实参s,
    									//若实参是s1,fSLice扩容，此时fSLice指向新空间，操作在新的底层数组，不会影响到实参s1,
    									//因此可以知道，在发生扩容之前，所有对slice的操作都会影响到原数组
	fmt.Printf("len(fSLice)=%d, cap(fSLice)=%d\n", len(fSLice), cap(fSLice))
}

func main() {
    s := make([]int64, 2, 3)
    s1 := make([]int64, 2, 2)
    s[0] = 0
    s[1] = 1
    s1[0] = 0
    s1[1] = 1
    appendSLice(s)
    appendSLice(s1)
    fmt.Println("=====")
	fmt.Printf("s的内容: %v\n", s)
	fmt.Printf("s1的内容: %v\n", s1)		//这里不会出现10的原因是因为s和s1中的len属性都是2,因此访问不到第三个数字，但实际上该位置已被修改
	fmt.Printf("len(s)=%d, cap(s)=%d\n", len(s), cap(s))
	fmt.Printf("len(s1)=%d, cap(s1)=%d\n", len(s1), cap(s1))
}
```



```shell
> go run hello.go
len(fSLice)=3, cap(fSLice)=3		# 这里是传s
len(fSLice)=3, cap(fSLice)=4		# 这里是传s1
=====
s的内容: [100 200]
s1的内容: [100 1]
len(s)=2, cap(s)=3
len(s1)=2, cap(s1)=2
```

# 8. map原理

# 9. golang只有传值没有传引用

类似传slice和map会发生变化是因为修改的是他们所指向的底层数组。本质上还是传值，只不过这个值和实参都是指向同一个底层数组。

同理channel也是因为底层指针指向了一个环形队列数组。所以函数传递也一样

# 10. 空导入

使用空导入可以让里面的init()被发现并执行，完成一些初始化。

如sql中通过空导入来注册数据库驱动

```go
import (
    "database/sql"
    _ "github.com/ziutek/mymysql/godrv"
)
```

eg2

postgres.go

```go
package postgres

import (
	"database/sql"
	"database/sql/driver"
	"errors"
)

// PostgresDriver provides our implementation for the
// sql package.
type PostgresDriver struct{}

// Open provides a connection to the database.
func (dr PostgresDriver) Open(string) (driver.Conn, error) {
	return nil, errors.New("Unimplemented")
}

var d *PostgresDriver

// init is called prior to main.
// 这里初始化，注册到sql中
func init() {
	d = new(PostgresDriver)
	sql.Register("postgres", d)
}
```

main.go

```go
// Sample program to show how to show you how to briefly work
// with the sql package.
package main

import (
	"database/sql"

	_ "github.com/goinaction/code/chapter3/dbdriver/postgres"
)

// main is the entry point for the application.
func main() {
	sql.Open("postgres", "mydb")
}
```

# 11. 顺序打印

```go
package main

import (
	"fmt"
	"sync"
)

var printTime = 100
var dogCh chan struct{}
var catCh chan struct{}
var birdCh chan struct{}
var wg sync.WaitGroup

func dog() {
	for i := 0; i < printTime; i++ {
		<-dogCh
		fmt.Println("dog!")
		catCh <- struct{}{}
	}
	wg.Done()
}

func cat() {
	for i := 0; i < printTime; i++ {
		<-catCh
		fmt.Println("cat!")
		birdCh <- struct{}{}
	}
	wg.Done()
}

func bird() {
	for i := 0; i < printTime; i++ {
		<-birdCh
		fmt.Println("bird!")
		dogCh <- struct{}{}
	}
	wg.Done()
}

func main() {
	dogCh = make(chan struct{}, 1)
	catCh = make(chan struct{}, 1)
	birdCh = make(chan struct{}, 1)
    defer func() {
		close(dogCh)
		close(catCh)
		close(birdCh)
	}()
	wg.Add(3)
	go dog()
	go cat()
	go bird()
	dogCh <- struct{}{}			//这里触发
	wg.Wait()
	fmt.Println("finish")
}
```

# 12. make 和 new

make用于内建类型(map, slice和channel)的内存分配

而new则用于各种类型的内存分配，注意，map,slice和channel都可以new。不过new出来后他们只是一个指针，还需要进行初始化

# 13. for+select, break无法跳出for循环

# 14. 两个goroutine交替打印数字

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	letter := make(chan struct{}, 1)
	number := make(chan struct{}, 1)
	stop := make(chan struct{})
    defer func() {
		close(letter)
		close(number)
		close(stop)
	}()
	wg := sync.WaitGroup{}
	wg.Add(2)

	go func(wg *sync.WaitGroup) {
		defer wg.Done()
		for ch := 'a'; ch < 'z'; ch = ch + 2 {
			<-letter
			fmt.Printf("%c%c", ch, ch+1)
			number <- struct{}{}
		}
		stop <- struct{}{}
	}(&wg)

	go func(wg *sync.WaitGroup) {
		defer wg.Done()
		num := 1
		for {
			select {
			case <-stop:
				return
			default:
				<-number
				fmt.Printf("%d%d", num, num+1)
				num = num + 2
				letter <- struct{}{}
			}
		}
	}(&wg)
	number <- struct{}{}
	wg.Wait()
}
```

# 15. panic 有无recover都不会阻止defer

```go
func defer_call() {
	defer func() {
		if err := recover(); err != nil {
			fmt.Printf("%s,\n", err)
		}
	}()
	defer func() {
		fmt.Println("111")
	}()
	defer func() {
		fmt.Println("222")
	}()
	defer func() {
		fmt.Println("333")
	}()
	panic("异常")
}

func main() {
    defer_call()
}
///output
333
222
111
异常
```

# 16. map非线程安全

https://cloud.tencent.com/developer/article/1539049

sycn.Map

1. 空间换时间。通过冗余的两个数据结构(read、dirty),实现加锁对性能的影响。
2. 使用只读数据(read)，避免读写冲突。
3. 动态调整，miss次数多了之后，将dirty数据提升为read。
4. double-checking。
5. 延迟删除。删除一个键值只是打标记，只有在提升dirty的时候才清理删除的数据。
6. 优先从read读取、更新、删除，因为对read的读取不需要锁。

```go
type Map struct {
	// 当涉及到dirty数据的操作的时候，需要使用这个锁
	mu Mutex
	// 一个只读的数据结构，因为只读，所以不会有读写冲突。
	// 所以从这个数据中读取总是安全的。
	// 实际上，实际也会更新这个数据的entries,如果entry是未删除的(unexpunged), 并不需要加锁。如果entry已经被删除了，需要加锁，以便更新dirty数据。
	read atomic.Value // readOnly
	// dirty数据包含当前的map包含的entries,它包含最新的entries(包括read中未删除的数据,虽有冗余，但是提升dirty字段为read的时候非常快，不用一个一个的复制，而是直接将这个数据结构作为read字段的一部分),有些数据还可能没有移动到read字段中。
	// 对于dirty的操作需要加锁，因为对它的操作可能会有读写竞争。
	// 当dirty为空的时候， 比如初始化或者刚提升完，下一次的写操作会复制read字段中未删除的数据到这个数据中。
	dirty map[interface{}]*entry
	// 当从Map中读取entry的时候，如果read中不包含这个entry,会尝试从dirty中读取，这个时候会将misses加一，
	// 当misses累积到 dirty的长度的时候， 就会将dirty提升为read,避免从dirty中miss太多次。因为操作dirty需要加锁。
	misses int
}
```

# 17. 数组可以比较，切片不行

# 18. waitgroup wait()后不能再add()了

# 19. GOMAXPROCS修改的是P，不是M

# 20. 切片，映射，通道，接口和函数类型都是引用类型

# 21. interface{}接口内存布局

<font color=red>**interface{} 16字节**</font>

内存模型

![image-20211012234757134](https://gitee.com/dopamine-joker/image-host/raw/master/202110122347210.png)

方法集规则

![image-20211012235009967](https://gitee.com/dopamine-joker/image-host/raw/master/202110122350021.png)

# 22. 嵌套地址相等验证

```go
package main

import (
	"fmt"
	"reflect"
)

type Person interface {
	call()
}

type man struct {
	name string
}

type son struct {
	man
}

func (s *man) call() {
	fmt.Println(s.name)
}

func main() {
	s := &son{
		man{
			name: "name",
		},
	}
	fmt.Printf("%v\n", reflect.ValueOf(s.call).Pointer())
	fmt.Printf("%v\n", reflect.ValueOf(s.man.call).Pointer())
	fmt.Printf("%v\n", &s.name)
	fmt.Printf("%v\n", &s.man.name)
}
```

# 23. GOMAXPROCS(1)

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
)

func main() {
	runtime.GOMAXPROCS(1)
	var wg sync.WaitGroup
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			fmt.Println("A:", i)
		}()
	}

	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func(i int) {
			defer wg.Done()
			fmt.Println("B:", i)
		}(i)
	}

	fmt.Println("main end loop")
	wg.Wait()
	fmt.Println("main exit")
}
```

```go
///output
main end loop
B: 9
A: 10
A: 10
A: 10
A: 10
A: 10
A: 10
A: 10
A: 10
A: 10
A: 10
B: 0
B: 1
B: 2
B: 3
B: 4
B: 5
B: 6
B: 7
B: 8
main exit
```

为什么是先输出了9,然后其他按顺序输出?

https://blog.csdn.net/zhangdell/article/details/116979998

可以看到“---main end loop---”总是最先输出，表明在1个操作系统线程的情况下，只有main协程执行到wg.Wait()阻塞等待时，其子协程才能被执行，而子协程的执行顺序正好对应于它们入队列的顺序。

其中Go1.13.8和Go1.14.6，在实现上和早期版本有一点不同，每增加一个子协程就把其对应的函数地址存放到”\_p\_.runnext“，而把”\_p\_.runnext“原来的地址（即上一个子协程对应的函数地址）移动到队列”\_p\_.runq“里面，这样当执行到wg.Wait()时，”\_p\_.runnext“存放的就是最后一个子协程对应的函数地址（即输出B: ９的那个子协程）。

当开始执行子协程对应的函数时，首先执行”\_p\_.runnext“对应的函数，然后按先进先出的顺序执行队列”\_p\_.runq“里的函数。所以这就解释了为什么总是B：9打在第一个，而后面打印的则是进入队列的顺序。

这里一次性把一个goroutine的内容全部都输出了这是因为执行时间过短，还来不及调度就已经执行完了。

相关源码：$GOROOT/src/runtime/proc.go

**个人理解**

runnext一般来说存的是当前G(这里是main)准备的下一个可以运行的G，如果当前运行的G的时间片还有时间，那么runnext的这个G就会继承当前的G继续运行，这样可以减少先把runnext的G放入队列中，然后再去runq拿出来运行的时延。但如果此时有新的G创建，则这个在runnext的老的G只能将其移动到runq队列中，然后新的G放到runq中。由于上述代码中main运行到wait中阻塞了，此时把P让出来，这时候由于打印9的这个G刚好在runnext中，直接拿出来运行，随后才去一个个运行runq中的G。

# 24. sync.pool

https://www.jianshu.com/p/8fbbf6c012b2

https://www.cnblogs.com/sunsky303/p/9706210.html

```go
// A Pool is a set of temporary objects that may be individually saved and
// retrieved.
//
// Any item stored in the Pool may be removed automatically at any time without
// notification. If the Pool holds the only reference when this happens, the
// item might be deallocated.
//
// A Pool is safe for use by multiple goroutines simultaneously.
//
// Pool's purpose is to cache allocated but unused items for later reuse,
// relieving pressure on the garbage collector. That is, it makes it easy to
// build efficient, thread-safe free lists. However, it is not suitable for all
// free lists.
//
// An appropriate use of a Pool is to manage a group of temporary items
// silently shared among and potentially reused by concurrent independent
// clients of a package. Pool provides a way to amortize allocation overhead
// across many clients.
//
// An example of good use of a Pool is in the fmt package, which maintains a
// dynamically-sized store of temporary output buffers. The store scales under
// load (when many goroutines are actively printing) and shrinks when
// quiescent.
//
// On the other hand, a free list maintained as part of a short-lived object is
// not a suitable use for a Pool, since the overhead does not amortize well in
// that scenario. It is more efficient to have such objects implement their own
// free list.
//
// A Pool must not be copied after first use.
type Pool struct {
	noCopy noCopy

	local     unsafe.Pointer // local fixed-size per-P pool, actual type is [P]poolLocal
	localSize uintptr        // size of the local array

	victim     unsafe.Pointer // local from previous cycle
	victimSize uintptr        // size of victims array

	// New optionally specifies a function to generate
	// a value when Get would otherwise return nil.
	// It may not be changed concurrently with calls to Get.
	New func() interface{}
}

// Local per-P Pool appendix.
type poolLocalInternal struct {
	private interface{} // Can be used only by the respective P.
	shared  poolChain   // Local P can pushHead/popHead; any P can popTail.
}

type poolLocal struct {
	poolLocalInternal

	// Prevents false sharing on widespread platforms with
	// 128 mod (cache line size) = 0 .
	pad [128 - unsafe.Sizeof(poolLocalInternal{})%128]byte
}
```

为了使得在多个goroutine中高效的使用goroutine，sync.Pool为每个P(对应CPU)都分配一个本地池，当执行Get或者Put操作的时候，会先将goroutine和某个P的子池关联，再对该子池进行操作。  **每个P的子池分为私有对象和共享列表对象**，私有对象只能被特定的P访问，共享列表对象可以被任何P访问。因为同一时刻一个P只能执行一个goroutine，所以无需加锁，但是对共享列表对象进行操作时，因为可能有多个goroutine同时操作，所以需要加锁。

值得注意的是poolLocal结构体中有个pad成员，目的是为了防止false sharing。cache使用中常见的一个问题是false sharing。当不同的线程同时读写同一cache line上不同数据时就可能发生false sharing。false  sharing会导致多核处理器上严重的系统性能下降。具体的可以参考[伪共享(False Sharing)](http://ifeve.com/falsesharing/)。

- local这里面真正的是[P]poolLocal其中P就是GPM模型中的P，有多少个P数组就有多大，也就是每个P维护了一个本地的poolLocal。
- poolLocal里面维护了一个private一个shared，看名字其实就很明显了，private是给自己用的，而shared的是一个队列，可以给别人用的。注释写的也很清楚，自己可以从队列的头部存然后从头部取，而别的P可以从尾部取。
- victim这个从字面上面也可以知道，幸存者嘛，**当进行gc的时候，会将local中的对象移到victim中去，也就是说幸存了一次gc**

![img](https://upload-images.jianshu.io/upload_images/10213518-c5f7694be2ded30a.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1132)

# 24. sync.Map

https://www.cnblogs.com/jiujuan/p/13365901.html

https://studygolang.com/articles/10345

```go
type Map struct {
	// 当涉及到dirty数据的操作的时候，需要使用这个锁
	mu Mutex
	// 一个只读的数据结构，因为只读，所以不会有读写冲突。
	// 所以从这个数据中读取总是安全的。
	// 实际上，实际也会更新这个数据的entries,如果entry是未删除的(unexpunged), 并不需要加锁。如果entry已经被删除了，需要加锁，以便更新dirty数据。
	read atomic.Value // readOnly
	// dirty数据包含当前的map包含的entries,它包含最新的entries(包括read中未删除的数据,虽有冗余，但是提升dirty字段为read的时候非常快，不用一个一个的复制，而是直接将这个数据结构作为read字段的一部分),有些数据还可能没有移动到read字段中。
	// 对于dirty的操作哦需要加锁，因为对它的操作可能会有读写竞争。
	// 当dirty为空的时候， 比如初始化或者刚提升完，下一次的写操作会复制read字段中未删除的数据到这个数据中。
	dirty map[interface{}]*entry
	// 当从Map中读取entry的时候，如果read中不包含这个entry,会尝试从dirty中读取，这个时候会将misses加一，
	// 当misses累积到 dirty的长度的时候， 就会将dirty提升为read,避免从dirty中miss太多次。因为操作dirty需要加锁。
	misses int
}
```

```go
type entry struct {
	p unsafe.Pointer // *interface{}
}
```

p有三种值：

- nil: entry已被删除了，并且m.dirty为nil
- expunged: entry已被删除了，并且m.dirty不为nil，而且这个entry不存在于m.dirty中
- 其它： entry是一个正常的值

# 25. golang容器

1. heap
2. list
3. ring

- **heap**

    heap通过一组api对已经实现了heap接口的结构进行调整

    ```go
    package main
    
    import (
    	"container/heap"
    	"fmt"
    )
    
    type Heap []int
    
    func (h *Heap) Len() int {
    	return len(*h)
    }
    
    func (h *Heap) Less(i, j int) bool {
    	return (*h)[i] < (*h)[j]
    }
    
    func (h *Heap) Swap(i, j int) {
    	(*h)[i], (*h)[j] = (*h)[j], (*h)[i]
    }
    
    func (h *Heap) Push(x interface{}) {
    	*h = append(*h, x.(int))
    }
    
    func (h *Heap) Pop() interface{} {
    	x := (*h)[h.Len()-1]
    	*h = (*h)[:h.Len()-1]
    	return x
    }
    
    func main() {
    	h := &Heap{}
    	h.Push(2)
    	h.Push(4)
    	h.Push(6)
    	h.Push(9)
    	h.Push(1)
    	h.Push(0)
    	heap.Init(h)				//api初始化heap
    	fmt.Printf("%v\n", *h)
    	heap.Push(h, 100)			//heap放入元素
    	fmt.Printf("%v\n", *h)		//自动建堆
    	(*h)[2]= -1					//手动修改元素后使用fix效率更高
    	heap.Fix(h, 2)				//fix修复堆
    	fmt.Printf("%v\n", *h)
    	num := heap.Pop(h)			//弹出元素
    	fmt.Println(num)	
    	fmt.Printf("%v\n", *h)
    }
    ```

    ```go
    ///output
    [0 1 2 9 4 6]
    [0 1 2 9 4 6 100]
    [-1 1 0 9 4 6 100]
    -1
    [0 1 6 9 4 100]
    ```

    - **list**

        普通双向列表，并且每一个元素都是一个Interface类型

        ```go
        // Element is an element of a linked list.
        type Element struct {
        	// Next and previous pointers in the doubly-linked list of elements.
        	// To simplify the implementation, internally a list l is implemented
        	// as a ring, such that &l.root is both the next element of the last
        	// list element (l.Back()) and the previous element of the first list
        	// element (l.Front()).
        	next, prev *Element
        
        	// The list to which this element belongs.
        	list *List
        
        	// The value stored with this element.
        	Value interface{}
        }
        
        // List represents a doubly linked list.
        // The zero value for List is an empty list ready to use.
        type List struct {
        	root Element // sentinel list element, only &root, root.prev, and root.next are used
        	len  int     // current list length excluding (this) sentinel element
        }
        ```

    - ring(环)

        循环的双向链表

        ![image-20211013165911240](https://gitee.com/dopamine-joker/image-host/raw/master/202110131659301.png)

        

        ```go
        package main
        
        import (
        	"container/ring"
        	"fmt"
        )
        
        func printf(x interface{}) {
        	switch x.(type) {
        	case int:
        		fmt.Println(x.(int))
        	}
        }
        
        func main() {
        	r1 := ring.New(3)
        	for i := 0; i < 3; i++ {
        		r1.Value = i
        		r1 = r1.Next()
        	}
        	r1.Do(printf)
        	fmt.Println("=====")
        	r2 := ring.New(3)
        	for i := 0; i < 3; i++ {
        		r2.Value = i * 2
        		r2 = r2.Next()
        	}
        	r2.Do(printf)
        	fmt.Println("=====")
        	r := r1.Link(r2)			//link先循环绕一圈，然后再将r2指向的节点连接，再循环绕一圈
        	r.Do(printf)
        }
        ```

        ```output
        2
        0
        1
        =====
        0
        2
        4
        =====
        0
        1
        2
        0
        2
        4
        ```

    # 26. context

    ```go
    func WithValue(parent Context, key, val interface{}) Context			//创建子context,增加新的value
    func WithCancel(parent Context) (ctx Context, cancel CancelFunc)		//创建子context,并且可以通过cancel取消
    func WithDeadline(parent Context, d time.Time) (Context, CancelFunc)	//创建子context,伴随过期日期
    func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)	//创建子context,伴随过期时间	
    ```

    

    
