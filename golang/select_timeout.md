# 使用select实现超时等待的问题场景
```golang
package main 

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan int)
	quit := make(chan bool)

	go func() {
		for {
			select {
			case num := <- ch:
				fmt.Println("num := ", num)
			case <- time.After(3*time.Second):
				quit <- true
			}
		}
	}()

	for i := 0; i < 8; i++ {
		ch <- i
	}
	<- quit
	fmt.Println("主协程已退出")
}
运行结果:
num =  0
num =  1
num =  2
num =  3
num =  4
num =  5
num =  6
num =  7
num =  8
程序结束
```