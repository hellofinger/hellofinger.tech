---
title: Go基础--Defer关键字
date: 2021-03-01 20:24:00
slug: go-defer
tags:
  - Go
categories:
  - Go
---

## 概述

在Go语言中，`defer`关键字用于延迟（defer）函数的执行，即在函数执行结束时再执行被延迟的函数。`defer`通常用于确保一些清理工作在函数执行完毕后被执行，无论函数是正常返回还是发生了panic。

`defer`具有以下特点：

- 在函数返回之前执行
- 可以放在函数中任意位置
- 可以同时执行多个defer函数，多个defer按栈(FILO)先进后出顺序执行
- defer函数的传入参数在定义时就已经明确
- 可以修改函数返回值
- 用于文件IO、数据库连接等释放和关闭

### 在函数返回之前执行

defer语句函数会在函数返回之前执行，下面程序将会依次输出B A：

```go
package main

import "fmt"

func main() {
    defer fmt.Println("A")
    fmt.Println("B")
}

```

```bash
>go run main.go
B
A
```

### defer可以放在函数中任意位置

```go
package main

import "fmt"

func main() {
    fmt.Println("A")
    defer fmt.Println("B")
    fmt.Println("C")
}

```

```bash
>go run main.go
A
C
B
```

*注意：*

1. defer语句一定要在函数return语句之前，这样才能生效。下面程序将会只输出A

```go
func main() {
    fmt.Println("A")
    return 
    fmt.Println("B")
}
```

2. 在调用os.Exit时候，defer不会执行。下面程序只会输出B

```go
func main() {
    defer fmt.Println("A")
    fmt.Println("B")
    os.Exit(0)
}
```

### 可以同时执行多个defer函数

多个defer函数执行遵循先进后出（FILO）顺序

```go
package main

import "fmt"

func main() {
    defer fmt.Println("A")
    fmt.Println("B")
    defer fmt.Println("C")
    fmt.Println("D")
}

```

```bash
go run main.go
B
D
C
A
```

多个defer嵌套情况：

```go
package main

import "fmt"

func main() {
    fmt.Println("A")
    defer func() {
        fmt.Println("B")
        defer fmt.Println("C")
        fmt.Println("D")
    }()

    defer fmt.Println("E")
    fmt.Println("F")
}


```

```bash
>go run main.go
A
F
E
B
D
C
```

### defer函数的传入参数在定义时就已经明确

defer函数的传入参数在定义时就已经明确，不论传入的参数是变量、表达式、函数语句，都会先计算出计算出实参结果，再随defer语句入栈

```go
package main

import "fmt"

func main() {
    i := 1
    defer fmt.Println(i)
    i++
    return
}

```

上面程序输出1，而不是2

```bash
>go run main.go
1
```



*注意：* 当defer类似闭包使用时候，访问的总是循环中最后一个值

```go
package main

import "fmt"

func main() {
    for i := 0; i < 5; i++ {
        defer func() {
            fmt.Println(i)
        }()
    }
}

```

上面程序连续输出5个5

```bash
>go run main.go
5
5
5
5
5
```

解决办法可将值传入闭包函数中

```go
package main

import "fmt"

func main() {
    for i := 0; i < 5; i++ {
        defer func(i int) {
            fmt.Println(i)
        }(i)
    }
}

```

此时依次输出4 3 2 10

```bash
>go run main.go
4
3
2
1
0
```

### 可以修改函数返回值

```go
package main

import "fmt"

func main() {
    fmt.Println(test())
}

func test() (i int) {
    defer func() {
        i++
    }()
    return 100
}

```

```bash
>go run main.go
101
```

### 用于文件IO、数据库连接等释放和关闭

没有使用defer的代码

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }

    written, err = io.Copy(dst, src)
    dst.Close()
    src.Close()
    return
}

```

使用defer后的代码

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.Close()

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    defer dst.Close()

    return io.Copy(dst, src)
}

```

## 参考

- [1] https://juejin.cn/post/6886710490530054158