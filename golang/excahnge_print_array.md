# 使用channel来实现交替打印两个数组的信息
* 使用两个协程 两个channel来控制
* 两个协程的启动由main协程来控制

## 代码演示如下:
```golang
package main

import (
	"fmt"
	"time"
)

func main() {
	var users = []string{"fj", "tn"}
	var jobs = []string{"tencent", "ali"}
	var c1 = make(chan bool)
	var c2 = make(chan bool)

	go func() {
		for _, val := range users {
			<-c1
			fmt.Printf("name: %v ", val)
			c2 <- true
		}
	}()

	go func() {
		for _, val := range jobs {
			<-c2
			fmt.Printf("job: %v\n", val)
			c1 <- true
		}
	}()
	c1 <- true
	time.Sleep(time.Second)
	fmt.Println("程序结束")
}
运行结果:
name: fj job: tencent
name: tn job: ali
程序结束
```