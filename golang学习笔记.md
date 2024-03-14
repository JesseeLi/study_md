## golang学习笔记

#### **一、基础知识**

##### **1. 结构、包、变量**

声明显式变量 	var 变量名 数据类型 = 值	例如 var a string = "hello"

声明隐式变量	变量名 := 值			例如 a := "hello"

关键字不能作为变量名，不能使用数字开头作为变量名

25个关键字：

- 包管理：`import、 package`

- 变量与常量：`var、 const`

- 函数：`func、 return`

- 自定义类型：`interface、 struct、 type(定义类型)`

- 引用类型：`map(Map 是一种无序的集合，我们可以像迭代数组和切片那样迭代它，它是使用 hash 表来实现的。)、 range(从slice、map等结构中取元素)`

- 并发：`go(并发执行)、 select(go语言特有的channel选择结构)、 chan(定义channel)`

- 流程控制：`if、 else、 switch、 case、 default、 fallthrough(如果case带有fallthrough，程序会继续执行下一条case,不会再判断下一条case的值)、 for、 break、 continue、 goto(跳转语句)`

- 延迟型流程控制：`defer`

预定义标识符：

- 内建常量：`true、 false、 iota、 nil` 
- 内建类型：`int、 int8、 int16、 int32、 int64、 uint、 uint8、 uint16、 uint32、 uint64、 uintprt、 float32、 float64、 complex64、 complex128、 bool、 byte、 rune、 string、 error`
- 内建函数：`make、 len（长度）、 cap（容量）、 new、 append、 copy、 close、 delete、 complex、 real、 imag、 panic、 recover、 print、 println` 

注释

- 单行注释： //
- 多行注释：/**/

包

- 一个文件夹只能有一个包，但是可以有多个文件。一个项目内main包只能有一个
- 引入包修改别名	例如：import (fm  "fmt")。另一种 import (.  "fmt")，就直接引入当前包内，不用写包前缀
- 作用域。包内全局变量   小写字母开头为私有，只能包内使用，不能跨包使用。大写字母开头可以跨包使用

#####  **2. 基本数据类型**

基本数据类型

- 整数类型
	- **uint**、 uint8（0到255）、 uint16（0到65535）、 uint32、 uint64
	- **int**、 int8（-128到127）、 int16（-32768到32767）、 int32、 int64
	- byte（需要字节存储时才会用到，一般是 字节组成的数组）
	- rune（等价于 int32 ，存储一个unicode编码）
	
- 浮点类型
	- float32
	- **float64**

- 字符串类型
	- 加上双信号都是字符串，"这就是字符串"

- 布尔类型
	- true
	- false

复杂数据类型

- 结构体 struct
- 接口 interface
- 数组 [数组长度]值类型{值1,值2,值3}
- 切片 slice []值类型{值1,值2,值3}
- map [key类型]值类型{key:值}
- 指针 *
- 函数 func
- 管道 chan


判断数据类型，使用fmt.Printf("%T",变量或值)
数据类型转换

- string到int
	- int,err = strconv.Atoi(string)
- string到int64
	- int64,err = strconv.ParseInt(string,10,64)
- int到string
	- string,err = strconv.Itoa(int)
- int64到string
	- string,err = strconv.FormatInt(int64,10)
- string到float32/float64
	- float32,err = ParseFloat(string,32)
	- float64,err = ParseFloat(string,64)

#####  **3. 流程控制语句**

递增递减语句，go 只有后置++
- ++ 自增
-  -- 自减

条件语句
- if
- if else
- if elseif

选择语句
- switch
	- case
	- fallthrough  如果case带有fallthrough，程序会继续执行下一条case,不会再判断下一条case的值
	- default

循环语句
- for
	- 死循环 for{}
	- for a := 0; a < 10; a++{}
	- a := 10    for a<10{}

跳转语句
- continue 跳出本次循环
- break 结束循环
- goto 调整执行位置，跳转位置。使用：goto 标签名 

#####  **4. 数组和切片**

数组的长度是固定的，是不可更改的。切片的长度是可以改变的（不固定长度的数组）。

数组定义

- 固定长度数组定义：[元素长度]元素类型{元素1,元素2,......}

- 未知长度数组定义：[...]元素类型{元素1,元素2,......}

- 先声明数组，再赋值：new([元素长度]元素类型)

遍历数组
- 使用 for 循环数组下标
- 使用 range 内置关键字。 for i,v := range a{}

切片 
- 定义切片 
	- var aa []int  生成的是空的[]
	- make([]int,4)  生成的是 定义好的长度4的切片 [0,0,0,0]
- 先定义一个数组，获取切片。（前闭后开）
	- [:] 获取全部数组 
	- [2:] 获取下标2之后的所有数组，包含下标2
	- [:5] 获取下标5之前的所有数组，不包含下标5 
	- [2:5] 获取下标2到下标5的所有数组，包含下标2 不包含下标5
- append   向切片里添加元素，可以扩充cap。当len长度超过cap时，会自动分配内存(避免多次申请)
- copy  把第二个切片里的内容拷贝到第一个切片里面并覆盖

#####  **5. map 的声明和使用**

map 类似于其他语言 字典 或 哈希表 的东西，表现为 key - value，，键值对。关联数组

map 定义(常用 map[key数据类型]value数据类型{})
- var m map[string]string		m = map[string]string{}
- m1 := map[string]string{}
- m2 := make(map[string]string)

遍历map 使用 for range

使用 delete 删除 map 指定元素

#####  **6. 函数 func**

基本结构

- func 函数名(入参1 入参类型，入参2 入参类型 ...)(出参1 出参类型，出参2 出参类型 ...){函数体 、写一些逻辑代码、一定要 return 你的出参 符合出参类型 且入参一定要使用}   出参可写可不写

匿名函数
- 可以在函数内部定义。 a := func(){}

自执行函数
- 声明的那一刻就执行。(func(){})()

闭包函数
- 一个函数返回一个函数。func ()func(){}

延迟调用函数
- 调用函数前 使用 defer  后 函数会最后执行。 func A(){}       defer A()

其他知识
- 不定项参数、一定在入参最后一位。使用  ... 来表示传入多个参数，函数内接收为一个切片。 func m(data1 int,data2 ... string)。 		可以使用数组来传递不定项参数，arr := []string{"1","2","3"} 	m(1, arr...)
- 函数返回值、定义好的出参，可以在函数内使用赋值、赋值之后可以直接 return 。简略掉返回变量

#####  **7. 指针和地址**

指针

- 用  *  表示，数据类型前添加，表示这个变量本身没有存储任何值，而是拿到了原本数据的存储路径。  var a *int
- 不能执行具体的数值或者变量，只能执行和它类型一样的地址
- 指针执行一个变量的原始地址，用 * 表示修改原始变量的值

地址

- & 来表示变量的地址
- 地址需要和指针搭配使用，go也可以隐式赋值得到某个地址

#####  **8. 结构体 struct 声明和使用**

可以存储不同数据类型的数据类型，声明完之后，可以和数据类型一样使用，也有指针、值 等

type 结构体名 struct{}

声明

- var 变量名 结构体名		例如 var qm QM
- 变量名 := 结构体{}    		例如 qm := QM{}	
- 变量名 := new(结构体名)	 例如 qm := new(QM)

使用

- 直接访问使用  qm.XXX
- 作为函数 参数使用  func A(qm QM){}
- 指针使用。 qmp *QM    qmp = &qm 。    (\*qmp).Name
- 结构体内添加函数方法 。 func (q *QM) song(){}
- 结构体嵌套结构体

#####  **9. 接口 interface**

接口是一类规范 是某一些方法的集合

type 接口名 interface{}

先定义接口，随后使用结构体实现 接口内定义的全部方法 。

- type Animal interface {eat() run()}		type Dog struct{} 
- func (d Dog) eat (){} 		func (d Dog) run () {}

使用接口 实现类似 泛型 的功能。 函数参数定义为 interface 后 ，可以传入任何类型的参数

#####  **10. 并发神器 goroutine 和 channel**

在一个方法前添加 go 就是 goroutine ，它会让方法 异步 执行。   func Demo(){}		go Demo()

协程管理器

- sync.WaitGroup()
- sync.Add()
- sync.Wait()
- sync.Done()



channel 是 goroutine 之间的桥梁。

定义chan

- 可读可取  c := make(chan int)
- 可读  var readChan <-chan int = c
- 可取  var setChan chan <- int = c
- 有缓冲  c := make(chan int,5)
- 无缓冲  c := make(chan int)

channel 开启之后是可以 close 的，当不再需要并且已经 set 完成的时候，记得 close 。注意使用 range 则必须在range 之前关闭它。 close 之后的数据依然会存在于管道内，可以获取 不能添加

select 

#####  **11. 断言 Assertion 和反射 reflect**

断言 把一个接口类型指定为它的原始类型

通过 v.(type) 获取当前传入数据类型

`func check1(v interface{}) {`
	`switch v.(type) {`
	`case User:`
		`fmt.Println("我是User")`
		`fmt.Println(v.(User).Name)`
	`case Student:`
		`fmt.Println("我是Student")`
		`fmt.Println(v.(Student).Class)`
	`}`
`}`

反射

- 在编译不知道类型的情况下，可 更新变量、运行时查看值、调用方法 以及 直接对他们的布局进行操作的机制。
- 可以知道本数据的 原始数据类型、数据内容、方法等，并且可以进行一定操作
- 改变原始数据必须传入地址
- 举例
  - 普通反射
  - struct 反射
  - 匿名或嵌入字段的反射
  - 判断传入的类型是否是我们想要的类型
  - 通过反射修改内容
  - 通过反射调用方法

一些方法
- reflect.ValueOf()		获取输入参数接口中的数据的值
- reflect.TypeOf()		动态获取输入参数接口中的值的类型
- reflect.TypeOf.Kind()		用来判断类型
- reflect.ValueOf().field(int)		用来获取值
- reflect.FieldByIndex([]int{0,1})		层级取值
- reflect.ValueOf().Elem()		获取原始数据并操作

#####  **12. 泛型**

#### **二、标准包的使用**

#####  **1. context 包**



#####  **2. 并发编程 sync 包**

Mutex 互斥锁

- Lock()
- Unlock()

RWMutex 读写互斥锁

- Lock()
- Unlock()
- Rlock()		锁定为 读取状态，禁止其他线程写入，但不禁止读取（都能同时读，但不能同时写）
- Runlock()		解除rw的锁定状态

Once  Once.Do（一个函数）这个方法无论被调用多少次，这里只会执行一次

WaitGroup

- Add(delta int) 设定需要Done多少次
- Done() Done一次加1
- Wait() 在到达Done的次数前一直堵塞

Map 一个并发字典

- Store
- Load
- LoadOrStore
- Range
- Delete

Pool 并发池

- Put
- Get

Cond 没多大用的通知解锁

- NewCond(lock) 创建一个Cond
- co.L.Lock()		co.L.Unlock()		创建一个锁区间 在区域内部都可以co.wait()
- co.Broadcast()	解锁全部
- co.Signal()	解锁一个 

#####  **3. 文件操作**

io包基础接口

- type Reader interface{Read(p []byte)(n int,err error)}
  - 将len(p) 个字节读取到p中
  - ReadFrom()常用方法  实现了 Reader 接口的都可以用
- type Writer interface{Write(p []byte)(n int,err error)}
  - Write 方法将 p 中 的数据写入到对象的数据流中
- type Seeker interface{Seek(offset int64,whence int)(ret int64,err error)}
  - Seek 设置下一次读写操作的指针位置，每次读写操作都是从指针位置开始的
    - whence 为 0 ：表示从数据的开头开始移动指针
    - whence 为 1 ：表示从数据的当前指针位置开始移动指针
    - 如果 whence 为 2 ： 表示从数据的尾部开始移动指针
  - SeekStart = 0
  - SeekCurrent = 1
  - SeekEnd = 2
  - offset 是指针移动的的偏移量
- Type Closer interface{Close() error}
  - Close 一般用于关闭文件，关闭链接，关闭数据库等

一些常量

- O_RDONLY int = syscall.O_RDONLY		只读模式打开文件
- O_WRONLY int = syscall.O_WRONLY		只写模式打开文件
- O_RDWR int = syscall.O_RDWR		读写模式打开文件
- O_APPEND int = syscall.O_APPEND		写操作时将数据附加到文件尾部
- O_CREATE int = syscall.O_CREATE		如果不存在则创建一个新文件
- O_EXCL int = syscall.O_EXCL		和 O_CREATE 配合使用，文件必须不存在
- O_SYNC int = syscall.O_SYNC		打开文件用于同步I/O
- O_TRUNC int = syscall.OTRUNC		如果可能，打开时清空文件

读取文件的几个关键方法

- os.OpenFile()		用于打开文件 获取到 *file
- bufio.NewReader(f)		将文件转换为 reader
- reader.ReadString('字符')		调用 Reader 上的方法，还有ReadLine、ReadByte、ReadSlice 等
- Ioutil.ReadAll(f)		直接读取整个文件 os.ReadFile(文件路劲) 也能达到同样效果
- os.ReadDir("./")		读取文件夹 获取目标文件夹下的文件信息

写文件的几个关键方法

- os.OpenFile()		用于打开文件 获取到 *file
- f.Seek()		挪动光标位置
- f.WriteString()		直接写入
- bufio.NewWrite(f)		创建一个缓存的写
  - writer.WriteString()		先写入内存
  - writer.Flush()		缓存内容生效 写入文件

复制文件

- Io.Copy(f1,f2)		open两个文件，将后面的文件写入到前面的文件内，只是覆盖操作

#####  **4. net 包**

######  **4.1 tcp 模块**



######  **4.2 http 模块**

###### **4.3 RPC 模块**

###### **4.4 WebSocket 模块**