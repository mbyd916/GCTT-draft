# 检测Go程序中按值传递的locks

`go tool vet -copylocks`命令简介

Go 语言安装包附带[vet](https://golang.org/cmd/vet/)命令行工具。该工具能对程序源码运行一套启发式算法以发现可疑的程序结构，如无法执行的代码或对```fmt.Printf```函数的错误调用（指arguments没有对齐format参数）：

```go
package main
import "fmt"

func f() {
    fmt.Printf("%d\n")
    return
    fmt.Println("Done")
}
```
```
> go tool vet vet.go
vet.go:8: unreachable code
vet.go:6: missing argument for Printf("%d"): format reads arg 1, have only 0 args
```

本文专讲该工具的copylocks选项。让我们看看它能做什么以及如何在实际的程序中发挥作用。

假设程序使用互斥锁进行同步：

```go
package main
import "sync"

type T struct {
    lock sync.Mutex
}
func (t *T) Lock() {
    t.lock.Lock()
}
func (t T) Unlock() {
    t.lock.Unlock()
}

func main() {
    t := T{lock: sync.Mutex{}}
    t.Lock()
    t.Unlock()
    t.Lock()
}
```

> 如果变量 v 是可寻址的，并且 &v 的方法集合包含 m，那么 v.m() 是 (&v).m() 的简写。

想一想上述程序运行的结果可能是什么...

程序会进入死锁状态：

```
fatal error: all goroutines are asleep — deadlock!
goroutine 1 [semacquire]:
sync.runtime_Semacquire(0x4201162ac)
    /usr/local/go/src/runtime/sema.go:47 +0x30
sync.(*Mutex).Lock(0x4201162a8)
    /usr/local/go/src/sync/mutex.go:85 +0xd0
main.(*T).Lock(0x4201162a8)
...
```

运行上述程序得到了糟糕的结果，根本原因是把receiver按值传递给Unlock方法，所以 ```t.lock.Unlock()``` 实际上是由lock的副本调用的。很容易忽视这点，特别在更大型的程序中。Go编译器不会检测这方面，因为这可能是程序员有意为之。该vet工具登场啦...

```
> go tool vet vet.go
vet.go:13: Unlock passes lock by value: main.T
```

Option copylocks (enabled by default) checks if passed by value is something of a type having Lock method with pointer receiver. If this is the case then it throws a warning.

选项copylocks (默认启用) 会检测含有Lock方法(实际需要pointer receiver)的type是否按值传递。如果是这种情况，则会发出警告。

sync包有使用该机制的例子，它有一个命名为noCopy的特殊type。为了避免某type按值拷贝(实际上通过vet工具进行检测)，需要往struct定义中添加一个field(如WaitGroup):

```go
package main
import "sync"
type T struct {
    wg sync.WaitGroup
}
func fun(T) {}
func main() {
    t := T{sync.WaitGroup{}}
    fun(t)
}
```

```
> go tool vet lab.go
lab.go:9: fun passes lock by value: main.T contains sync.WaitGroup contains sync.noCopy
lab.go:13: function call copies lock value: main.T contains sync.WaitGroup contains sync.noCopy
```

Under the hood

深入理解该机制


Sources are placed in /src/cmd/vet. Every option for vet registers itself using register function which takes (among others) a variadic parameter of types of AST nodes that option is interested in and a callback. That callback function will be fired for every node of specified types. For copylocks nodes to investigate are i.e. return statements. Ultimately it all goes to lockPath which verifies if passed value is of type which has a pointer receiver method named Lock. During the whole process go/ast package is used extensively. A gentle introduction to that package can be found in Go’s Testable Examples under the hood.

vet工具的源代码放在/src/cmd/vet路径下。vet的每一个选项都利用注册函数注册自己，注册函数以该选项感兴趣的AST结点类型的可变参数以及一个回调函数。该回调函数将因特定类型的结点触发。对于copylocks结点，需要发现的是，如 return语句。最终它会走到lockPath，以验证传入的值是否有一个以指针接收者的Lock方法。在整个处理过程中，go/ast包使用非常频繁。对该包的一个入门介绍可以在Go可测试的样例中找到。

多点击下方的"👏"按钮， 以帮助其他人找到这篇文章哦。如果您想获得有关新帖子的更新或未来工作进展的消息， 请在这儿或者 Twitter上关注我。
----------------

via: https://medium.com/golangspec/detect-locks-passed-by-value-in-go-efb4ac9a3f2b

作者：[Michał Łowicki](https://medium.com/@mlowicki)
译者：[mbyd916](https://github.com/mbyd916)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [GCTT](https://github.com/studygolang/GCTT) 原创编译，[Go 中文网](https://studygolang.com/) 荣誉推出
