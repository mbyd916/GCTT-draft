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

----------------

via: https://medium.com/golangspec/detect-locks-passed-by-value-in-go-efb4ac9a3f2b

作者：[Michał Łowicki](https://medium.com/@mlowicki)
译者：[mbyd916](https://github.com/mbyd916)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [GCTT](https://github.com/studygolang/GCTT) 原创编译，[Go 中文网](https://studygolang.com/) 荣誉推出
