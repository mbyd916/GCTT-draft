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



----------------

via: https://medium.com/golangspec/detect-locks-passed-by-value-in-go-efb4ac9a3f2b

作者：[Michał Łowicki](https://medium.com/@mlowicki)
译者：[mbyd916](https://github.com/mbyd916)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [GCTT](https://github.com/studygolang/GCTT) 原创编译，[Go 中文网](https://studygolang.com/) 荣誉推出
