[TOC]

## 学习资料汇总

语法标准库先简单看看，用到的时候自然就懂。

| 名称                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 《Go 语言实战，Go in action》                                | 老外的书翻译而来，中规中矩，入门可看。基础的语法、概念等。没有教你环境搭建、工程相关的东西。 |
| 《Go 并发编程实战》                                          |                                                              |
| [Go 语言官网首页](https://golang.google.cn/)                 |                                                              |
| [Go 语言官网使用文档](https://golang.google.cn/doc/)         |                                                              |
| [Go 语言官网标准库文档](https://golang.google.cn/pkg/)       | 标准库。子项目（Sub-repositories），需要时通过 go get 安装。 |
| [Go 语言规范文档](https://golang.google.cn/ref/spec)         | Go 语言所有语法                                              |
| [命令文档页面](https://golang.google.cn/cmd/go/)             | Go 语言命令使用文档，比如go build、go get、go mod的使用      |
| [Go 标准命令详解](https://www.kancloud.cn/cattong/go_command_tutorial/261346) | Go 语言命令使用文档。中文、非官方。                          |
| [GO 命令教程](https://github.com/hyper0x/go_command_tutorial) | 极客时间作者推荐                                             |
| [go doc与godoc](https://github.com/hyper0x/go_command_tutorial/blob/master/0.5.md) | 极客时间作者推荐                                             |
| [极客时间 Go 语言核心36讲](https://time.geekbang.org/column/article/13176) |                                                              |
| [GO 知识图谱](https://www.processon.com/view/link/5a9ba4c8e4b0a9d22eb3bdf0?from=timeline#map) |                                                              |
| [How to avoid Go gotchas](https://divan.dev/posts/avoid_gotchas/) |                                                              |

##  Go 语言的介绍

Go 语言由 Google 出品。2009 年 11 月 10 日诞生。

.s 汇编语言文件



## 新的概念

Go 语言的类型推断，好处有哪些

短变量声明 := 

变量可以被赋值为一个函数（匿名函数），函数也可以作为参数传给另一个函数。 类似 java中 lambada表达式

变量重声明

Go 语言运行时系统，包含哪些部分

nil

errors 包

error 类型 Error 方法 errors.New

Go 静态类型编程语言

python 动态类型编程语言

time.Sleep

time.AfterFunc：在一定时间后执行一个函数

发送表达式

空接口类型interface{}

sync包 sync.WaitGroup

atomic包 LoadUnit32 AddUnit32

runtime包

make函数：初始化通道、切片







### goroutine

goroutine 是可以与其他 goroutine 并行执行的函数，也和主程序并行执行。很像线程，但占用内存少于线程，使用更简单。

go 语句：优先复用一个 goroutine，用这个 G 包装 go 函数，然后追加到 G 的队列中。

 go 函数

主 goroutine 的 go 函数是 main 函数 。主 goroutine 结束，当前 Go 程序结束，此时还未执行的 goroutine 没有执行的机会了。

```go
func log(msg String) {

}

go log
```

- Go 语言并发编程模型：
  - G（goroutine 的缩写）：存在于某个 FIFO 队列中。相当于为需要并发执行代码片段服务的上下文环境。
  - P（processor 的缩写）：可以承载若干个 G，且能够使这些 G 适时地与 M 进行对接，并得到真正运行的中介。
  - M（machine 的缩写）：系统级线程。由调度器申请、销毁。

- 调度器（是不是类似java的线程池？）
  - 用于调度 goroutine、对接系统级线程。统筹调配G、P、M

![img](/Users/fanxudong/IdeaProjects/blog/4 go/asset/M、P、G 之间的关系（简化版）.png)

### 通道

通道是一种数据结构。用于在几个运行的goroutine之间发送数据。保证同一时刻只有一个goroutine修改数据。不提供跨goroutine的数据访问保护机制。



通道：类型、容量、FIFO队列。通道内部有goroutine的发送和接收等待队列。

非缓冲通道：容量为0，同步方式传递数据

缓冲通道：容量大于0，通道已满，所有发送操作阻塞，所在goroutine进入通道内部的发送等待队列。

操作符： <- 接送操作符，将接收到的元素值存起来 elem1 := <-ch1。定义单向、双向通道。

发送操作：复制元素值 + 放置副本到通道内

接收操作：“复制通道内的元素值”“放置副本到接收方”“删掉原值”

通道关闭

通道发送和接收操作的基本特性

对于同一个通道，发送操作之间是互斥的，接收操作之间也是互斥的。

发送操作和接收操作中对元素值的处理都是不可分割的。

发送操作在完全完成之前会被阻塞。接收操作也是如此。

发送和接收之间是互斥的吗？是

Go没有深copy，复制的是指针（引用类型）。数组是值类型，会完全复制。

通道底层是环形链表

单向通道：发送通道、接收通道。所谓函数参数、返回值时，可以用来约束函数内部的行为。

双向通道

select 语句：用于操作通道。语法结构类似switch语句，只是case子句中只能为通道表达式。

### 关键字

https://golang.google.cn/ref/spec#Keywords

```te
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```



## 类型系统

https://golang.google.cn/ref/spec#Types

### Method sets

### Boolean types

```go
package builtin

// bool is the set of boolean values, true and false.
type bool bool

// true and false are the two untyped boolean values.
const (
	true  = 0 == 0 // Untyped bool.
	false = 0 != 0 // Untyped bool.
)
```

### Numeric types

整数、浮点数

大小与架构无关的类型

```tex
uint8       the set of all unsigned  8-bit integers (0 to 255)
uint16      the set of all unsigned 16-bit integers (0 to 65535)
uint32      the set of all unsigned 32-bit integers (0 to 4294967295)
uint64      the set of all unsigned 64-bit integers (0 to 18446744073709551615)

int8        the set of all signed  8-bit integers (-128 to 127)
int16       the set of all signed 16-bit integers (-32768 to 32767)
int32       the set of all signed 32-bit integers (-2147483648 to 2147483647)
int64       the set of all signed 64-bit integers (-9223372036854775808 to 9223372036854775807)

float32     the set of all IEEE-754 32-bit floating-point numbers
float64     the set of all IEEE-754 64-bit floating-point numbers

complex64   the set of all complex numbers with float32 real and imaginary parts 实部和虚部为float32的复数的集合
complex128  the set of all complex numbers with float64 real and imaginary parts 实部和虚部为float64的复数的集合

byte        alias for uint8 别名而已
rune        alias for int32
```

与架构实现有关的类型

```te
uint     either 32 or 64 bits
int      same size as uint
uintptr  an unsigned integer large enough to store the uninterpreted bits of a pointer value 无符号整数，存储指针
```

可移植性问题





## 内存管理



## 工作区和GOPATH

- GOROOT：Go 语言安装根目录的路径，也就是 GO 语言的安装路径。
- GOPATH：若干工作区目录的路径。是我们自己定义的工作空间。放置 Go 语言的源码文件（source file），以及安装（install）后的归档文件（archive file，也就是以“.a”为扩展名的文件）和可执行文件（executable file）。
- GOBIN：GO 程序生成的可执行文件（executable file）的路径。



1. Go 语言源码的组织方式是怎样的；
2. 你是否了解源码安装后的结果（只有在安装后，Go 语言源码才能被我们或其他代码使用）；
3. 你是否理解构建和安装 Go 程序的过程（这在开发程序以及查找程序问题的时候都很有用，否则你很可能会走弯路）。

![image-20210725151238072](/Users/fanxudong/IdeaProjects/blog/4 go/asset/image-20210725151238072.png)

```shell
go build # 构建： 编译、打包 （检查、验证）
go build -a -i
go install # 安装： 编译、打包、链接
go get
go help
```

 

## 访问权限

名称的首字母为大写的程序实体才可以被当前包外的代码引用，否则它就只能被当前包内的其他代码引用。通过名称，Go 语言自然地把程序实体的访问权限划分为了包级私有的和公开的。



### 

