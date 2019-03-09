# goroutine中间的通道channel 通信的相关知识
* channel通道的定义
* channel通道的使用
* channel通道的意义
* channel作为一等公民进行参数传递
* 不带缓冲区的channel使用情况
* 带缓冲区的channel使用情况
* channel和系统库sync.WaitGroup()等待多任务运行结束
* 用select进行调度

## 
![Image text](https://github.com/fengjun2016/myGitBook/blob/master/img_cut/goroutine_channel.png)
> 用于不同goroutine之间的消息通信切记 如果在同一个goroutine中间 chan <- 和  <- chan 操作的话 就会出现所有的协程都会deadblock, 也即是一个goroutine发了数据, 必须要有另外一个goroutine来收这个数据, 不然就会出现deadblock问题 代码展示:
```golang
package main

import (
	"fmt"
)

func chanDemo() {
	c := make(chan int)
	c <- 1
	c <- 2
	n := <-c
	fmt.Println(n)
}

func main() {
	chanDemo()
}
运行结果:
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.chanDemo()
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/channel.go:9 +0x59
main.main()
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/channel.go:16 +0x20
exit status 2

exit status 1
```

> 下面对上面的程序进行修改 注意下面的运行结果
```golang
package main

import (
	"fmt"
)

func chanDemo() {
	c := make(chan int)
	go func() {
		n := <-c
		fmt.Println(n)
	}()
	c <- 1
	c <- 2
}

func main() {
	chanDemo()
}
运行结果:
1
	
	fatal error: all goroutines are asleep - deadlock!
	
	goroutine 1 [chan send]:
	main.chanDemo()
		/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/channel.go:14 +0x90
	main.main()
		/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/channel.go:18 +0x20
	exit status 2
	
	exit status 1
// 我们可以发现 打印出了1 但是在打印2的时候 发现还是出了deadblock错误 这是因为在协程里面只收了一次的数据
```

> 再次对上面的程序作出一些修改, 添加一个for无限循环 代码展示如下所示:
```golang
package main

import (
	"fmt"
	_ "time"
)

func chanDemo() {
	c := make(chan int)
	go func() {
		for {
			n := <-c
			fmt.Println(n)
		}
	}()
	c <- 1
	c <- 2
	//time.Sleep(time.Millisecond)
}

func main() {
	chanDemo()
}
运行结果:
1
2
```

## channel作为一等功明进行参数传递的代码演示:
```golang
package main 

import (
	"fmt"
)

func worker(c chan int) {
	for {
		n := <- c 
		fmt.Println(n)
	}
}

func main() {
	c := make(chan int)
	go worker(c)
	c <- 1
	c <- 2
}
运行结果:
1
2
```

```golang
package main

import (
	"fmt"
	"time"
)

func worker(id int, c chan int) {
	for {
		fmt.Printf("Worker %d received %c\n", id, <-c)
	}
}

func chanDemo() {
	var channels [10]chan int
	for i := 0; i < 10; i++ {
		channels[i] = make(chan int)
		go worker(i, channels[i])
	}
	for i := 0; i < 10; i++ {
		channels[i] <- 'a' + i
	}
	time.Sleep(time.Millisecond)
}

func main() {
	chanDemo()
}
运行结果:
Worker 4 received e
Worker 6 received g
Worker 1 received b
Worker 3 received d
Worker 7 received h
Worker 5 received f
Worker 2 received c
Worker 8 received i
Worker 9 received j
Worker 0 received a
```

```golang
package main

import (
	"fmt"
	_ "time"
)

func worker(id int) chan int {
	c := make(chan int)
	go func() {
		for {
			fmt.Printf("Worker %d received %c\n", id, <-c)
		}
	}()
	return c
}

func chanDemo() {
	var channels [10]chan int
	for i := 0; i < 10; i++ {
		channels[i] = worker(i)
	}
	for i := 0; i < 10; i++ {
		channels[i] <- 'a' + i
	}
	for i := 0; i < 10; i++ {
		channels[i] <- 'A' + i
	}
	//time.Sleep(time.Millisecond)
}

func main() {
	chanDemo()
}
运行结果:
Worker 0 received a
Worker 6 received g
Worker 4 received e
Worker 5 received f
Worker 1 received b
Worker 1 received B
Worker 3 received d
Worker 2 received c
Worker 2 received C
Worker 8 received i
Worker 7 received h
Worker 0 received A
Worker 8 received I
Worker 6 received G
Worker 3 received D
Worker 4 received E
Worker 5 received F
Worker 7 received H
Worker 9 received j
Worker 9 received J
```

## 不带缓冲区的channel使用介绍 特别是阻塞情况的分析，发送动作和接收动作必须同时出现在不同的goroutine中时才不会出现deadblock情况

## 带缓冲区的channel使用介绍 因为普通不带缓冲区的channel会经常需要在接收数据和传数据两个goroutine之间切换才行，不然会发生阻塞，所以会很耗开销,从而这就催生出了带缓冲区的bufferedchannel的使用 在缓冲区填满之前, 发送动作不会出现deadblock, 只有在通道里面没有要接收的值时, 接收动作才会被阻塞住

* 注意如果有协程在收数据的话，那么缓冲通道就会出现新的空闲通道，则写数据的还可以往里面继续写且不会发生deadblock死锁


```golang
package main

import (
	"fmt"
	"time"
)

func worker(id int, c chan int) {
	for {
		fmt.Printf("Worker %d received %c\n", id, <-c)
	}
}
func createWorker(id int) chan int {
	c := make(chan int)
	go worker(id, c)
	return c
}

func chanDemo() {
	var channels [10]chan int
	for i := 0; i < 10; i++ {
		channels[i] = createWorker(i)
	}
	for i := 0; i < 10; i++ {
		channels[i] <- 'a' + i
	}
	for i := 0; i < 10; i++ {
		channels[i] <- 'A' + i
	}
	time.Sleep(time.Millisecond)
}

func bufferedChannel() {
	c := make(chan int, 3)
	//go worker(0, c)
	c <- 'a'
	c <- 'b'
	c <- 'c'
	c <- 'd'
	time.Sleep(time.Millisecond)
}
func main() {
	// chanDemo()
	bufferedChannel()
}

//这里在没有协程来收这个数据的时候 你会发现往缓冲通道里面写入四个数据的时候，会发生deadblock asleep等待 因为超过了它自己缓冲通道的个数
运行结果:
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.bufferedChannel()
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/channel.go:39 +0xa7
main.main()
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/channel.go:44 +0x20
exit status 2

exit status 1
```

```golang
package main

import (
	"fmt"
	"time"
)

func worker(id int, c chan int) {
	for {
		fmt.Printf("Worker %d received %c\n", id, <-c)
	}
}
func createWorker(id int) chan int {
	c := make(chan int)
	go worker(id, c)
	return c
}

func chanDemo() {
	var channels [10]chan int
	for i := 0; i < 10; i++ {
		channels[i] = createWorker(i)
	}
	for i := 0; i < 10; i++ {
		channels[i] <- 'a' + i
	}
	for i := 0; i < 10; i++ {
		channels[i] <- 'A' + i
	}
	time.Sleep(time.Millisecond)
}

func bufferedChannel() {
	c := make(chan int, 3)
	//go worker(0, c)
	c <- 'a'
	c <- 'b'
	c <- 'c'
	// c <- 'd'
	time.Sleep(time.Millisecond)   //这个等待是为了等待上面的输出打印结束
}
func main() {
	// chanDemo()
	bufferedChannel()
}
运行结果:
发现没有任何输出 也没有任何报错信息 也没有发生死锁
```


* 下面加上输出以及等待打印操作
```golang
package main

import (
	"fmt"
	"time"
)

func worker(id int, c chan int) {
	for {
		fmt.Printf("Worker %d received %c\n", id, <-c)
	}
}
func createWorker(id int) chan int {
	c := make(chan int)
	go worker(id, c)
	return c
}

func chanDemo() {
	var channels [10]chan int
	for i := 0; i < 10; i++ {
		channels[i] = createWorker(i)
	}
	for i := 0; i < 10; i++ {
		channels[i] <- 'a' + i
	}
	for i := 0; i < 10; i++ {
		channels[i] <- 'A' + i
	}
	time.Sleep(time.Millisecond)  //为了等待打印结束
}

func bufferedChannel() {
	c := make(chan int, 3)
	go worker(0, c)
	c <- 'a'
	c <- 'b'
	c <- 'c'
	c <- 'd'
	time.Sleep(time.Millisecond)  //为了等待数据打印结束
}
func main() {
	// chanDemo()
	bufferedChannel()
}
运行结果:
Worker 0 received a
Worker 0 received b
Worker 0 received c
Worker 0 received d
```

## 使用channel和sync.WaitGroup()来等待多任务运行结束，上面是用timne.Sleep()来等待数据打印结束，但是这种做法还是不完美，下面来采用channel来完成这个等待事件

```golang
package main

import (
	"fmt"
)

func doWorker(id int, c chan int, done chan bool) {
	for {
		fmt.Printf("Worker %d received %c\n", id, <-c)
		done <- true
	}
}

type worker struct {
	in   chan int
	done chan bool
}

func createWorker(id int) worker {
	w := worker{
		in:   make(chan int),
		done: make(chan bool),
	}
	go doWorker(id, w.in, w.done)
	return w
}

func chanDemo() {
	var workers [10]worker
	for i := 0; i < 10; i++ {
		workers[i] = createWorker(i)
	}
	for i := 0; i < 10; i++ {
		workers[i].in <- 'a' + i
		<-workers[i].done //用于等待打印数据结束
	}
	for i := 0; i < 10; i++ {
		workers[i].in <- 'A' + i
		<-workers[i].done //用于等待打印数据结束
	}
	//time.Sleep(time.Millisecond) //这里加上等待的原因是因为数据可能都传递过去了 但是还没有来得及打印程序就退出了
}

func main() {
	chanDemo()
}
运行结果:
Worker 0 received a
Worker 1 received b
Worker 2 received c
Worker 3 received d
Worker 4 received e
Worker 5 received f
Worker 6 received g
Worker 7 received h
Worker 8 received i
Worker 9 received j
Worker 0 received A
Worker 1 received B
Worker 2 received C
Worker 3 received D
Worker 4 received E
Worker 5 received F
Worker 6 received G
Worker 7 received H
Worker 8 received I
Worker 9 received J

//从上面的运行结果可以看出 这样每一个都等待打印结束done的时候，会造成顺序打印，并不是很好的履行了并发的随意性，所以下面改造成最后一起接收done
```

```golang
package main

import (
	"fmt"
)

func doWorker(id int, c chan int, done chan bool) {
	for {
		fmt.Printf("Worker %d received %c\n", id, <-c)
		done <- true
	}
}

type worker struct {
	in   chan int
	done chan bool
}

func createWorker(id int) worker {
	w := worker{
		in:   make(chan int),
		done: make(chan bool),
	}
	go doWorker(id, w.in, w.done)
	return w
}

func chanDemo() {
	var workers [10]worker
	for i := 0; i < 10; i++ {
		workers[i] = createWorker(i)
	}
	for i, worker := range workers {
		worker.in <- 'a' + i
		// <-workers[i].done
	}
	for i, worker := range workers {
		worker.in <- 'A' + i
		// <-workers[i].done //用于告诉外界打印结束
	}
	//time.Sleep(time.Millisecond) //这里加上等待的原因是因为数据可能都传递过去了 但是还没有来得及打印程序就退出了
	for _, worker := range workers {
		<-worker.done //两次是因为打印了两次
		<-worker.done
	}
}

func main() {
	chanDemo()
}
运行结果:
Worker 4 received e
Worker 0 received a
Worker 6 received g
Worker 3 received d
Worker 5 received f
Worker 2 received c
Worker 7 received h
Worker 9 received j
Worker 8 received i
Worker 1 received b

fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.chanDemo()
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:38 +0x14e
main.main()
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:49 +0x20

goroutine 5 [chan send]:
main.doWorker(0x0, 0xc420070060, 0xc4200700c0)
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:10 +0x124
created by main.createWorker
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:24 +0x98

goroutine 6 [chan send]:
main.doWorker(0x1, 0xc420070120, 0xc420070180)
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:10 +0x124
created by main.createWorker
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:24 +0x98

goroutine 7 [chan send]:
main.doWorker(0x2, 0xc4200701e0, 0xc420070240)
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:10 +0x124
created by main.createWorker
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:24 +0x98

goroutine 8 [chan send]:
main.doWorker(0x3, 0xc4200702a0, 0xc420070300)
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:10 +0x124
created by main.createWorker
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:24 +0x98

goroutine 9 [chan send]:
main.doWorker(0x4, 0xc420070360, 0xc4200703c0)
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:10 +0x124
created by main.createWorker
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:24 +0x98

goroutine 10 [chan send]:
main.doWorker(0x5, 0xc420070420, 0xc420070480)
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:10 +0x124
created by main.createWorker
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:24 +0x98

goroutine 11 [chan send]:
main.doWorker(0x6, 0xc4200704e0, 0xc420070540)
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:10 +0x124
created by main.createWorker
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:24 +0x98

goroutine 12 [chan send]:
main.doWorker(0x7, 0xc4200705a0, 0xc420070600)
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:10 +0x124
created by main.createWorker
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:24 +0x98

goroutine 13 [chan send]:
main.doWorker(0x8, 0xc420070660, 0xc4200706c0)
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:10 +0x124
created by main.createWorker
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:24 +0x98

goroutine 14 [chan send]:
main.doWorker(0x9, 0xc420070720, 0xc420070780)
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:10 +0x124
created by main.createWorker
	/Users/fengjun/workspace/go_workspace/src/go_mooc/channel/done/done.go:24 +0x98
exit status 2

exit status 1
```

> 上面再写入大写的时候发生了deadblock asleep 循环等待的问题 这是为什么呢? 

> 这是因为10个worker的done在小写字母打印结束之后，还没有人接收done的同时，又开始往worker的done里面发数据，所以就造成了deadblock 死锁的发生,下面换一种解决办法 不把两次done的接收放在同一个for循环里面解决，而是小写全部写入成功之后，下面看代码演示以及运行结果:

```golang
package main

import (
	"fmt"
)

func doWorker(id int, c chan int, done chan bool) {
	for {
		fmt.Printf("Worker %d received %c\n", id, <-c)
		done <- true
	}
}

type worker struct {
	in   chan int
	done chan bool
}

func createWorker(id int) worker {
	w := worker{
		in:   make(chan int),
		done: make(chan bool),
	}
	go doWorker(id, w.in, w.done)
	return w
}

func chanDemo() {
	var workers [10]worker
	for i := 0; i < 10; i++ {
		workers[i] = createWorker(i)
	}

	for i, worker := range workers {
		worker.in <- 'a' + i
	}

	for _, worker := range workers {
		<-worker.done //等待外界告知打印结束通知
	}

	for i, worker := range workers {
		worker.in <- 'A' + i
	}

	for _, worker := range workers {
		<-worker.done //等待外界告知打印结束通知
	}
}

func main() {
	chanDemo()
}
运行结果:
Worker 8 received i
Worker 2 received c
Worker 5 received f
Worker 3 received d
Worker 6 received g
Worker 4 received e
Worker 7 received h
Worker 0 received a
Worker 1 received b
Worker 9 received j
Worker 9 received J
Worker 5 received F
Worker 0 received A
Worker 1 received B
Worker 6 received G
Worker 7 received H
Worker 8 received I
Worker 3 received D
Worker 4 received E
Worker 2 received C
```
### 使用系统sync.WaitGroup方法来辅助等待多任务运行结束的模型 一定要保证同一个多任务使用同一个WaitGroup的引用 切记
```golang
package main

import (
	"fmt"
	"sync"
)

func doWorker(id int, w worker) {
	for {
		fmt.Printf("Worker %d received %c\n", id, <-w.in)
		w.done()
	}
}

type worker struct {
	in   chan int
	done func() //利用golang里面函数作为一等公民的角色来
}

func createWorker(id int, wg *sync.WaitGroup) worker {
	w := worker{
		in: make(chan int),
		done: func() {
			wg.Done()
		},
	}
	go doWorker(id, w)
	return w
}

func chanWithWaitGroupDemo() {
	var workers [10]worker
	var wg sync.WaitGroup

	for i := 0; i < 10; i++ {
		workers[i] = createWorker(i, &wg)
	}

	wg.Add(20)
	for i, worker := range workers {
		worker.in <- 'a' + i
	}

	for i, worker := range workers {
		worker.in <- 'A' + i
	}
	wg.Wait() //等待任务
}

func main() {
	chanWithWaitGroupDemo()
}
运行结果:
Worker 1 received b
Worker 2 received c
Worker 3 received d
Worker 5 received f
Worker 6 received g
Worker 0 received a
Worker 9 received j
Worker 7 received h
Worker 0 received A
Worker 8 received i
Worker 3 received D
Worker 1 received B
Worker 4 received e
Worker 4 received E
Worker 6 received G
Worker 9 received J
Worker 7 received H
Worker 8 received I
Worker 2 received C
Worker 5 received F
```

### 用select进行调度
> 一般在使用channel的时候都是阻塞式的，如果想要使用非阻塞式的话，就要使用select+default 例如下面这样的一段代码演示如下

```golang
package main

import (
	"fmt"
)

func main() {
	var c1, c2 chan int // c1 and c2 = nil
	for {
		select {
		case n := <-c1:
			fmt.Println("Received from c1:", n)
		case n := <-c2:
			fmt.Println("Received from c2:", n)
		default:
			fmt.Println("No value received")
		}
	}
}
//没有发生deadblock
```

* 这里我们做一个实验 当消费数据和发送数据速度不匹配时，在下面这种情况下会漏打印很多数据
```golang
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func generator() chan int {
	out := make(chan int)
	go func() {
		i := 0
		for {
			time.Sleep(time.Duration(rand.Intn(1500)) * time.Millisecond)
			out <- i
			i++
		}
	}()
	return out
}

func worker(id int, c chan int) {
	for n := range c {
		time.Sleep(5 * time.Second)
		fmt.Printf("Worker %d received %d\n", id, n)
	}
}

func createWorker(id int) chan int {
	c := make(chan int)
	go worker(id, c)
	return c
}

func main() {
	var c1, c2 = generator(), generator()
	worker := createWorker(0)
	n := 0
	hasValue := false
	for {
		var activeWorker chan int
		if hasValue {
			activeWorker = worker
		}
		select {
		case n = <-c1:
			hasValue = true
		case n = <-c2:
			hasValue = true
		case activeWorker <- n:
			hasValue = false
		}
	}
}
运行结果:
Worker 0 received 0
Worker 0 received 7
Worker 0 received 12
```
* 下面对上面的代码进行优化， 使用一个slice将这些数据存储起来
```golang
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func generator() chan int {
	out := make(chan int)
	go func() {
		i := 0
		for {
			time.Sleep(time.Duration(rand.Intn(1500)) * time.Millisecond)
			out <- i
			i++
		}
	}()
	return out
}

func worker(id int, c chan int) {
	for n := range c {
		time.Sleep(time.Second)
		fmt.Printf("Worker %d received %d\n", id, n)
	}
}

func createWorker(id int) chan int {
	c := make(chan int)
	go worker(id, c)
	return c
}

func main() {
	var c1, c2 = generator(), generator()
	worker := createWorker(0)
	n := 0
	var values []int
	for {
		var activeWorker chan int
		var activeValue int
		if len(values) > 0 {
			activeWorker = worker
			activeValue = values[0]
		}
		select {
		case n = <-c1:
			values = append(values, n)
		case n = <-c2:
			values = append(values, n)
		case activeWorker <- activeValue:
			values = values[1:]
		}
	}
}
运行结果:
Worker 0 received 0
Worker 0 received 0
Worker 0 received 1
Worker 0 received 1
Worker 0 received 2
Worker 0 received 2
Worker 0 received 3
Worker 0 received 3
Worker 0 received 4
Worker 0 received 5
Worker 0 received 4
Worker 0 received 5
Worker 0 received 6
Worker 0 received 6
Worker 0 received 7
Worker 0 received 7
Worker 0 received 8
Worker 0 received 9
Worker 0 received 8
Worker 0 received 10
Worker 0 received 9
Worker 0 received 11
Worker 0 received 10
Worker 0 received 11
Worker 0 received 12
Worker 0 received 13
Worker 0 received 14
Worker 0 received 12
Worker 0 received 13
Worker 0 received 14
Worker 0 received 15
Worker 0 received 15
Worker 0 received 16
Worker 0 received 17
Worker 0 received 16
Worker 0 received 18
Worker 0 received 17
Worker 0 received 19
Worker 0 received 18
Worker 0 received 20
Worker 0 received 19
Worker 0 received 21
Worker 0 received 22
Worker 0 received 20
Worker 0 received 23
Worker 0 received 21
Worker 0 received 24
Worker 0 received 22
Worker 0 received 25
Worker 0 received 23
Worker 0 received 26
Worker 0 received 27
Worker 0 received 24
Worker 0 received 28
Worker 0 received 29
Worker 0 received 25
Worker 0 received 26
Worker 0 received 30
Worker 0 received 27
Worker 0 received 31
Worker 0 received 32
Worker 0 received 33
Worker 0 received 28
Worker 0 received 34
Worker 0 received 29
Worker 0 received 35
Worker 0 received 30
Worker 0 received 36
Worker 0 received 31
Worker 0 received 37
Worker 0 received 32
Worker 0 received 33
Worker 0 received 38
Worker 0 received 34
Worker 0 received 39
Worker 0 received 35
Worker 0 received 40
Worker 0 received 36
Worker 0 received 41
Worker 0 received 42
Worker 0 received 37
Worker 0 received 43
Worker 0 received 44
Worker 0 received 38
Worker 0 received 39
Worker 0 received 45
Worker 0 received 46
Worker 0 received 40
Worker 0 received 41
Worker 0 received 47
Worker 0 received 42
Worker 0 received 43
Worker 0 received 48
Worker 0 received 44
Worker 0 received 45
Worker 0 received 49
Worker 0 received 46
Worker 0 received 47
Worker 0 received 48
Worker 0 received 50
Worker 0 received 49
Worker 0 received 50
Worker 0 received 51
Worker 0 received 51
Worker 0 received 52
Worker 0 received 52
Worker 0 received 53
Worker 0 received 54
Worker 0 received 53
Worker 0 received 55
Worker 0 received 54
Worker 0 received 55
Worker 0 received 56
Worker 0 received 57
Worker 0 received 56
Worker 0 received 58
Worker 0 received 57
Worker 0 received 59
Worker 0 received 58
Worker 0 received 60
Worker 0 received 59
Worker 0 received 61
Worker 0 received 62
Worker 0 received 60
Worker 0 received 61
Worker 0 received 63
Worker 0 received 64
Worker 0 received 62
Worker 0 received 65
Worker 0 received 66
Worker 0 received 63
Worker 0 received 67
```
* 使用time.After控制程序运行10秒便退出整个程序的运行

```golang
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func generator() chan int {
	out := make(chan int)
	go func() {
		i := 0
		for {
			time.Sleep(time.Duration(rand.Intn(1500)) * time.Millisecond)
			out <- i
			i++
		}
	}()
	return out
}

func worker(id int, c chan int) {
	for n := range c {
		time.Sleep(time.Second)
		fmt.Printf("Worker %d received %d\n", id, n)
	}
}

func createWorker(id int) chan int {
	c := make(chan int)
	go worker(id, c)
	return c
}

func main() {
	var c1, c2 = generator(), generator()
	worker := createWorker(0)
	n := 0
	var values []int
	tm := time.After(10 * time.Second)
	for {
		var activeWorker chan int
		var activeValue int
		if len(values) > 0 {
			activeWorker = worker
			activeValue = values[0]
		}
		select {
		case n = <-c1:
			values = append(values, n)
		case n = <-c2:
			values = append(values, n)
		case activeWorker <- activeValue:
			values = values[1:]
		case <-tm: //控制程序运行的时间
			fmt.Println("bye")
			return
		}
	}
}
运行结果:
Worker 0 received 0
Worker 0 received 0
Worker 0 received 1
Worker 0 received 1
Worker 0 received 2
Worker 0 received 2
Worker 0 received 3
Worker 0 received 3
Worker 0 received 4
bye
```

* 检测每两次select的时间间隔 用于检测每两次收数据的时间间隔

> 下面代码里面两次的time.After返回的都是channel,但是注意放的位置的不同，导致其意义不一样，上面一个是记录主程序开始运行的时间10秒以后，下面第2个则是记录两次selet到相同位置还没有发送数据过来的时间间隔
```golang
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func generator() chan int {
	out := make(chan int)
	go func() {
		i := 0
		for {
			time.Sleep(time.Duration(rand.Intn(1500)) * time.Millisecond)
			out <- i
			i++
		}
	}()
	return out
}

func worker(id int, c chan int) {
	for n := range c {
		time.Sleep(time.Second)
		fmt.Printf("Worker %d received %d\n", id, n)
	}
}

func createWorker(id int) chan int {
	c := make(chan int)
	go worker(id, c)
	return c
}

func main() {
	var c1, c2 = generator(), generator()
	worker := createWorker(0)
	n := 0
	var values []int
	tm := time.After(10 * time.Second)
	for {
		var activeWorker chan int
		var activeValue int
		if len(values) > 0 {
			activeWorker = worker
			activeValue = values[0]
		}
		select {
		case n = <-c1:
			values = append(values, n)
		case n = <-c2:
			values = append(values, n)
		case <-time.After(800 * time.Millisecond):
			fmt.Println("timeout")
		case activeWorker <- activeValue:
			values = values[1:]
		case <-tm: //控制程序运行的时间
			fmt.Println("bye")
			return
		}
	}
}

运行结果:
Worker 0 received 0
Worker 0 received 0
Worker 0 received 1
Worker 0 received 1
Worker 0 received 2
Worker 0 received 2
timeout
Worker 0 received 3
timeout
Worker 0 received 3
timeout
Worker 0 received 4
bye
```

* time.Tick()定时器的作用 

```golang
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func generator() chan int {
	out := make(chan int)
	go func() {
		i := 0
		for {
			time.Sleep(time.Duration(rand.Intn(1500)) * time.Millisecond)
			out <- i
			i++
		}
	}()
	return out
}

func worker(id int, c chan int) {
	for n := range c {
		time.Sleep(time.Second)
		fmt.Printf("Worker %d received %d\n", id, n)
	}
}

func createWorker(id int) chan int {
	c := make(chan int)
	go worker(id, c)
	return c
}

func main() {
	var c1, c2 = generator(), generator()
	worker := createWorker(0)
	n := 0
	var values []int
	tm := time.After(10 * time.Second)
	tick := time.Tick(time.Second)
	for {
		var activeWorker chan int
		var activeValue int
		if len(values) > 0 {
			activeWorker = worker
			activeValue = values[0]
		}
		select {
		case n = <-c1:
			values = append(values, n)
		case n = <-c2:
			values = append(values, n)
		case <-time.After(800 * time.Millisecond):
			fmt.Println("timeout")
		case <-tick:
			//检测数据积压长度 由于发和接存在速度不一致
			fmt.Printf("queue len = %d\n", len(values))
		case activeWorker <- activeValue:
			values = values[1:]
		case <-tm: //控制程序运行的时间
			fmt.Println("bye")
			return
		}
	}
}
运行结果:
queue len = 3
Worker 0 received 0
queue len = 5
Worker 0 received 0
queue len = 5
Worker 0 received 1
queue len = 10
Worker 0 received 1
queue len = 10
Worker 0 received 2
queue len = 11
Worker 0 received 2
queue len = 12
Worker 0 received 3
queue len = 13
Worker 0 received 3
queue len = 14
Worker 0 received 4
bye
```
## 关于值为nil的未初始化的channel在select中是一直会被阻塞的的注意点




