# 1. return机制

return 并非原子操作，会分为赋值，返回两个步骤

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

# 6. [slice原理](https://studygolang.com/articles/7118)

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



![img](http://images2015.cnblogs.com/blog/496176/201605/496176-20160514133733937-1151272381.png)

![img](http://images2015.cnblogs.com/blog/496176/201605/496176-20160514133752702-1527122175.png)

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

# 7. map原理



# 8. golang只有传值没有传引用

类似传slice和map会发生变化是因为修改的是他们所指向的底层数组。本质上还是传值，只不过这个值和实参都是指向同一个底层数组。

