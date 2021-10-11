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
