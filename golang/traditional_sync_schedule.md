# 关于golang中的传统同步机制的初步介绍
* WaitGroup
* Mutex 互斥量
* Cond
* 传统同步机制很少用到 因为是用共享内存来实现通信的 所以建议是使用channel通道来实现的

## Mutex互斥量的使用
```golang
package main

import (
	"fmt"
	"time"
)

type atomicInt int

func (a *atomicInt) increment() {
	*a++
}

func (a *atomicInt) get() int {
	return int(*a)
}

func main() {
	var a atomicInt
	a.increment()
	go func() {
		a.increment()
	}()
	time.Sleep(3 * time.Millisecond)
	fmt.Println(a)
}
运行结果:
go run -race atomic.go  //检测数据冲突
2

==================
WARNING: DATA RACE
Read at 0x00c42007a008 by main goroutine:
  main.main()
      /Users/fengjun/workspace/go_workspace/src/go_mooc/atomic/atomic.go:25 +0xd0

Previous write at 0x00c42007a008 by goroutine 6:
  main.main.func1()
      /Users/fengjun/workspace/go_workspace/src/go_mooc/atomic/atomic.go:11 +0x54

Goroutine 6 (finished) created at:
  main.main()
      /Users/fengjun/workspace/go_workspace/src/go_mooc/atomic/atomic.go:21 +0xb2
==================
Found 1 data race(s)
exit status 66

exit status 1
```
> 下面对上面的代码做相应的修改和处理 使用系统提供的同步锁机制对变量访问进行加锁
```golang
package main

import (
	"fmt"
	"sync"
	"time"
)

type atomicInt struct {
	value int
	lock  sync.Mutex
}

func (a *atomicInt) increment() {
	a.lock.Lock()
	defer a.lock.Unlock()
	a.value++
}

func (a *atomicInt) get() int {
	a.lock.Lock()
	defer a.lock.Unlock()
	return int(a.value)
}

func main() {
	var a atomicInt
	a.increment()
	go func() {
		a.increment()
	}()
	time.Sleep(3 * time.Millisecond)
	fmt.Println(a.get())
}
运行结果: 没有发生数据冲突
2
```

## 如果想要在某一块代码块里面加上sync.Mutex锁操作的话 只需要写上匿名函数将这块代码包起来即可 代码示例:

```golang
package main

import (
	"fmt"
	"sync"
	"time"
)

type atomicInt struct {
	value int
	lock  sync.Mutex
}

func (a *atomicInt) increment() {
	fmt.Println("safe increment")
	func() {
		a.lock.Lock()
		defer a.lock.Unlock()
		a.value++
	}()
}

func (a *atomicInt) get() int {
	a.lock.Lock()
	defer a.lock.Unlock()
	return int(a.value)
}

func main() {
	var a atomicInt
	a.increment()
	go func() {
		a.increment()
	}()
	time.Sleep(3 * time.Millisecond)
	fmt.Println(a.get())
}
```
