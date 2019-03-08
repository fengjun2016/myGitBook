# 关于goroutine和channel使用时需要注意的事项
* 协程协程Coroutine定义
* goroutine的定义
* goroutine可能的切换点
* channel使用作为通信的注意点

## 协程Coroutine定义
> * 轻量级"线程"
  * 非抢占式多任务处理,由协程主动交出控制权
  * 编译器/解释权/虚拟机层面的多任务
  * 多个协程可能在一个或多个线程上运行

## goroutine的定义
> * 任何函数只需加上go就能送给调度器运行
  * 不需要在定义时区分是否是异步函数
  * 调度器在合适的点进行切换
  * 使用-race来检测数据访问冲突
  * 多个协程可能只是隐射到同一个线程当中

  
### 代码展示以及实验

> 没有goroutine 就会一直无限循环执行i=0的时候
```golang
package main

import (
	"fmt"
)

func main() {
	for i := 0; i < 10; i++ {
		func(i int) {
			for {
				fmt.Printf("Hello from goroutine %d\n", i)
			}
		}(i)
	}
}
运行结果:
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
```

> 加上关键字go以后 相当于在main里面开了10个协程 主程序和协程也一直在并发往下执行 当main程序执行完了以后整个程序就会退出
```golang
package main

import (
	"fmt"
)

func main() {
	for i := 0; i < 10; i++ {
		go func(i int) {
			for {
				fmt.Printf("Hello from goroutine %d\n", i)
			}
		}(i)
	}
}
运行结果:

什么也没有输出 因为并发执行 协程还来不及打印的时候 main程序执行完并且退出
```

> 所以在调试阶段 我们一般使用time.Sleep()让主程序稍微等待一毫秒 发现程序有输出到i=5的时候 自动退出整个程序的运行
```golang
package main

import (
	"fmt"
	"time"
)

func main() {
	for i := 0; i < 10; i++ {
		go func(i int) {
			for {
				fmt.Printf("Hello from goroutine %d\n", i)
			}
		}(i)
	}
	time.Sleep(time.Millisecond)
}

运行结果:
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 2
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 1
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 4
Hello from goroutine 4
Hello from goroutine 4
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 3
Hello from goroutine 3
Hello from goroutine 3
Hello from goroutine 3
Hello from goroutine 3
Hello from goroutine 3
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 3
Hello from goroutine 3
Hello from goroutine 3
Hello from goroutine 3
Hello from goroutine 3
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 7
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 9
Hello from goroutine 4
Hello from goroutine 4
Hello from goroutine 4
Hello from goroutine 4
Hello from goroutine 3
Hello from goroutine 3
Hello from goroutine 3
Hello from goroutine 3
Hello from goroutine 3
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 8
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 2
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 7
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 6
Hello from goroutine 3
Hello from goroutine 3
Hello from goroutine 3
Hello from goroutine 3
Hello from goroutine 3
Hello from goroutine 3
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 0
Hello from goroutine 4
Hello from goroutine 4
Hello from goroutine 4
Hello from goroutine 4
Hello from goroutine 4
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 4
Hello from goroutine 4
Hello from goroutine 4
Hello from goroutine 4
Hello from goroutine 4
Hello from goroutine 4
Hello from goroutine 4
Hello from goroutine 4
Hello from goroutine 4
Hello from goroutine 4
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
Hello from goroutine 5
```

> 关于cpu占有主动权的出让的实验 goroutine无法自己交出控制权, 因为没有fmt等io打印操作,而且go又是非抢占式,而且main自己本身也是一个goroutine,无法得到控制权,从而导致整个程序无法退出执行

```golang
package main 

import (
	"fmt"
)

func main() {
	var a [10]int 
	for i := 0; i < 10; i++ {
		go func(i int) {
			a[i]++	
		}(i)
	}
	time.Sleep(time.Millisecond)
	fmt.Println(a)	
}
运行结果:
处于gorourinte死循环中
```

> 除了io操作可以主动让系统交出控制权之外，还可以手动交出控制权使用runtime.Gosched(), 让其他goroutine也有机会运行，同时控制权后面也有机会回到自己手上
```golang
package main 

import (
	"fmt"
	"time"
	"runtime"
)

func main() {
	var a [10]int 
	for i := 0; i < 10; i++ {
		go func(i int) {
			a[i]++	
			runtime.Gosched()
		}(i)
	}
	time.Sleep(time.Millisecond)
	fmt.Println(a)	
}
运行结果:
[1 1 1 1 1 1 1 1 1 1]
//其实这个例子里面也有race condition这个问题 虽然go run goroutine.go检测不出什么问题 但是👇:
go run -race goroutine.go 
运行结果:
	[1 1 1 1 1 1 1 1 1 1]
	
	==================
	WARNING: DATA RACE
	Read at 0x00c42001c0f0 by main goroutine:
	  main.main()
	      /Users/fengjun/workspace/go_workspace/src/go_mooc/goroutine/goroutine.go:18 +0xfe
	
	Previous write at 0x00c42001c0f0 by goroutine 6:
	  main.main.func1()
	      /Users/fengjun/workspace/go_workspace/src/go_mooc/goroutine/goroutine.go:13 +0x74
	
	Goroutine 6 (finished) created at:
	  main.main()
	      /Users/fengjun/workspace/go_workspace/src/go_mooc/goroutine/goroutine.go:12 +0xc6
	==================
	Found 1 data race(s)
	exit status 66
	
	exit status 1


//这里出现是由于 main在打印a 而上面的goroutine协程却在并发的修改a 这个问题应该由channel通道来解决
```

#### 下面来做一个实验 代码如下所示: 函数式编程的闭包
```golang
package main

import (
	"fmt"
	"runtime"
	"time"
)

func main() {
	var a [10]int
	for i := 0; i < 10; i++ {
		go func() { //闭包里面数据访问冲突 race condition!
			a[i]++
			runtime.Gosched()
		}()
	}
	time.Sleep(time.Millisecond)
	fmt.Println(a)
}

go run -race goroutine.go 

运行结果:
	==================
	WARNING: DATA RACE
	Read at 0x00c420082008 by goroutine 6:
	  main.main.func1()
	      /Users/fengjun/workspace/go_workspace/src/go_mooc/goroutine/goroutine.go:13 +0x3f
	
	Previous write at 0x00c420082008 by main goroutine:
	  main.main()
	      /Users/fengjun/workspace/go_workspace/src/go_mooc/goroutine/goroutine.go:11 +0x127
	
	Goroutine 6 (running) created at:
	  main.main()
	      /Users/fengjun/workspace/go_workspace/src/go_mooc/goroutine/goroutine.go:12 +0xf7
	==================
	==================
	WARNING: DATA RACE
	Read at 0x00c420094010 by goroutine 8:
	  main.main.func1()
	      /Users/fengjun/workspace/go_workspace/src/go_mooc/goroutine/goroutine.go:13 +0x63
	
	Previous write at 0x00c420094010 by goroutine 7:
	  main.main.func1()
	      /Users/fengjun/workspace/go_workspace/src/go_mooc/goroutine/goroutine.go:13 +0xab
	
	Goroutine 8 (running) created at:
	  main.main()
	      /Users/fengjun/workspace/go_workspace/src/go_mooc/goroutine/goroutine.go:12 +0xf7
	
	Goroutine 7 (finished) created at:
	  main.main()
	      /Users/fengjun/workspace/go_workspace/src/go_mooc/goroutine/goroutine.go:12 +0xf7
	==================
	panic: runtime error: index out of range
	
	goroutine 28 [running]:
	main.main.func1(0xc420094000, 0xc420082008)
		/Users/fengjun/workspace/go_workspace/src/go_mooc/goroutine/goroutine.go:13 +0xe4
	created by main.main
		/Users/fengjun/workspace/go_workspace/src/go_mooc/goroutine/goroutine.go:12 +0xf8
	exit status 2
	
	exit status 1
//运行时可以采用命令 go run -race goroutine.go 来跟踪这个数据冲突错误
//这是因为并发main程序在写这个i的时候、goroutine协程同时也在读这个i 从而导致了数据冲突 
```

## goroutine可能的切换点
> * I/O, select 比如打印 fmt.Printf()等待函数的使用的时候
  * channel 作为协程通信的时候
  * 等待锁
  * 函数调用
  * runtime.Gosched()
  * 上面所列举的只是参考，不能保证切换，不能保证在其他地方不切换