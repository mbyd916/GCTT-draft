# Detect locks passed by value in Go

# 检测Go程序中按值传递的locks

Introduction to `go tool vet -copylocks`

`go tool vet -copylocks`命令简介

Go is shipped with vet command line tool. It runs set of heuristics on a source code to find suspicious constructs like unreachable code or calls to fmt.Printf where arguments aren’t aligned with desired format:

Go 语言安装包附带[vet](https://golang.org/cmd/vet/)命令行工具。它对源码运行启发式算法以发现可疑的程序结构，如无法到达的代码或对```fmt.Printf```的调用存在参数没有按希望的格式排列：

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

This story is about one option specifically — copylocks. Let’s what it does and how it can be useful in real-world programs.

本文专讲该工具的一个选项 — copylocks。让我们看看它能做什么以及如何在实际的程序中发挥作用。

Suppose the program uses mutex for synchronization:

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

> 如果 v 变量是可寻址的，并且 &v 的方法集合包含 m，那么 v.m() 是 (&v).m() 的简写。

Think for a moment what might be the result of running what is implemented above…

想一想上述程序运行的结果可能是什么...

Program falls into a deadlock:

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

It’s not good and the root cause is passing receiver by value to Unlock method so t.lock.Unlock() is actually called on a copy of the lock. It’s very easy to overlook, especially in bigger programs. It isn’t detected by the compiler since this might be an intention of the programmer. This is where vet steps in…

这样的结果不对，根本原因在于接收者是按值传递给Unlock方法的，所以 t.lock.Unlock() 实际上是由lock的拷贝调用的。这点很容易被忽视，特别在更大的程序中。Go编译器不会检测这方面，因为这可能是程序员有意为之。这时vet工具派上用场啦。。。

```
> go tool vet vet.go
vet.go:13: Unlock passes lock by value: main.T
```






----------------

via: https://medium.com/golangspec/detect-locks-passed-by-value-in-go-efb4ac9a3f2b

作者：[Michał Łowicki](https://medium.com/@mlowicki)
译者：[mbyd916](https://github.com/mbyd916)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [GCTT](https://github.com/studygolang/GCTT) 原创编译，[Go 中文网](https://studygolang.com/) 荣誉推出
