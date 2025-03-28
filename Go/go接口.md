- [**接口**](#接口)
	- [空接口`interface{} `](#空接口interface{})
	- [接口的实现](#接口的实现)
	- [便于扩展的日志输出系统](#便于扩展的日志输出系统)
	- [接口和类型间转换](#接口和类型间转换)
	- [空接口类型（interface{}）](#空接口类型（interface{}）)
	- [类型分支](#类型分支)
	- [实现有限状态机（FSM）](#实现有限状态机（FSM）)



<br/><br/><br/>

***
<br/>

> <h1 id="接口">接口</h1>
&emsp; Go语言不是面向对象的编程语言（这是因为在Go语言中没有类和继承）​，但是它提供了接口。不同于面向对象的编程语言，**Go语言提供的接口是隐式接口**。那么，应该如何理解“隐式接口”呢？在Go语言程序开发中，**无须标明具体实现哪些接口，只要实现某个接口中的所有函数就说明这个接口被实现了。**

> 接口声明格式

```go
type 接口类型名 interface {
	方法1( 参数列表1 ) 返回值列表1
	方法2( 参数列表2 ) 返回值列表2

	 . . . . . 
}

```

● 接口类型名：使用type将接口定义为自定义的类型名。Go语言的接口在命名时，一般会在单词后面添加er，如有写操作的接口叫Writer，有字符串功能的接口叫Stringer，有关闭功能的接口叫Closer等。

● 方法名：**当方法名首字母是大写时，且这个接口类型名首字母也是大写时，这个方法可以被接口所在的包（package）之外的代码访问**。

● 参数列表、返回值列表：参数列表和返回值列表中的参数变量名可以被忽略，如：

```go
type writer interface {
	writer([]byte) (n int, err error)
}

// 上述代码还可以省略参数列表和返回值列表中的参数变量名
type writer interface {
	writer([]byte) (int, error)
}
```


<br/><br/><br/>
><h2 id="空接口interface{}">空接口interface{}</h2>
interface{} 是 Go 中的一个空接口类型。它是 Go 中最通用的类型，可以代表任何类型。

- **特点：**
	- 空接口：interface{} 被称为空接口，它不包含任何方法。因此，任何类型的值都可以赋给空接口。
	- 你可以使用空接口来表示一个未知类型或支持多种类型的数据。

**示例：**

```go
var a interface{}  // 空接口变量
a = 42             // 赋值为整数
fmt.Println(a)      // 输出: 42

a = "hello"         // 赋值为字符串
fmt.Println(a)      // 输出: hello
```
空接口的作用非常广泛，尤其是当你不知道确切类型时，或者需要存储多种不同类型的数据时。


<br/><br/>
> <h2 id='接口的实现'>接口的实现</h2>
规则1: 接口的方法与实现接口的类型方法格式一致

&emsp; 在类型中添加与接口签名一致的方法就可以实现该方法。**签名包括方法中的名称、参数列表、返回参数列表**。也就是说，只要实现接口类型中的方法的名称、参数列表、返回参数列表中的任意一项与接口要实现的方法不一致，那么接口的这个方法就不会被实现。 

```go
//定义一个数据写入器
type DataWriter interface {
	// interface{}类型的data，返回一个error结构表示可能发生的错误
	WriteData(data interface{}) error
}

// 定义文件结构，用于实现DataWriter
type file struct{}

// 实现DataWriter接口的WriteData()方法
func (d *file) WriteData(data interface{}) error {
	// 模拟写入数据
	fmt.Println("Write Data:", data)
	return nil
}

/** 接口的方法与实现接口的类型方法格式一致
 * @description:
 */
func testInterface1() {
	// 实例化file赋值给f，f的类型为*file
	f := new(file)
	// 声明一个DataWriter的接口
	var writer DataWriter
	// 将接口赋值f，也就是＊file类型
	writer = f
	// 使用DataWriter接口进行数据写入
	writer.WriteData("data")
}


testInterface1()
```

打印：

```
<=============== 🍎 🍎 🍎 ===============> 

Write Data: data


<=============== 🍑 🍑 🍑 ===============> 
```


<br/>

> **接口中所有方法均被实现**


<br/><br/>
> <h2 id='便于扩展的日志输出系统'>便于扩展的日志输出系统</h2>
**日志对外接口**

```go
type LogWriter interface {
	Write(data interface{}) error
}

// 日志器
type Logger struct {
	// 这个日志器用到的日志写入器
	writerList []LogWriter
}

// 注册一个日志写入器
func (l *Logger) RegisterWriter(writer LogWriter) {
	l.writerList = append(l.writerList, writer)
}

// 将一个data类型的数据写入日志
func (l *Logger) Log(data interface{}) {
	// 遍历所有注册的写入器
	for _, writer := range l.writerList {
		// 将日志输出到每一个写入器中
		writer.Write(data)
	}
}

// 创建日志器的实例￼
func NewLogger() *Logger {
	return &Logger{}
}

```


<br/>

> **文件写入器**

```
/**
 * @description: 文件写入器
 * 文件写入器的功能是根据一个文件名创建日志文件（file Writer的Set File方法）。
 * 在有日志写入时，将日志写入文件中。
 */
// 声明文件写入器，在结构体中保存一个文件句柄，以方便每次写入时操作
type fileWriter struct {
	file *os.File
}

// 设置文件写入器写入的文件名
func (f *fileWriter) SetFile(filename string) (err error) {
	// 如果文件已经打开，关闭前一个文件
	// 考虑到SetFile()方法可以被多次调用（函数可重入性）
	// 假设之前已经调用过Set File()后再次调用，此时的f.file不为空，就需要关闭之前的文件，重新创建新的文件。
	if f.file != nil {
		f.file.Close()
	}
	// 创建一个文件并保存文件句柄
	f.file, err = os.Create(filename)
	// 如果创建的过程出现错误，则返回错误
	return err
}

// 实现LogWriter的Write()方法
func (f *fileWriter) Write(data interface{}) error {
	// 如果文件没有准备好，文件句柄为nil
	// 此时使用errors包的New()函数返回一个错误对象，包含一个字符串“file not created”
	if f.file == nil {
		// 日志文件没有准备好
		return errors.New("file not created")
	}
	// 将数据序列化为字符串
	// 使用fmt.Sprintf将data转换为字符串，这里使用的格式化参数是“%v”，意思是将data按其本来的值转换为字符串
	str := fmt.Sprintf("%v\n", data)
	// 通过f.file的Write()方法，将str字符串转换为[]byte字节数组，再写入到文件中。如果发生错误，则返回
	_, err := f.file.Write([]byte(str))
	return err
}

// 创建文件写入器实例
func newFileWriter() *fileWriter {
	return &fileWriter{}
}

```


&emsp; 在操作文件时，会出现文件无法创建、无法写入等错误。开发中尽量不要忽略这些底层报出的错误，应该处理可能发生的所有错误。

&emsp; 文件使用完后，要注意使用os.File的Close()方法进行及时关闭，否则文件再次访问时会因为其属性出现无法读取、无法写入等错误。

&emsp; 提示：一个完备的文件写入器会提供多种写入文件的模式，例子中使用的模式是将日志添加到日志文件的尾部。随着文件越来越大，文件的访问效率和查看便利性也会大大降低。此时，就需要另外一种写入模式：滚动写入文件。

- 滚动写入文件模式也是将日志添加到文件的尾部，但当文件达到设定的期望大小时，会自动开启一个新的文件继续写入文件，最终将获得多个日志文件。

- 日志文件名不仅可以按照文件大小进行分割，还可以按照日期范围进行分割。在到达设定的日期范围，如每天、每小时的周期范围时，日志器会自动创建新的日志文件。这种日志文件创建方法也能方便开发者按日志查看日志。



> 命令行写入器

&emsp; 在UNIX的思想中，一切皆文件。文件包括内存、磁盘、网络和命令行等。这种抽象方法方便我们访问这些看不见摸不着的虚拟资源:
- 命令行在Go中也是一种文件;
- os.Stdout对应标准输出，一般表示屏幕，也就是命令行，也可以被重定向为打印机或者磁盘文件；
- os.Stderr对应标准错误输出，一般将错误输出到日志中，不过大多数情况，os.Stdout会与os.Stderr合并输出；
- os.Stdin对应标准输入，一般表示键盘。
- os.Stdout、os.Stderr、os.Stdin都是*os.File类型，和文件一样实现了io.Writer接口的Write()方法。


```
// 命令行写入器
type consoleWriter struct{}

// 实现LogWriter的Write()方法
func (f *consoleWriter) Write(data interface{}) error {
	// 将数据序列化为字符串
	str := fmt.Sprintf("%v\n", data)
	// 将数据以字节数组写入命令行中
	_, err := os.Stdout.Write([]byte(str))
	return err
}

// 创建命令行写入器实例
func newConsoleWriter() *consoleWriter {
	return &consoleWriter{}
}
```

&emsp; 除了命令行写入器（console Writer）和文件写入器（file Writer），读者还可以自行使用net包中的Socket封装实现网络写入器socket Writer，让日志可以写入远程的服务器中或者可以跨进程进行日志保存和分析。


<br/>

> 使用日志

```go
// 创建日志器
func testInterface2() {
	// 创建日志器
	l := NewLogger()
	// 创建命令行写入器
	cw := newConsoleWriter()
	// 注册命令行写入器到日志器中
	l.RegisterWriter(cw)
	// 创建文件写入器
	fw := newFileWriter()
	// 设置文件名
	if err := fw.SetFile("log.log"); err != nil {
		fmt.Println(err)
	}
	// 注册文件写入器到日志器中
	l.RegisterWriter(fw)

	// 写一个日志
	l.Log("hello")
}
```

打印：

```
<=============== 🍎 🍎 🍎 ===============> 

hello


<=============== 🍑 🍑 🍑 ===============> 
```

&emsp; 同时还有一个log.log文件生成，在`l.RegisterWriter(cw)`中，其意思是：使用日志器方法RegisterWriter()将一个日志写入器（LogWriter）注册到日志器（Logger）中。注册的意思就是将日志写入器的接口添加到write List中

<br/><br/>

> <h2 id='接口和类型间转换'>接口和类型间转换</h2>

> 类型断言的格式

```go
t, ok := i.(T)
```

&emsp; 如果发生接口未实现时，将会把ok置为false，t置为T类型的0值。正常实现时，ok为true。这里ok可以被认为是：i接口是否实现T类型的结果

<br/>

> **将接口转换为其他接口**

实现某个接口的类型同时实现了另外一个接口，此时可以在两个接口间转换。

**`interface{}类型表示空接口，意思就是这种接口可以保存为任意类型。`**


```go
// 定义飞行动物接口
type Flyer interface {
	Fly()
}

// 定义行走动物接口
type Walker interface {
	Walk()
}

// 定义鸟类
type bird struct{}

// 实现飞行动物接口
func (b *bird) Fly() {
	fmt.Println("bird: fly")
}

// 为鸟添加Walk()方法，实现行走动物接口
func (b *bird) Walk() {
	fmt.Println("bird: walk")
}

// 定义猪
type pig struct{}

// 为猪添加Walk()方法，实现行走动物接口
func (p *pig) Walk() {
	fmt.Println("pig: walk")
}
func testInterface3() {
	// 创建动物的名字到实例的映射
	animals := map[string]interface{}{
		"bird": new(bird),
		"pig":  new(pig),
	}
	// 遍历映射
	for name, obj := range animals {
		// 使用类型断言获得f，类型为Flyer及is Flyer的断言成功的判定
		f, isFlyer := obj.(Flyer)
		// 判断对象是否为行走动物
		w, isWalker := obj.(Walker)
		fmt.Printf("name: %s is Flyer: %v is Walker: %v\n", name, isFlyer, isWalker)
		// 如果是飞行动物则调用飞行动物接口
		if isFlyer {
			f.Fly()
		}
		// 如果是行走动物则调用行走动物接口
		if isWalker {
			w.Walk()
		}
	}
}
```

打印：

```
<=============== 🍎 🍎 🍎 ===============> 

name: bird is Flyer: true is Walker: true
bird: fly
bird: walk
name: pig is Flyer: false is Walker: true
pig: walk

<=============== 🍑 🍑 🍑 ===============> 
```



<br/>

> 将接口转换为其他类型

```
func testInterface4() {
	p1 := new(pig)

	// 由于pig实现了Walker接口，因此可以被隐式转换为Walker接口类型保存于a中
	var a Walker = p1
	// 由于a中保存的本来就是*pig本体，因此可以转换为*pig类型
	p2 := a.(*pig)

	fmt.Printf("p1=%p p2=%p", p1, p2)
}
```


打印：

```
<=============== 🍎 🍎 🍎 ===============> 

p1=0x100602af8 p2=0x100602af8

<=============== 🍑 🍑 🍑 ===============> 
```

&emsp; 接口断言类似于流程控制中的if。但大量类型断言出现时，应使用更为高效的类型分支switch特性。


<br/><br/>

> <h2 id='空接口类型（interface{}）'>空接口类型（interface{}）</h2>

&emsp； interface{} 是一个空的 interface 类型，根据前文的定义：一个类型如果实现了一个 interface 的所有方法就说该类型实现了这个 interface，空的 interface 没有方法，所以可以认为所有的类型都实现了 interface{}。如果定义一个函数参数是 interface{} 类型，这个函数应该可以接受任何类型作为它的参数。

```dart
func doAnything(i interface{}){
    
}
```

&emsp； 提示：空接口类型类似于C#或Java语言中的Object、C语言中的void*、C++中的std::any。

- 在泛型和模板出现前，空接口是一种非常灵活的数据抽象保存和使用的方法。
- 空接口的内部实现保存了对象的类型和指针。使用空接口保存一个数据的过程会比直接用数据对应类型的变量保存稍慢。因此在开发中，应在需要的地方使用空接口，而不是在所有地方使用空接口。

<br/>

空接口赋值如下：

```
// 声明any为interface{}类型的变量
var any interface{}

any = 1
// 打印any的值，提供给fmt.Println的类型依然是interface{}
fmt.Println(any)

any = "hello"
fmt.Println(any)
```

输出：

```
1
hello
```



<br/>

> 空接口获取值

保存到空接口的值，如果直接取出指定类型的值时，会发生编译错误，代码如下：

```
// 声明a变量，类型int，初始值为1 var a int = 1 
// 声明i变量，类型为interface{}，初始值为a，此时i的值变为1 var i interface{} = a 
// 声明b变量，尝试赋值i var b int = i

```

编译报错：

```
cannot use i (type interface {}) as type int in assignment: need type assertion
```

编译器告诉我们，不能将i变量视为int类型赋值给b。

&emsp; 将a的值赋值给i时，虽然i在赋值完成后的内部值为int，但i还是一个interface{}类型的变量。类似于无论集装箱装的是茶叶还是烟草，集装箱依然是金属做的，不会因为所装物的类型改变而改变。


编译器提示我们得使用`type assertion`，意思就是类型断言。
使用类型断言修改如下：

```go var b int = i.(int)
```


<br/><br/>
> <h2 id='类型分支'>类型分支</h2>


> **类型断言的书写格式**

```go
switch 接口变量.(type) {
	
		case类型1:
			
			// 变量是类型1时的处理
			
		case类型2:
			
			// 变量是类型2时的处理
			
			…
		default:
			// 变量不是所有case中列举的类型时的处理
	}
```

● 接口变量：表示需要判断的接口类型的变量。

● 类型1、类型2……：表示接口变量可能具有的类型列表，满足时，会指定case对应的分支进行处理。

<br/>

```
func printType(v interface{}) {
	switch v.(type) {
	case int:
		fmt.Println(v, "is int")
	case string:
		fmt.Println(v, "is string")
	case bool:
		fmt.Println(v, "is bool")
	}
}
func testInterface5() {
	printType(1024)
	printType("pig")
	printType(true)
}


testInterface5()
```

输出：

```
<=============== 🍎 🍎 🍎 ===============> 

1024 is int
pig is string
true is bool


<=============== 🍑 🍑 🍑 ===============> 
```


v.(type)就是类型分支的典型写法。

通过这个写法，在switch的每个case中写的将是各种类型分支。代码经过switch时，会判断v这个interface{}的具体类型从而进行类型分支跳转。

switch的default也是可以使用的，功能和其他的switch一致

<br/>
<br/>

> <h2 id='实现有限状态机（FSM）'>实现有限状态机（FSM）</h2>

&emsp; 有限状态机（Finite-State Machine，FSM），表示有限个状态及在这些状态间的转移和动作等行为的数学模型。

**状态的概念**
- 状态机中的状态与状态间能够自由转换。但是现实当中的状态却不一定能够自由转换，例如：人可以从站立状态转移到卧倒状态，却不能从卧倒状态直接转移到跑步状态，需要先经过站立状态后再转移到跑步状态。
- 每个状态可以设置它可以转移到的状态。一些状态机还允许在同一个状态间互相转换，这也需要根据实际情况进行配置。


<br/>

**自定义状态需要实现的接口**

&emsp; 有限状态机系统需要制定一个状态需具备的属性和功能，由于状态需要由用户自定义，为了统一管理状态，就需要使用接口定义状态。状态机从状态接口查询到用户的自定义状态应该具备的属性有：

● 名称，对应State接口的Name()方法。

● 状态是否允许在同状态间转移，对应State接口的Enable Same Transit()方法。

● 能否从当前状态转移到指定的某一个状态，对应State接口的Can Transit To()方法。


&emsp; 除此之外，状态在转移时会发生的事件可以由状态机通过状态接口的方法通知用户自己的状态，对应的是两个方法OnBegin()和OnEnd()，分别代表状态转移前和状态转移后。


<br/>

> **状态接口**

```
// 此接口用于状态管理器内部保存和外部实现
type State interface {
	// 获取状态名字
	Name() string
	// 该状态是否允许同状态转移
	EnableSameTransit() bool
	
	/*
	* 需要实现状态的事件，分别是“状态开始”和“状态结束”。
	* 当一个状态转移到另外一个状态时，当前状态的OnEnd()方法会被调用，而目标状态的OnBegin()方法也将被调用。
	*/
	// 响应状态开始时
	OnBegin()
	// 响应状态结束时
	OnEnd()
	// 判断能否转移到某个状态
	CanTransitTo(name string) bool
}

// 从状态实例获取状态名
func StateName(s State) string {
	if s == nil {
		return "none"
	}
	// 使用反射获取状态的名称
	return reflect.TypeOf(s).Elem().Name()
}
```



<br/>

> **状态基本信息**

&emsp; State接口中定义的方法，在用户自定义时都是重复的，为了避免重复地编写很多代码，使用State Info来协助用户实现一些默认的实现。


&emsp; StateInfo包含有名称，在状态初始化时被赋值。StateInfo同时实现了OnBegin()、OnEnd()方法。此外，StateInfo的EnableSameTransit()方法还能判断是否允许状态在同类状态中转移，CanTransi To()方法能判断是否能转移到某个目标状态

```
/**
 * @description: 状态基本信息
 */
// 状态的基础信息和默认实现
type StateInfo struct {
	// 状态名
	name string
}

// 状态名
func (s *StateInfo) Name() string {
	return s.name
}

// 提供给内部设置名字
// setName()方法的首字母小写，表示这个方法只能在同包内被调用。
// 这里我们希望setName()不能被使用者在状态初始化后随意修改名称，而是通过后面提到的状态管理器自动赋值
func (s *StateInfo) setName(name string) {
	s.name = name
}

// 允许同状态转移
func (s *StateInfo) EnableSameTransit() bool {
	return false
}

// 默认将状态开启时实现
func (s *StateInfo) OnBegin() {

}

// 默认将状态结束时实现
func (s *StateInfo) OnEnd() {}

// 默认可以转移到任何状态
func (s *StateInfo) CanTransitTo(name string) bool {
	return true
}
```


<br/>

> **状态管理**

&emsp; 状态管理器管理和维护状态的生命期。用户根据需要，将需要进行状态转移和控制的状态实现后添加（StateManager的Add()方法）到状态管理器里，状态管理器使用名称对这些状态进行维护，同一个状态只允许一个实例存在。状态管理器可以通过回调函数（StateManager的OnChange成员）提供状态转移的通知。

```
/**
 * @description: 状态管理器
 */

type StateManager struct {
	// 已经添加的状态
	// 声明一个以状态名为键，以State接口为值的map
	stateByName map[string]State
	// 状态改变时的回调
	OnChange func(from, to State)
	// 当前状态
	curr State
}

// 添加一个状态到管理器中
func (sm *StateManager) Add(s State) {
	// 获取状态的名称
	name := StateName(s)
	// 将s转换为能设置名字的接口，然后调用该接口
	// 将s（State接口）通过类型断言转换为带有set Name()方法(name string)的接口。
	// 接着调用这个接口的set Name()方法设置状态的名称。使用该方法可以快速调用一个接口实现的其他方法
	s.(interface {
		setName(name string)
	}).setName(name)
	// 根据状态名获取已经添加的状态，检查该状态是否存在
	if sm.Get(name) != nil {
		panic("duplicate state:" + name)
	}
	// 根据名字保存到map中
	sm.stateByName[name] = s
}

// 根据名字获取指定状态
func (sm *StateManager) Get(name string) State {
	if v, ok := sm.stateByName[name]; ok {
		return v
	}
	return nil
}

// 初始化状态管理器
func NewStateManager() *StateManager {
	return &StateManager{
		stateByName: make(map[string]State),
	}
}

```



<br/>


> **在状态间转移**


&emsp; 状态管理器不仅管理状态的实例，还可以控制当前的状态及转移到新的状态。状态管理器从当前状态转移到给定名称的状态过程中，如果发现状态不存在、目标状态不能转移及同类状态不能转移时，将返回error错误对象，这些错误以Err开头，在包（package）里提前定义好。

本例一共涉及3种错误，分别是：

● 状态没有找到的错误，对应Err State Not Found。

● 禁止在同状态间转移的错误，对应Err Forbid Same State Transit。

● 不能转移到指定状态的错误，对应Err Cannot Transit To State。状态转移时，还会调用状态管理器的On Change()函数进行外部通知。

```
/**
 * @description: 在状态间转移
 */
// 状态没有找到的错误
var ErrStateNotFound = errors.New("state not found")

// 禁止在同状态间转移
var ErrForbidSameStateTransit = errors.New("forbid same state transit")

// 不能转移到指定状态
var ErrCannotTransitToState = errors.New("cannot transit to state")

// 获取当前的状态
func (sm *StateManager) CurrState() State {
	return sm.curr
}

// 当前状态能否转移到目标状态
func (sm *StateManager) CanCurrTransitTo(name string) bool {
	if sm.curr == nil {
		return true
	}
	// 相同的状态不用转换
	if sm.curr.Name() == name && !sm.curr.EnableSameTransit() {
		return false
	}
	// 使用当前状态，检查能否转移到指定名字的状态
	return sm.curr.CanTransitTo(name)
}

// 转移到指定状态
func (sm *StateManager) Transit(name string) error {
	// 获取目标状态
	next := sm.Get(name)
	// 目标不存在
	if next == nil {
		return ErrStateNotFound
	}
	// 记录转移前的状态
	pre := sm.curr
	// 当前有状态
	if sm.curr != nil {
		// 相同的状态不用转换
		if sm.curr.Name() == name && !sm.curr.EnableSameTransit() {
			return ErrForbidSameStateTransit
		}
		// 不能转移到目标状态
		if !sm.curr.CanTransitTo(name) {
			return ErrCannotTransitToState
		}
		// 结束当前状态
		sm.curr.OnEnd()
	}
	// 将当前状态切换为要转移到的目标状态
	sm.curr = next
	// 调用新状态的开始
	sm.curr.OnBegin()
	// 通知回调
	if sm.OnChange != nil {
		sm.OnChange(pre, sm.curr)
	}
	return nil
}

```



<br/>


> 自定义状态实现状态接口

```
/**
 * @description: 自定义状态实现状态接口
 */

// 闲置状态
type IdleState struct {
	StateInfo // 使用State Info实现基础接口
}

// 重新实现状态开始
func (i *IdleState) OnBegin() {
	fmt.Println("Idle State begin")
}

// 重新实现状态结束
func (i *IdleState) OnEnd() {
	fmt.Println("Idle State end")
}

// 移动状态
type MoveState struct {
	StateInfo
}

func (m *MoveState) OnBegin() {
	fmt.Println("Move State begin")
}

// 允许移动状态互相转换
func (m *MoveState) EnableSameTransit() bool {
	return true
}

// 跳跃状态
type JumpState struct {
	StateInfo
}

func (j *JumpState) OnBegin() {
	fmt.Println("Jump State begin")
} // 跳跃状态不能转移到移动状态
func (j *JumpState) CanTransitTo(name string) bool {
	return name != "Move State"
}



// 状态机调用
func testInterface6() {
	// 实例化一个状态管理器
	sm := NewStateManager()
	// 响应状态转移的通知
	sm.OnChange = func(from, to State) {
		// 打印状态转移的流向
		fmt.Printf("%s ---> %s\n\n", StateName(from), StateName(to))
	}
	// 添加3个状态
	sm.Add(new(IdleState))
	sm.Add(new(MoveState))
	sm.Add(new(JumpState))
	// 在不同状态间转移
	transitAndReport(sm, "IdleState")
	transitAndReport(sm, "MoveState")
	transitAndReport(sm, "MoveState")
	transitAndReport(sm, "JumpState")
	transitAndReport(sm, "JumpState")
	transitAndReport(sm, "IdleState")
}

```

输出：

```
<=============== 🍎 🍎 🍎 ===============> 

Idle State begin
none ---> IdleState

Idle State end
Move State begin
IdleState ---> MoveState

Move State begin
MoveState ---> MoveState

Jump State begin
MoveState ---> JumpState

FAILED! JumpState --> JumpState, forbid same state transit

Idle State begin
JumpState ---> IdleState



<=============== 🍑 🍑 🍑 ===============> 
```
