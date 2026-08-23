美国有一家出版社，专门出版以动物为封面的技术类书籍。我和一些研发人员私下里称其为“动物书”。我买过很多本，有讲 C++ 高性能编程的，也有讲 OpenCV 的。在 AI Coding 出现之前，这些书大多算是整体质量不错的技术指导书。这类书的特点不是学院派地讲述深奥的理论，而是立足于实战。因此，其受众主要是实验室里做项目的学生或科技公司的研发人员。

这本书的封面是一只象鼩，一种生活在非洲的哺乳动物。这让我联想到以前给项目命名的方法：使用系外天体的名称，根本不用担心重名。

这本书是我在 2021 年年末买的。当时我刚离开百度自动驾驶，加入了一家做 DevOps 的公司。在百度时，有一点给我带来了很强烈的震撼：自动驾驶部门拥有非常完善的研发基础设施，尤其是 DevOps 和整套测试体系的架构，在当时处于非常先进的水平。我之前也在国内很优秀的机器人公司待过，相比之下，二者基础设施完善程度的差距十分巨大。当然，一个很重要的原因是百度此前在硅谷设立了美国研发机构，从 Google 等公司招揽了很多技术大牛来做架构。例如，小马智行的楼天城、Anthropic 的创始人当时都曾在百度美国研发中心工作（谁知道他在百度经历了什么……）。所以，百度自动驾驶的基础架构设计在当时还是很有技术含量的。那时我曾有一小段时间有些自卑，觉得自己太菜，不知道得多厉害的人才能设计出这样的架构。

坦率地说，我当时的技术水平相比现在只是一个零头，的确太弱。例如，那套架构整体采用 Web 技术栈，毕竟百度和 Google 都是互联网公司，但我当时对此并不熟悉，只能勉强使用；如果要自己设计或修改，就很困难了。当然，现在看来应该不算什么。也正是因为这份不甘心，我后来才决定去一家专门研发测试工具的公司，学习究竟该如何构建 DevOps 系统，最终也如愿以偿。

几年后再站在现在的角度看当时的架构设计，其实已经能发现一些落后之处。经过几年的学习和项目历练，我也可以很自信地按照自己的想法，自主设计并实现整套软件体系。而且现在赶上了 AI 发展的好时候，这意味着我们可以突破许多过去的设计局限。只要自己足够强，就能做出一些让世界眼前一亮的新东西。

花费大量的时间精力去学习别人的知识经验，从来不是仅仅为了重复，而是为了创造刻有自己印记的新东西。以我的经验，去一个地方，不管是公司、大学还是实验室，第一天就应该问自己一个问题：如果明天我离开这里，有什么东西是我可以带走的？

公司给的职位带不走，平台给的光环带不走，组织赋予的权限和资源同样带不走。真正属于自己的，是在这里建立起来的能力、见过的系统、踩过的坑、形成的判断，以及最终能够独立创造出来的东西。

如果一段经历结束以后，离开平台时的自己和进入平台前并没有本质区别，那么这几年很可能只是工作了几年，而不是成长了几年。

这本书整体难度一般，思路比较常规，API略老，毕竟有点年头了。

# 1. 基础

并发的内容都比较基础。

死锁是指两个协程在各自持有一把锁的同时，都在等待获取对方持有的锁，谁也无法继续执行；如果没有外部干预，程序就会一直卡住。形成死锁需要四个条件：互斥、持有并等待、不可抢占和循环等待。在架构层面避免死锁，除了使用专门的检测和代码分析工具，还应破坏死锁形成的至少一个条件。

活锁类似于两个人迎面而行：A 向左避让时 B 向右避让，A 向右避让时 B 又向左避让，谁都无法通过。活锁比死锁更难检测，因为发生活锁时，代码并没有卡死，CPU 占用等特征也可能不明显。

饥饿是指高优先级的进程或线程不断抢占资源，导致低优先级的进程或线程长时间得不到调度。

Go 除了在 sync 包中提供常规的、基于共享数据的并发支持外，还提供了 goroutine 和 channel 机制。它在系统线程之上增加了一层抽象，通过 Go 运行时将 goroutine 调度到具体的系统线程上执行。采用这种运行时设计的不只有 Go，一些中间件系统也采用了类似设计。“不要通过共享内存来通信，而应通过通信来共享内存”，这是 Go 在并发方面的一种设计哲学。在实际开发中，究竟什么场景应使用传统的同步原语，什么场景应使用 CSP？保护共享状态、构建极低延迟的数据结构时，优先使用底层并发原语；描述任务流、事件流、生产者—消费者模型和流水线时，优先使用 CSP；在复杂系统中，最常见的做法是将二者结合使用。

## 1.1 sync 包
这个包主要提供底层的内存访问同步原语。

### WaitGroup
等待一组协程运行结束，使用起来比较简单。

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var wg sync.WaitGroup

	for i := 1; i <= 3; i++ {
		// 启动一个 goroutine，就 +1
		wg.Add(1)

		go func(id int) {
			// goroutine 结束时 -1
			defer wg.Done()

			fmt.Printf("任务 %d 开始执行\n", id)

			time.Sleep(time.Second)

			fmt.Printf("任务 %d 执行完成\n", id)
		}(i)
	}

	// 等待所有 goroutine 执行完成
	wg.Wait()

	fmt.Println("所有任务执行完成")
}
```
上述代码在每次循环中启动一个协程，并在主协程中等待它们结束。每启动一个协程，计数就加 1；每结束一个协程，计数就减 1。wg 的 Wait 方法会一直等待，直到计数变为 0，然后继续执行。

### 互斥锁和读写锁
sync.Mutex 就是常规的互斥锁，访问共享数据时必须独占该锁。
sync.RWMutex 是读写互斥锁，多个读取协程可以同时访问共享数据；发生写入时，其他读写协程都需要等待。

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var (
	mu   sync.RWMutex
	data = map[string]string{
		"name": "Tom",
	}
)

// 读取数据
func read(key string) string {
	// 加读锁
	mu.RLock()
	defer mu.RUnlock()

	fmt.Println("开始读取:", key)
	time.Sleep(time.Second)

	return data[key]
}

// 修改数据
func write(key, value string) {
	// 加写锁
	mu.Lock()
	defer mu.Unlock()

	fmt.Println("开始修改:", key)
	time.Sleep(time.Second)

	data[key] = value
}

func main() {
	var wg sync.WaitGroup

	// 两个读操作可以同时执行
	wg.Add(2)

	go func() {
		defer wg.Done()
		fmt.Println("读取结果:", read("name"))
	}()

	go func() {
		defer wg.Done()
		fmt.Println("读取结果:", read("name"))
	}()

	wg.Wait()

	// 写操作必须独占
	wg.Add(1)

	go func() {
		defer wg.Done()
		write("name", "Jack")
	}()

	wg.Wait()

	fmt.Println("最终结果:", read("name"))
}
```
对于缓存等读多写少的场景，读写互斥锁相比普通互斥锁可以提升系统吞吐量。

### Cond
条件变量可以让协程在条件不满足时进入等待状态，并在收到信号后继续执行后续代码。

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var mu sync.Mutex

	// Cond 通常需要配合 Mutex
	cond := sync.NewCond(&mu)

	ready := false

	// 消费者
	go func() {
		mu.Lock()

		for !ready {
			fmt.Println("条件不满足，等待...")

			// 进入等待
			cond.Wait()
		}

		fmt.Println("条件满足，开始执行")

		mu.Unlock()
	}()

	// 模拟生产者准备数据
	time.Sleep(2 * time.Second)

	mu.Lock()

	ready = true

	fmt.Println("数据准备完成")

	// 唤醒一个等待的 goroutine
	cond.Signal()

	mu.Unlock()

	time.Sleep(time.Second)
}
```
这就是条件变量的基本使用方法。

Cond 提供两种通知机制：Signal 会唤醒一个正在等待的协程，Broadcast 则会唤醒所有正在等待的协程。

下面是 Broadcast 的示例。
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var mu sync.Mutex
	var wg sync.WaitGroup

	cond := sync.NewCond(&mu)

	ready := false

	// 启动 3 个 worker
	for i := 1; i <= 3; i++ {
		wg.Add(1)

		go func(id int) {
			defer wg.Done()

			mu.Lock()

			// 条件不满足就等待
			for !ready {
				fmt.Printf("worker %d: 等待中...\n", id)
				cond.Wait()
			}

			// Wait 返回时，当前 goroutine 已经重新拿到了 mu
			fmt.Printf("worker %d: 收到广播\n", id)

			mu.Unlock()

			// 模拟真正处理业务
			fmt.Printf("worker %d: 开始处理任务\n", id)
			time.Sleep(time.Second)
			fmt.Printf("worker %d: 处理完成\n", id)
		}(i)
	}

	// 只是为了方便演示，让 worker 有时间进入 Wait
	time.Sleep(2 * time.Second)

	// 修改条件
	mu.Lock()

	fmt.Println("main: 修改 ready = true")

	ready = true

	// 唤醒所有 Wait 的 goroutine
	fmt.Println("main: Broadcast")
	cond.Broadcast()

	mu.Unlock()

	// 等待所有 worker 真正执行完成
	wg.Wait()

	fmt.Println("main: 所有 worker 执行完成")
}
```
运行结果

```shell
go run broadcast.go 
worker 3: 等待中...
worker 2: 等待中...
worker 1: 等待中...
main: 修改 ready = true
main: Broadcast
worker 1: 收到广播
worker 1: 开始处理任务
worker 3: 收到广播
worker 3: 开始处理任务
worker 2: 收到广播
worker 2: 开始处理任务
worker 2: 处理完成
worker 1: 处理完成
worker 3: 处理完成
main: 所有 worker 执行完成
```
### sync.Once
保证函数只被调用一次。

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var once sync.Once
	var wg sync.WaitGroup

	for i := 1; i <= 5; i++ {
		wg.Add(1)

		go func(id int) {
			defer wg.Done()

			fmt.Printf("goroutine %d 调用 once.Do()\n", id)

			once.Do(func() {
				fmt.Println("===== 初始化，只会执行一次 =====")
			})
		}(i)
	}

	wg.Wait()
}

```

运行结果

```shell
goroutine 5 调用 once.Do()
===== 初始化，只会执行一次 =====
goroutine 2 调用 once.Do()
goroutine 1 调用 once.Do()
goroutine 3 调用 once.Do()
goroutine 4 调用 once.Do()
```
每个协程都会尝试执行初始化操作，但实际上该操作只会执行一次。这在实现某些单例模式时非常有用。

### Pool
可以理解成一个临时对象复用池，用完的对象先放进 Pool，后面需要的时候尝试拿出来重复使用，从而减少频繁创建对象带来的内存分配和 GC 压力。

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	pool := sync.Pool{
		New: func() any {
			fmt.Println("创建新对象")
			return &struct {
				Name string
			}{}
		},
	}

	// 第一次获取
	obj1 := pool.Get().(*struct {
		Name string
	})

	obj1.Name = "Tom"

	fmt.Println("obj1:", obj1.Name)

	// 使用完放回池子
	pool.Put(obj1)

	// 再次获取
	obj2 := pool.Get().(*struct {
		Name string
	})

	fmt.Println("obj2:", obj2.Name)

	pool.Put(obj2)
}
```

运行结果
```shell
go run pool.go 
创建新对象
obj1: Tom
obj2: Tom
```
池化是一种设计模式，例如线程池、内存池和对象池等。上面的示例使用对象池，避免因频繁创建和销毁对象而造成性能损耗。

### Channel
Go 的 channel 比较常规，这里不再详细介绍。

```go
package main

import "fmt"

func main() {
	var ss chan interface{}

	ss = make(chan interface{}, 10)

	ss <- 123
	ss <- "hello"
	ss <- true

	fmt.Println(<-ss)
	fmt.Println(<-ss)
	fmt.Println(<-ss)
}
```
上面的示例展示了一个技巧：将 channel 的元素类型设置为空接口，使其可以接收任意类型的数据。其实 C 中也有类似的技巧，但需要程序员自行负责内存数据的解析，稍有不慎就很容易导致程序崩溃。

channel 被关闭后，已经写入其缓冲区但尚未被消费的数据仍然可以继续接收。缓冲区读空后，再次读取会立即返回该类型的零值，并且 `ok == false`。

```go
package main

import "fmt"

func main() {
	ss := make(chan interface{}, 3)

	ss <- "msg-1"
	ss <- "msg-2"
	ss <- "msg-3"

	close(ss)

	fmt.Println("channel closed")

	// close 后仍然可以消费之前已经发送进去的消息
	for {
		msg, ok := <-ss

		fmt.Printf("msg=%v, ok=%v\n", msg, ok)

		if !ok {
			fmt.Println("channel 已经读完")
			break
		}
	}
}
```

如果 channel 为 nil，读取和写入操作都会阻塞，调用 close 则会引发 panic。

### select

# 2. 并发模式
主协程终止子协程时，一般可以预先设置一个 channel，在需要时通过它向子协程发送信号。

```go
package main

import (
	"fmt"
	"time"
)

func worker(done <-chan struct{}) {
	for {
		select {
		case <-done:
			fmt.Println("worker: 收到取消信号，退出")
			return

		default:
			fmt.Println("worker: working...")
			time.Sleep(500 * time.Millisecond)
		}
	}
}

func main() {
	done := make(chan struct{})

	// 启动子协程
	go worker(done)

	// 父协程运行一段时间
	time.Sleep(2 * time.Second)

	fmt.Println("main: 取消 worker")

	// 父协程通过关闭 channel 广播取消信号
	close(done)

	// demo 中防止 main 立即退出
	time.Sleep(time.Second)

	fmt.Println("main: exit")
}
```
上述代码中的 worker 协程在每次循环时执行 default 分支。主协程关闭 done channel 后，子协程收到取消信号并退出。

Go 虽然有 GC，但在并发场景下，如果设计不当，仍然存在协程泄漏的隐患。

## 2.1 Pipeline

```go
package main

import (
	"fmt"
)

// stage 1:
// 生成数据
func generator(nums ...int) <-chan int {
	out := make(chan int)

	go func() {
		defer close(out)

		for _, n := range nums {
			out <- n
		}
	}()

	return out
}

// stage 2:
// 每个数字乘以 2
func multiply(in <-chan int) <-chan int {
	out := make(chan int)

	go func() {
		defer close(out)

		for n := range in {
			out <- n * 2
		}
	}()

	return out
}

// stage 3:
// 过滤掉小于 10 的数字
func filter(in <-chan int) <-chan int {
	out := make(chan int)

	go func() {
		defer close(out)

		for n := range in {
			if n >= 10 {
				out <- n
			}
		}
	}()

	return out
}

func main() {
	// pipeline:
	//
	// generator
	//     ↓
	// multiply
	//     ↓
	// filter
	//     ↓
	// consumer

	ch1 := generator(1, 3, 5, 7, 10)

	ch2 := multiply(ch1)

	ch3 := filter(ch2)

	for result := range ch3 {
		fmt.Println("result:", result)
	}
}
```
上面的代码中，每个函数都返回一个输出通道，每个 stage 负责完成数据管线中的一部分计算。我认为这也算是一种函数式编程范式。
只要上游 stage 完成一次数据发送，下游 stage 就会开始计算，这是一种流式计算。

使用管线时，一个问题是如何取消整条管线。一项工程原则是让每个 stage 负责关闭自己的输出通道；关闭动作既可以由上游触发，也可以由下游触发。

上游触发：
```go
package main

import (
	"fmt"
)

func generator(nums ...int) <-chan int {
	out := make(chan int)

	go func() {
		defer func() {
			fmt.Println("[generator] close output")
			close(out)
		}()

		for _, n := range nums {
			fmt.Println("[generator] send:", n)
			out <- n
		}

		fmt.Println("[generator] all data sent, exit")
	}()

	return out
}

func multiply(in <-chan int) <-chan int {
	out := make(chan int)

	go func() {
		defer func() {
			fmt.Println("[multiply] close output")
			close(out)
		}()

		for n := range in {
			result := n * 2

			fmt.Println("[multiply] receive:", n)
			fmt.Println("[multiply] send:", result)

			out <- result
		}

		// 能走到这里，说明 in 已经被上游关闭
		fmt.Println("[multiply] input closed, exit")
	}()

	return out
}

func filter(in <-chan int) <-chan int {
	out := make(chan int)

	go func() {
		defer func() {
			fmt.Println("[filter] close output")
			close(out)
		}()

		for n := range in {
			fmt.Println("[filter] receive:", n)

			if n >= 10 {
				fmt.Println("[filter] send:", n)
				out <- n
			} else {
				fmt.Println("[filter] drop:", n)
			}
		}

		fmt.Println("[filter] input closed, exit")
	}()

	return out
}

func main() {
	result := filter(
		multiply(
			generator(1, 3, 5),
		),
	)

	for v := range result {
		fmt.Println("[main] receive:", v)
	}

	fmt.Println("[main] result channel closed")
	fmt.Println("[main] exit")
}

```
在这个示例中，关闭动作由上游生成器完成数据生成后触发，数据管线中的每个 stage 处理完自己的部分后，都会关闭其输出通道。

下面的例子由下游触发：
```go
package main

import (
	"context"
	"fmt"
	"sync"
)

func generator(
	ctx context.Context,
	wg *sync.WaitGroup,
	nums ...int,
) <-chan int {

	out := make(chan int)

	wg.Add(1)

	go func() {
		defer wg.Done()
		defer func() {
			fmt.Println("[generator] close output")
			close(out)
		}()

		for _, n := range nums {
			select {
			case <-ctx.Done():
				fmt.Println("[generator] canceled, exit")
				return

			case out <- n:
				fmt.Println("[generator] send:", n)
			}
		}

		fmt.Println("[generator] all data sent, exit")
	}()

	return out
}

func multiply(
	ctx context.Context,
	wg *sync.WaitGroup,
	in <-chan int,
) <-chan int {

	out := make(chan int)

	wg.Add(1)

	go func() {
		defer wg.Done()
		defer func() {
			fmt.Println("[multiply] close output")
			close(out)
		}()

		for {
			select {
			case <-ctx.Done():
				fmt.Println("[multiply] canceled while waiting input")
				return

			case n, ok := <-in:
				if !ok {
					fmt.Println("[multiply] input closed, exit")
					return
				}

				result := n * 2

				fmt.Println("[multiply] receive:", n)

				select {
				case <-ctx.Done():
					fmt.Println("[multiply] canceled while sending")
					return

				case out <- result:
					fmt.Println("[multiply] send:", result)
				}
			}
		}
	}()

	return out
}

func filter(
	ctx context.Context,
	wg *sync.WaitGroup,
	in <-chan int,
) <-chan int {

	out := make(chan int)

	wg.Add(1)

	go func() {
		defer wg.Done()
		defer func() {
			fmt.Println("[filter] close output")
			close(out)
		}()

		for {
			select {
			case <-ctx.Done():
				fmt.Println("[filter] canceled while waiting input")
				return

			case n, ok := <-in:
				if !ok {
					fmt.Println("[filter] input closed, exit")
					return
				}

				fmt.Println("[filter] receive:", n)

				if n < 10 {
					fmt.Println("[filter] drop:", n)
					continue
				}

				select {
				case <-ctx.Done():
					fmt.Println("[filter] canceled while sending")
					return

				case out <- n:
					fmt.Println("[filter] send:", n)
				}
			}
		}
	}()

	return out
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())

	var wg sync.WaitGroup

	result := filter(
		ctx,
		&wg,
		multiply(
			ctx,
			&wg,
			generator(
				ctx,
				&wg,
				1, 3, 5, 7, 10, 20, 30, 40,
			),
		),
	)

	for v := range result {
		fmt.Println("[main] receive:", v)

		fmt.Println("[main] only need first result")
		fmt.Println("[main] cancel pipeline")

		cancel()
		break
	}

	// 等待所有 pipeline goroutine 真正退出
	wg.Wait()

	fmt.Println("[main] all stages exited")
	fmt.Println("[main] exit")
}

```
下游接收到一条数据后就请求取消整个 pipeline，但主协程仍会等待所有子协程退出，再执行最后的打印并结束。

## 2.2 context
context 并非 Go 独有。例如，C++ 的 Asio、libuv 以及 Python 中都有类似机制，只是具体实现细节存在一定差异。它是并发系统中非常基础的概念。

context 是在 goroutine 或函数调用链之间传递取消信号、超时、截止时间以及请求级数据的标准机制。

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func worker(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("worker: 收到取消信号")
			fmt.Println("reason:", ctx.Err())
			return

		case <-time.After(500 * time.Millisecond):
			fmt.Println("worker: working...")
		}
	}
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())

	go worker(ctx)

	time.Sleep(2 * time.Second)

	fmt.Println("main: cancel")

	cancel()

	time.Sleep(time.Second)
}

```
上述示例中，主协程启动一个子协程。子协程每隔 500 ms 循环一次，收到 context 的结束信号后便退出。


超时示例
```go
package main

import (
	"context"
	"fmt"
	"time"
)

func worker(ctx context.Context) {
	fmt.Println("worker: 开始执行")

	select {
	case <-time.After(5 * time.Second):
		fmt.Println("worker: 任务完成")

	case <-ctx.Done():
		fmt.Println("worker: 被取消")
		fmt.Println("worker: 原因:", ctx.Err())
	}
}

func main() {
	ctx, cancel := context.WithTimeout(
		context.Background(),
		2*time.Second,
	)
	defer cancel()

	worker(ctx)

	fmt.Println("main: exit")
}

```

deadline 示例：

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func worker(ctx context.Context) {
	fmt.Println(
		"worker: 开始:",
		time.Now().Format("15:04:05"),
	)

	select {
	case <-time.After(10 * time.Second):
		fmt.Println("worker: 任务完成")

	case <-ctx.Done():
		fmt.Println(
			"worker: 停止:",
			time.Now().Format("15:04:05"),
		)

		fmt.Println(
			"worker: 原因:",
			ctx.Err(),
		)
	}
}

func main() {
	deadline := time.Now().Add(3 * time.Second)

	fmt.Println(
		"截止时间:",
		deadline.Format("15:04:05"),
	)

	ctx, cancel := context.WithDeadline(
		context.Background(),
		deadline,
	)
	defer cancel()

	worker(ctx)

	fmt.Println("main: exit")
}

```

上面的示例比较简单，但在复杂的并发系统中仍然很容易发生竞态问题。


# 3. 大规模并发与运行时
大规模并发通常应用于分布式系统。这类分布式并发系统经常会出现一些稀奇古怪的 Bug，因此需要设计异常处理机制，配合日志系统上报错误。还需要定义错误的数据结构，包括 ID、机器信息和错误信息等，以便在出现问题时进行诊断。

超时和取消也很常见。例如，系统达到饱和状态，来不及处理任务，导致大量任务超时；或者任务执行到一半时被取消。分布式系统由多台通过网络连接的主机构成，难免会有主机发生故障，因此需要通过心跳机制定时获取各节点的状态，及时发现节点离线等情况。
