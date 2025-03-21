
> <h4/>
- [**‌并发编程**](#并发编程)
	- [进程、线程与协程进程](#进程、线程与协程进程)
	- [goroutine](#goroutine)
		- [为普通函数创建goroutine](#为普通函数创建goroutine)
		- [匿名函数创建goroutine](#匿名函数创建goroutine)
	- [channel（通道）](#channel（通道）)
		- [通道的声明和创建](#通道的声明和创建)
		- [通道发送数据](#通道发送数据) 
		- [通道接收数据](#通道接收数据)
		- [单向通道](#单向通道) 
		- [带缓冲的通道](#带缓冲的通道)
	- [轻量级线程](#轻量级线程)
	- [线程锁](#线程锁)
		- [互斥锁](#互斥锁)
		- [读写互斥锁](#读写互斥锁)
		- [等待组](#等待组)
	- [sync包学习](#sync包学习)
		- [Once 执行一次](#Once执行一次)
	

<br/><br/><br/>

***
<br/>

> <h1 id="并发编程">并发编程</h1>

并发是指在同一时间内执行多个任务。并发编程包括多线程编程、多进程编程及分布式程序等。本章讲解并发编程中的多线程编程。Go语言支持并发的特性，并且通过goroutine完成。goroutine类似于线程，是由Go语言运行时(runtime)调度和管理的。Go程序能够将goroutine中的任务合理地分配给每个CPU。

![go.0.0.14.png](./../Pictures/go.0.0.14.png)

<br/>

Socket的收发包分配一个线程。开发人员需要在线程数量和CPU数量间建立一个对应关系，以保证每个任务能及时地被分配到CPU上进行处理，同时避免多个任务频繁地在线程间切换执行而损失效率。


<br/><br/><br/>
> <h2 id="进程、线程与协程进程">进程、线程与协程进程</h2>
进程、线程与协程进程是计算机系统进行资源分配和调度的基本单位，是操作系统结构的基础。

线程是CPU独立调度和分派的基本单位。它被包含在进程之中，是进程中的实际运作单位。一个进程中可以并发多个线程，每条线程并行执行不同的任务。

与线程类似，协程与协程之间相对独立，每个协程都有自己的上下文；由当前协程切换到其他协程的过程是由当前协程进行控制并实现的。

<br/>

- **并发与并行**

- 并行：同时执行；多个CPU同时执行多个线程。
- 并发：穿插执行；一个CPU在不同时间段执行不同的线程，也就是说多个线程轮流穿插执行。

为了方便理解，使用如图11.1所示的示意图展示并行与并发的区别。


![go.0.0.15.png](./../Pictures/go.0.0.15.png)

&emsp; 并发编程是指让一个CPU在某个时间段内执行一个含有多个线程的程序，这些线程被这个CPU轮流穿插执行。并发编程的优势在于当一个CPU执行含有多个线程的程序时，另一个线程不必等待当前线程被执行完毕后再被执行，进而提高使用CPU的效率。


<br/><br/><br/>
> <h2 id="goroutine">goroutine</h2>
在Go语言中，goroutine不仅是轻量级线程，而且是用户级线程。用户级线程是指由用户控制代码的执行流程，不需要操作系统进行调度和分派。也就是说，Go程序智能地将goroutine中的任务合理分配给每个CPU。

Go语言不仅有goroutine，还有用于调度goroutine的、对接系统级线程的调度器。调度器主要负责统筹调配Go语言并发编程模型（即GPM模型）中的3个主要元素，它们分别是G（goroutine的缩写）​、P（processor的缩写）和M（machine的缩写）​。

其中，M指的是系统级线程；P指的是能够使若干个G在恰当的时机与M对接，并得以运行的中介。

Go程序在启动时，从main包的main()函数开始，为main()函数创建一个默认的goroutine。下面讲解两种创建goroutine方法：一种是为普通函数创建goroutine；另一种是为匿名函数创建goroutine。

<br/>

&emsp; 虽然，线程池为逻辑编写者提供了线程分配的抽象机制。但是，如果面对随时随地可能发生的并发和线程处理需求，线程池就不是非常直观和方便了。能否有一种机制：使用者分配足够多的任务，系统能自动帮助使用者把任务分配到CPU上，让这些任务尽量并发运作。这种机制在Go语言中被称为goroutine。

&emsp; goroutine的概念类似于线程，但goroutine由Go程序运行时的调度和管理。Go程序会智能地将goroutine中的任务合理地分配给每个CPU。


<br/><br/><br/>
> <h2 id="为普通函数创建goroutine">为普通函数创建goroutine</h2>

```
go 函数名称(prameter)
```

参数说明如下。

**parameter：参数列表。**



<br/>

**‌演示当创建goroutine时，如何为被调用函数设置参数:**

```go
package concurrent_program_practice

import (
	"MLC_GO/TestNotes/GenPracticeExample/pkg/logging"
	"time"
)

func getOff(names []string) {
	for i, name := range names {
		logging.DebugInfo("第",i+1,"位乘客", name, "正在下车")
		// 延时一秒
		time.Sleep(1 * time.Second)
	}
}


// goroutine被调用函数设置参数
func BaseConcurrentProgram_v1 () {
	// 乘客姓名切片
	var offNames = [6]string{"David", "Levon", "Steven", "James", "Tom", "Jack"}
	// 执行并发程序
	go getOff(offNames[:])
	var onNames = [6]string{"张三", "李四", "万物", "糟熘", "周期", "礼拜"}

	for i, name := range onNames {
		logging.DebugInfo("第",i+1,"位乘客", name, "正在上车")
		time.Sleep(1 * time.Second)
	}
}
```


<br/><br/>
> <h3 id="匿名函数创建goroutine">匿名函数创建goroutine</h3>

使用go关键字还可以为匿名函数创建goroutine。

注意，go关键字的后面须包含两个内容：一个是定义的匿名函数；另一个是匿名函数的调用参数。使用go关键字为匿名函数创建goroutine的语法格式如下。

```go
go func(parameter) {
	func field
}(para)
```

- func: Go语言的关键字，用于定义匿名函数。
- parameter：参数列表。
- func field：匿名函数的实现代码。
- para：匿名函数被调用时所需设置的参数。


<br/><br/><br/>
> <h2 id="channel（通道）">channel（通道）</h2>
通道是Go语言在两个或多个goroutine之间的一种通信方式。通道可以让一个goroutine给另一个goroutine发送消息。当需要在goroutine之间共享一个数据资源时，通道是确保同步交换数据资源的方法。goroutine与通道的关系如图11.2所示。

![go.0.0.16.png](./../Pictures/go.0.0.16.png)

多个goroutine为了争抢数据资源，势必会降低执行效率。为了保证执行效率不降低，goroutine之间通过通道进行通信，确保同一时刻只有一个goroutine访问通道，并执行发送和接收数据的操作。

通道就像队列一样，遵循“先入先出”的规则，保证发送和接收数据的顺序。


<br/><br/><br/>
> <h2 id="通道的声明和创建">通道的声明和创建</h2>
**声明**

```go
var name chan type
```

- **参数说明如下。**
	- var: Go语言关键字，用于声明变量。
	- name：通道的名称。
	- chan: Go语言关键字，通道类型。
	- type：在通道内部传输的数据的类型。为了创建通道，需要使用make()函数。创建通道的语法格式如下。

<br/>

**为了创建通道，需要使用make()函数。创建通道的语法格式如下:**

```go
name := make(chan type)
```

- **参数说明如下**。
	- name：通道的名称。
	- make: make()函数，用于创建通道。
	- chan: Go语言关键字，通道类型。
	- type：在通道内部传输的数据的类型。在实际开发中，可以先声明通道，再创建通道。

```go
var chel chan string

chel = make(chan string)
```
<br/>

**或者**

```
1 := make(chan int)            // 创建一个整型类型的通道
 ch2 := make(chan interface{})  // 创建一个空接口类型的通道，可以存放任意格式
 type Equip struct{ /＊ 一些字段 ＊/ }
 ch2 := make(chan ＊Equip)         // 创建Equip指针类型的通道，可以存放＊Equip
```

<br/><br/><br/>
> <h2 id="通道发送数据">通道发送数据</h2>

通道的发送使用特殊的操作符“<-”，将数据通过通道发送的格式为：

- **`通道变量 <- 值`**
	- ● 通道变量：通过make创建好的通道实例。
	- ● 值：可以是变量、常量、表达式或者函数返回值等。值的类型必须与ch通道的元素类型一致。


```go
name <- value
```
- name：已经创建的通道的名称。
- value：值，可以是变量、常量、表达式或函数返回值等；值的类型必须与在通道内部传输的数据的类型保持一致。

<br/>

```go
// 创建一个空接口通道 ch := make(chan interface{})
 // 将0放入通道中 ch <-0
 // 将hello字符串放入通道中 ch <- "hello"
```

<br/>

**或者**

```go
//创建传输任意类型数据的通道
chel := make(chan interface{})
//使用通道发送数字711
chel <- 711
//使用通道发送字符串hello world
chel <- "hello"
```


<br/><br/><br/>
> <h2 id="通道接收数据">通道接收数据</h2>

- **使用通道接收和发送数据具有以下特性:**
	- 如果没有接收通道传输的数据，那么发送数据操作将被持续阻塞。
	- 如果接收了通道传输的数据，但尚未执行发送数据操作，那么接收数据操作将被持续阻塞。
	- 通道一次只能传输一个数据。

<br/>

使用通道接收数据也要使用特殊的操作符"**<-**",通道接收数据有4种写法：

**1).阻塞接受数据(发送数据操作将被持续阻塞，直到通道传输的数据被接收)：将接收变量作为“<-”操作符的左值**

```go
data := <-name
```

- data：变量名。
- name：通道的名称。

<br/>

**2). 非阻塞接收数据(通道传输的数据被接收且不会发送阻塞)**

```go
data, ok = <-name
```
- ok：表示通道传输的数据是否被接收。

&emsp; data：表示接收到的数据。未接收到数据时，data为通道类型的零值。

● ok：表示是否接收到数据。非阻塞的通道接收方法可能造成高的CPU占用，因此使用非常少。如果需要实现接收超时检测，可以配合select和计时器channel进行。

<br/>

**3). 发送数据操作被持续阻塞，直到接收通道传输的数据，但是接收到的数据会被忽略**

```
<-ch
```

执行该语句时将会发生阻塞，直到接收到数据，但接收到的数据会被忽略。这个方式实际上只是通过通道在goroutine间阻塞收发实现并发同步。

<br/>

**4).循环接收**

通道的数据接收可以借用for range语句进行多个元素的接收操作。

```
for data := range ch {

}
```


通道ch是可以进行遍历的，遍历的结果就是接收到的数据。数据类型就是通道的数据类型。通过for遍历获得的变量只有一个，即上面例子中的data。

<br/><br/><br/>
> <h2 id="单向通道">单向通道</h2>

声明格式:

只能发送的通道类型为chan<-，只能接收的通道类型为<-chan: 

```
// 只能发送通道
var 通道实例 chan <- 元素类型                      

// 只能接收通道
var 通道实例 <-chan 元素类型                      
```

● 元素类型：通道包含的元素类型。

● 通道实例：声明的通道变量。


```
// 声明一个只能发送的通道类型，并赋值为ch
ch := make(chan int)
var chSendOnly chan<- int = ch

//声明一个只能接收的通道类型，并赋值为ch
var chRecvOnly <-chan int = ch
```

<br/><br/><br/>
> <h2 id="带缓冲的通道">带缓冲的通道</h2>

&emsp; 在无缓冲通道的基础上，为通道增加一个有限大小的存储空间形成带缓冲通道。带缓冲通道在发送时无需等待接收方接收即可完成发送过程，并且不会发生阻塞，只有当存储空间满时才会发生阻塞。同理，如果缓冲通道中有数据，接收时将不会发生阻塞，直到通道中没有数据可读时，通道将会再度阻塞。

&emsp; **提示：** 无缓冲通道保证收发过程同步。无缓冲收发过程类似于快递员给你电话让你下楼取快递，整个递交快递的过程是同步发生的，你和快递员不见不散。但这样做快递员就必须等待所有人下楼完成操作后才能完成所有投递工作。如果快递员将快递放入快递柜中，并通知用户来取，快递员和用户就成了异步收发过程，效率可以有明显的提升。带缓冲的通道就是这样的一个“快递柜”。

<br/>

有缓冲的通道是指在接收数据前能够存储一个或多个值的通道。创建有缓冲的通道的语法格式如下:

- **`通道实例 := make(chan通道类型, 缓冲大小)`**
	- ● 通道类型：和无缓冲通道用法一致，影响通道发送和接收的数据类型。
	- ● 缓冲大小：决定通道最多可以保存的元素数量。
	- ● 通道实例：被创建出的通道实例。

```go
name := make(chan type, num)
```

- name：通道的名称。
- make: make()函数，用于创建通道。
- chan: Go语言关键字，通道类型。
- type：在通道内部传输的数据的类型。
- num：通道内部存储的数据的数量上限。

可以把有缓冲的通道看作是一个元素队列，使用有缓冲的通道执行发送数据的操作就是在元素队列的尾部插入元素，而接收数据的操作则是从元素队列的头部移除一个元素。

如果有缓冲的通道存储数据的数量达到上限，那么执行发送数据操作的goroutine将被持续阻塞，直到执行接收数据操作的goroutine接收有缓冲的通道传输的数据。

如果有缓冲的通道存储数据的数量为零，那么执行接收数据操作的goroutine将被持续阻塞，直到执行发送数据操作的goroutine使用有缓冲的通道发送数据。

在使用有缓冲的通道完成发送与接收数据的操作时，不强制要求执行发送数据操作的goroutine和执行接收数据操作的goroutine同时准备就绪。

<br/><br/><br/>
> <h2 id='轻量级线程'>轻量级线程</h2>

`runtime.GOMAXPROCS(逻辑CPU数量)`

逻辑CPU数量可以有如下几种数值：

● <1：不修改任何数值。

● =1：单核心执行。

● >1：多核并发执行。

`runtime.NumCPU()`： 查询CPU数量


<br/> <br/> <br/>
> <h2 id='线程锁'>线程锁</h1>
<br/>

- **静态检测**

```go
/**
 * @description: 序列号生成器
 */

var (
	// 序列号
	seq int64
)

func testLock1() {
	//生成10个并发序列号
	for i := 0; i < 10; i++ {
		go GenID()
	}
	fmt.Println(GenID())
}
func GenID() int64 {
	// 尝试原子的增加序列号
	// 使用原子操作函数atomic.Add Int64()对seq()函数加1操作。
	// 不过这里故意没有使用atomic.Add Int64()的返回值作为Gen ID()函数的返回值，因此会造成一个竞态问题
	atomic.AddInt64(&seq, 1)
	return seq
}

```

在运行程序时，为运行参数加入“-race”参数，开启运行时（runtime）对竞态问题的分析，命令如下：`go run -race main.go`

![go11](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/go11.png)

根据报错信息是`atomic.AddInt64(&seq, 1)`有竞态问题，根据atomic.AddInt64()的参数声明，这个函数会将修改后的值以返回值方式传出。修改后如下：

```go
func GenID() int64 {
	// 尝试原子的增加序列号
	return atomic.AddInt64(&seq, 1)
}

```

执行命令后输出：

```
go run -race main.go
<=============== 🍎 🍎 🍎 ===============> 

10


<=============== 🍑 🍑 🍑 ===============> %  
```

本例中只是对变量进行增减操作，虽然可以使用互斥锁（sync.Mutex）解决竞态问题，但是对性能消耗较大。

<br/><br/>
> <h3 id='互斥锁'>互斥锁</h3>
&emsp; **`（sync.Mutex）`** 保证同时只有一个goroutine可以访问共享资源互斥锁是一种常用的控制共享资源访问的方法。


<br/><br/>
> <h3 id='读写互斥锁'>读写互斥锁</h3>
&emsp; **`（sync.RWMutex）`**在读比写多的环境下比互斥锁更高效

&emsp;在读多写少的环境中，可以优先使用读写互斥锁，sync包中的RWMutex提供了读写互斥锁的封装。


<br/><br/>
> <h3 id='等待组'>等待组</h3>
&emsp; **（sync.Wait Group）** 保证在并发环境中完成指定数量的任务

&emsp; 除了可以使用通道（channel）和互斥锁进行两个并发程序间的同步外，还可以使用等待组进行多个任务的同步。

![go12](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/go12.jpeg)

&emsp; 等待组内部拥有一个计数器，计数器的值可以通过方法调用实现计数器的增加和减少。当我们添加了N个并发任务进行工作时，就将等待组的计数器值增加N。每个任务完成时，这个值减1。同时，在另外一个goroutine中等待这个等待组的计数器值为0时，表示所有任务已经完成。


<br/><br/><br/>

***
<br/>

> <h1 id="sync包学习">sync包学习</h1>
[](https://www.cnblogs.com/rickiyang/p/11074167.html)
<br/>

> <h1 id="Once 执行一次">Once 执行一次</h1>
Once 的作用是多次调用但只执行一次，Once 只有一个方法，Once.Do()，向 Do 传入一个函数，这个函数在第一次执行 Once.Do() 的时候会被调用，以后再执行 Once.Do() 将没有任何动作，即使传入了其它的函数，也不会被执行，如果要执行其它函数，需要重新创建一个 Once 对象。

```go
package main

import (
	"fmt"
	"sync"
)


func main() {
	var once sync.Once
	onceBody := func() {
		fmt.Println("我只会出现一次")
	}
	done := make(chan bool)
	for i := 0; i < 3; i++ {
		go func() {
			once.Do(onceBody)
			done <- true
		}()
	}
	for i := 0; i < 3; i++ {
		<-done
	}
}

```
