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