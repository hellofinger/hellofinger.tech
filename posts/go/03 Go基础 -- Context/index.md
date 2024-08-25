---

title: Go基础--Context
date: 2021-02-26 14:07:52
slug: go-basic-context
tags:
  - Go
categories:
  - Go

---

## 什么是 Context

Context 是 Go 并发编程中的一种编程模式。
主要指的是 goroutine的上下文，包含 goroutine 的运行状态，环境，现场等信息。
Context 主要用来在goroutine 之间传递上下文信息，包含：取消信号，超时时间，截止时间，k-v等。

### 为什么需要 Context

在并发程序中，由于超时，取消操作或者一些异常情况，往往需要进行抢占操作或者中断后续的操作。例如：主协程中有多个任务1,2，...m，主协程对这些任务有超时控制；而其中任务1又有多个子任务1,2，...n。任务1对这些子任务也有自己的超时控制，那么这些子任务既要感知主协程的取消信号，也需要感知任务1的取消信号。
因此我们需要解决以下问题：
1、上层任务取消后，所有的下层任务都会被取消
2、中间某一层的任务取消后，只会将当前任务的下层任务取消，而不会影响上层的任务以及同级任务。

### Context 的定义

```go
type Context interface {
    // 当 context 被取消或者到了 deadline，返回一个被关闭的 channel
    Done() <-chan struct{}

    // 在 channel Done 关闭后，返回 context 取消原因
    Err() error

    // 返回 context 是否会被取消以及自动取消时间（即 deadline）
    Deadline() (deadline time.Time, ok bool)

    // 获取 key 对应的 value
    Value(key interface{}) interface{}
}
```
Context 是一个接口，定义了4个方法，它们都是幂等的。也就是说连续多次调用同一个方法，得到的结果都是相同的。

- Deadline()(deadline time.Time,ok bool)：返回 context 的截止时间，通过此时间，函数就可以决定是否进行接下来的操作，如果时间太短，就可以不往下做了，否则浪费系统资源。当然，也可以用这个 deadline 来设置一个 I/O 操作的超时时间。

- Done() <-chan struct{}： 返回一个 channel，可以表示 context 被取消的信号：当这个 channel 被关闭时，说明 context 被取消了。注意，这是一个只读的channel。 我们又知道，读一个关闭的 channel 会读出相应类型的零值。并且源码里没有地方会向这个 channel 里面塞入值。换句话说，这是一个 receive-only 的 channel。因此在子协程里读这个 channel，除非被关闭，否则读不出来任何东西。也正是利用了这一点，子协程从 channel 里读出了值（零值）后，就可以做一些收尾工作，尽快退出。

- Err()： 返回一个错误，表示 channel 被关闭的原因。例如是被取消，还是超时。

- Value()： 获取该Context上绑定的值，是一个键值对，所以要通过一个Key才可以获取对应的值，这个值是线程安全的

Context接口不需要我们自己实现，go内置已经实现了2个。
一个是Background，主要用于main函数 、初始化以及测试代码中，作为Context这个树结构的最顶层的Context，也就是根Context

一个是todo，它是目前还不知道具体的使用场景，如果我们不知道该使用什么Context的时候，可以使用这个

他们两个本质上都是emptyCtx结构体类型，是一个不可取消，没有设置截止时间，没有携带任何值的Context

```go

type emptyCtx int

func (*emptyCtx) Deadline() (deadline time.Time, ok bool) {
    return
}

func (*emptyCtx) Done() <-chan struct{} {
    return nil
}

func (*emptyCtx) Err() error {
    return nil
}

func (*emptyCtx) Value(key interface{}) interface{} {
    return nil
}

var (
    background = new(emptyCtx)
    todo       = new(emptyCtx)
)

func Background() Context {
    return background
}

func TODO() Context {
    return todo
}

```

### Context 的继承衍生

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
func WithValue(parent Context, key, val interface{}) Context
```
通过以上函数，就可以创建一个Context树，树的每个节点都可以有任意多个子节点，节点层级可以有任意多个。

WithCancel函数，传递一个父Context作为参数，返回子Context，以及一个取消函数用来取消Context。 

WithDeadline函数，和WithCancel差不多，它会多传递一个截止时间参数，意味着到了这个时间点，会自动取消Context，当然我们也可以不等到这个时候，可以提前通过取消函数进行取消。

WithTimeout和WithDeadline基本上一样，这个表示是超时自动取消，是多少时间后自动取消Context的意思。

WithValue函数和取消Context无关，它是为了生成一个绑定了一个键值对数据的Context，这个绑定的数据可以通过Context.Value方法访问到，后面我们会专门讲。WithCancel 函数，传递一个父Context作为参数，返回子Context，以及一个取消函数用于取消Context。


### 示例

```go
# 一个简单用法
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	go func(ctx context.Context) {
		for {
			select {
			case <-ctx.Done():
				fmt.Println("monitor exit, stopped...")
				return
			default:
				fmt.Println("goroutine monitoring...")
				time.Sleep(2 * time.Second)
			}
		}
	}(ctx)

	time.Sleep(10 * time.Second)
	fmt.Println("ok, notify monitor stop.")

	cancel()

	time.Sleep(5 * time.Second)
}
```

```go
# 使用Channel的方式

package main

import (
	"fmt"
	"time"
)

func main() {
	stop := make(chan bool)

	go func() {
		for {
			select {
			case <-stop:
				fmt.Println("monitor exit, stopped...")
				return
			default:
				fmt.Println("goroutine monitoring...")
				time.Sleep(2 * time.Second)
			}
		}
	}()

	time.Sleep(10 * time.Second)
	fmt.Println("ok, notify monitor stop")
	stop <- true
	time.Sleep(5 * time.Second)
}

```

```go
# Context 控制多个 goroutine
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	go watch(ctx, "monitor1")
	go watch(ctx, "monitor2")
	go watch(ctx, "monitor3")

	time.Sleep(10 * time.Second)
	fmt.Println("ok, notify monitor stop")
	cancel()

	time.Sleep(5 * time.Second)
}

func watch(ctx context.Context, name string) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println(name, "monitor exit, stopped...")
			return
		default:
			fmt.Println(name, "goroutine monitoring...")
			time.Sleep(2 * time.Second)
		}
	}
}

```

```go
# WithValue 传递元数据

package main

import (
	"context"
	"fmt"
	"time"
)

var key string = "name"

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	valueCtx := context.WithValue(ctx, key, "monitor 1")
	go watch(valueCtx)
	time.Sleep(10 * time.Second)

	fmt.Println("ok, notify monitor stop")
	cancel()

	time.Sleep(5 * time.Second)
}

func watch(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println(ctx.Value(key), "monitor exit, stopped...")
			return
		default:
			fmt.Println(ctx.Value(key), "goroutine monitoring...")
			time.Sleep(2 * time.Second)
		}
	}
}

```

### Context 使用原则
- 不要把Context放在结构体中，要以参数的方式传递
- 以Context作为参数的函数方法，应该把Context作为第一个参数，放在第一位
- 给一个函数方法传递Context的时候，不要传递nil，如果不知道传递什么，就使用context.TODO()
- Context的Value相关方法应该传递必须的参数，不要什么数据都使用这个传递
- Context是线程安全的，可以放心的在多个goroutine中传递

## 总结

Context主要用于父子任务之间的同步取消信号，本质上是一种协程调度的方式。另外在使用Context时有两点值得注意：上游任务仅仅使用Context通知下游任务不再需要，但不会直接干涉和中断下游任务的执行，由下游任务自行决定后续的处理操作，也就是说Context的取消操作是无侵入的；Context是线程安全的，因为Context本身是不可变的（immutable），因此可以放心地在多个协程中传递使用。

- 根Context：通过context.Background() 创建
- 子Context：context.WithCancel(parentContext) 创建
- ctx, cancel := context.WithCanncel(context.Background())
- 当前Context被取消时，基于他的子 context 都会被取消
- 接收取消通知 <- ctx.Done()

## 参考
- https://zhuanlan.zhihu.com/p/68792989
- https://zhuanlan.zhihu.com/p/88915174
- https://zhuanlan.zhihu.com/p/110085652
