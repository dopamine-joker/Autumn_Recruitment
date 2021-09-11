# [golang调度器](https://medium.com/a-journey-with-go/go-goroutine-os-thread-and-cpu-management-2f5a5eaf518a#id_token=eyJhbGciOiJSUzI1NiIsImtpZCI6Ijc0MjE3YjhkYWRiYjM2NTc4MzU4MGY5ZTkyNDg3ZDcwMWNkMzhmZTYiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJuYmYiOjE2MzEzNTU1MzEsImF1ZCI6IjIxNjI5NjAzNTgzNC1rMWs2cWUwNjBzMnRwMmEyamFtNGxqZGNtczAwc3R0Zy5hcHBzLmdvb2dsZXVzZXJjb250ZW50LmNvbSIsInN1YiI6IjExNzYwMjU0MzcwMzcxNjU3OTQzOSIsImVtYWlsIjoiZG9wYW1pbmVyam9rZXJAZ21haWwuY29tIiwiZW1haWxfdmVyaWZpZWQiOnRydWUsImF6cCI6IjIxNjI5NjAzNTgzNC1rMWs2cWUwNjBzMnRwMmEyamFtNGxqZGNtczAwc3R0Zy5hcHBzLmdvb2dsZXVzZXJjb250ZW50LmNvbSIsIm5hbWUiOiJqb2tlciBkb3BhbWluZXIiLCJwaWN0dXJlIjoiaHR0cHM6Ly9saDMuZ29vZ2xldXNlcmNvbnRlbnQuY29tL2EvQUFUWEFKd2l6SHF6blVHNXNCdkRBd0JZemdEUTNFR0c3SmVrSWo0VnpKNXc9czk2LWMiLCJnaXZlbl9uYW1lIjoiam9rZXIiLCJmYW1pbHlfbmFtZSI6ImRvcGFtaW5lciIsImlhdCI6MTYzMTM1NTgzMSwiZXhwIjoxNjMxMzU5NDMxLCJqdGkiOiI1NDAyZjdhYzRiMGZlYjJiYmE0ZTk1NDM5NjIxMmY1NDcyNTNkYmU0In0.VH8Zu9jPB7-bxNL0SiLr57g_Utaeh5t4YlF3ITj9TEAzaeSr8h9PfAhqbMEmwdzAI1y-4vTvS6PJsd1cm7FzpzUHWmBXiKYW69AoWVDPhyR7bYFLW_fWb8rPZ9VnpZ118Fdc9PwbsJLaujtwlhEH-eHheNv4-F80ei4IKTmvQzBmtqgJFrN1T8cM2zxWLL5teAIOEkopg_feEPysnWUz4Cg6Fwh0XN6aIqu77_5JxHps6-nxdsyur01gIY7DGN7qHFlTfglvQU3bMrJh0cnK467V_54805MJjLBSpe4dd6ftWdrMYf5U9oh8QgvVcebV3cg17BhnUqfBgVYNiOeHgg)

https://www.jianshu.com/p/d01dd0d4cdd0



1. **GMP模型概念**

```
GMP调度模型的主要概念
G - goroutine (go协程)
M - worker, or machine (工作的内核线程)
P - processor, a resource that is required to execute Go code.
    M must have an associated P to execute Go code[...].
```

其中P是一个比较抽象的概念。并不是特指物理机上的CPU。可以理解为M执行时的上下文。

![image-20210911220833543](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210911220833543.png)

**每一个goroutine(go协程)都运行在一个内核线程上(M)，并且这个M是与一个P进行相关联的。**

2. **调度的例子**

    ```go
    func main() {
       var wg sync.WaitGroup
       wg.Add(2)
    
       go func() {
          println(`hello`)
          wg.Done()
       }()
    
       go func() {
          println(`world`)
          wg.Done()
       }()
    
       wg.Wait()
    }
    ```

    Go会首先根据机器的逻辑CPU个数(可自己设定)创建不同的`P`。并且把他们保存到一个空闲`P`列表中。

    ![image-20210911221444444](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210911221444444.png)

    然后，一个**新的goroutine**或者**准备运行的goroutines**会唤醒一个`P`以便来服务这个goroutine。然后这个`P`将会创建一个与内核线程相关联的`M`。

    ![image-20210911221833089](C:/Users/78620/AppData/Roaming/Typora/typora-user-images/image-20210911221833089.png)

    然而，像P那样，如果一个没有被使用的`M`(即**此时没有等待运行的goroutine**)从系统调用中返回，或甚至被垃圾收集器强制停止，则它会返回到空闲的`M`队列中。

    ![image-20210911222409893](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210911222409893.png)

    在程序引导时，Go就已经创建了一些与内核线程关联的`M`。在这个代码例子中，打印`hello`的线程将会使用main这个goroutine，然后打印`world`的线程将会从`M`和`P`的空闲列表中取出一个`M`和`P`。(这里之所以第一个是用的main线程是因为这里的main在`wg.wait()`处等待，即阻塞，则它对应P和M会被让出来供其他使用)

    ![image-20210911222758712](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210911222758712.png)

3. **系统调用**

    Go会在运行时通过包装来优化系统调用(无论是否阻塞)。这个包装将会自动将`P`和`M`分离出来，并且允许其他线程来运行这个`P`。

    文件读写的例子(没有网络轮询器):

    ```go
    func main() {
       buf := make([]byte, 0, 2)
    
       fd, _ := os.Open("number.txt")
       fd.Read(buf)
       fd.Close()
    
       println(string(buf)) // 42
    }
    ```

    工作流程:

    ![image-20210911223729392](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210911223729392.png)

    现在`p0`已经在空闲列表中并且是可被使用的。然后，一旦打开文件的系统调用退出，则Go将遵循下面规则执行只到其中一个被满足才退出

    - 尝试获取相同的P，这里是`p0`，然后继续执行
    - 尝试在空闲`P`列表中取得一个并恢复执行
    - 把goroutine放入全局队列(专门存放goroutine的队列，等待被取出执行)并把`M`放到空闲的`M`列表中。

    然而，在非阻塞I/O调用的情况下，Go也会处理资源尚未准备继续的情况。在这个例子中，第一个系统用调用(前一个工作流之后)并不会成功，因为资源尚未就绪，这将使得go使用network poller来停驻这些goroutine。

    > 老版本的Go，`network poller`(网络轮询器)是一个`goroutine`，负责事件通知，内部使用系统kqueue或者epoll。这个goroutine通过channel通知等待中的goroutine。通过channel唤醒等待中的goroutine，避免l了单个线程系统调用的开销。
    >
    > 当前版本的Go，`network poller`集成在运行中。由于运行时知道一个等待socket的goroutine何时准备好再次执行，因此可以更加及时地分配cpu资源，进而降低延时。

    > netpoll本质上是对 I/O 多路复用技术的封装，所以自然也是和epoll一样脱离不了下面几步
    >
    > - netpoll创建及其初始化；
    >
    > - 向netpoll中加入待监控的任务；
    >
    > - 从netpoll获取触发的事件；

    ```go
    func main() {
       http.Get(`https://httpstat.us/200`)
    }
    ```

    一旦第一次系统调用完成并明确表示资源尚未就绪，则goroutine会停驻直到network poller通知它资源已经准备好了。在这个例子中，线程`M`将不会被阻塞。

    ![image-20210911230642946](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210911230642946.png)

    当GO调度器等待时，goroutine将会再次执行。在成功获取goroutine的信息后，调度器将会询问network poller是否该goroutine在等待运行。

    ![image-20210911230921309](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210911230921309.png)

    如果多个goroutine已经准备就绪，则只有一个会被运行，多余的会加入到全局队列并等待调度执行。

    **[另一个解释版本](https://www.jianshu.com/p/d01dd0d4cdd0)**

    - **网络调度器**

    当运行的操作系统具有异步处理系统调用的能力时，可以通过使用称之为网络轮询器（network poller）的东西来更有效的处理系统调用。这是通过使用kqueue（MacOS），epoll（Linux）或iocp（Windows）在不同的操作系统上完成的。

    我们今天使用的许多操作系统都可以异步地处理基于网络的系统调用。这也是网络轮询器名称的来源，因为它的主要用途是处理网络操作。通过使用网络轮询器进行网络系统调用，调度器可以防止goroutines在进行这些系统调用时阻塞M。这能够让M执行P的LRQ中其他的goroutines，而不需要创建新的M。这有助于降低操作系统上的调度负载。

    **图3**

    ![img](https://upload-images.jianshu.io/upload_images/13462240-d6987f215a940944.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

    图3是基本的调度示图。G1正在M上被执行，LRQ中还有3个G等待获取他们在M上的时间。网络轮询器是空闲的，无事可做。

    **图4**

    ![img](https://upload-images.jianshu.io/upload_images/13462240-cf0b473dbe6c4799.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

    在图4中，G1希望进行网络调用，因此将G1移动到了网络轮询器，这里将会处理异步网络调用。M现在就可以执行来自LRQ上的另外一个G。在本例中，G2被切换到了M上。

    **图5**

    ![img](https://upload-images.jianshu.io/upload_images/13462240-721e925d17a3e3d6.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

    在图5中，网络调用器完成了异步网络调用，G1被移回了P的LRQ队列中。一旦G1可以在M上进行上下文切换，它对应的Go相关代码就可以再次执行。这里最大的优势是，执行网络系统调用不需要额外的M。网络轮询器只有一个OS线程，它正在处理一个有效的时间循环。

4. **调度示意图**

![image-20210912001049913](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210912001049913.png)

> LRQ: Local Run Queue
>
> GRQ: Global Run Queue

其中P的数量由变量<font color=red>`GOMAXPROCS`</font>环境变量或runtime中的<font color=red>GOMAXPROCS()</font>函数决定的。M的数量由runtime/debug包中的setMaxThreads()决定。如果当前的M阻塞，那就会新建一个新的线程。

M的数量与P没有关系。如果当前M阻塞了，则P的goroutine会运行到其他的M上，或者新建一个M。所以可能出现多个M，一个P的情况。

- **同步系统调用**

    当Goroutine想要进行不能异步完成的系统调用时会发生什么？在这种情况下，网络轮询器不可用，进行系统调用的Goroutine将会阻塞M。这很不幸，但无法阻止这种情况发生。不能进行异步系统调用的一个例子就是基于文件的系统调用。如果你正在使用CGO，那么调用C函数也会阻塞M也是其中一种情况。

    让我们继续看看同步系统调用（如文件I/O）导致M阻塞时会发生什么。

    **图6**

    ![img](https://upload-images.jianshu.io/upload_images/13462240-57b1bdaa2ce517e9.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

    图6再次展示了基本的调度示图。但是这次G1将会进行阻塞M的同步系统调用。

    **图7**

    ![img](https://upload-images.jianshu.io/upload_images/13462240-6c422a16217d2ce5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

    在图7中，调度器能够识别G1导致M阻塞。此时，调度器将绑定了G1的M1从P中分离出来。然后调度器引入一个新的M2来为P服务。此时就可以从LRQ中选择G2，然后切换到M2上。如果一个M因为之前的转换已经存在，那么这个转换比创建一个新的M要快。

    **图8**

    ![img](https://upload-images.jianshu.io/upload_images/13462240-a9ef302c20c6f6f5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

    在图8中，G1进行的阻塞系统调用完成了。此时G1能够移回LRQ中并且被P继续服务。M1被放在一旁，如果将来这种情况再次发生，M1可以继续使用。

5. **操作系统线程问题**

    使用系统调用时，Go不会限制可以阻止的OS线程数

6. **go func(){}**

![image-20210911225050563](https://gitee.com/dopamine-joker/image-host/raw/master/image/image-20210911225050563.png)

这里当M阻塞时，会将M从P解除，把G运行到其他空闲的M或创建新的M(再来一个M)

M恢复后，就会尝试获取一个空闲的P。如果没有P空闲，则M会休眠，G放到全局队列。