# 关于range使用的时候会复制对象的疑问 现在暂时不是很理解 暂时mark下

## 场景一 range数组的时候会出现影响原来的数据

```golang
	a := [3]int{0, 1, 2}

	for i, v := range a {    //此时index、value都是从复制品中取出
		if i == 0 {   //在修改前,我们先修改原数组
			a[1], a[2] = 999, 999
			fmt.Println(a)     //确认修改有效,输出[0, 999, 999]
		}
		a[i] = v + 100   //使用复制品中取出的value修改原数组
	}
	fmt.Println(a)    //输出[100, 101, 102]
```


## 场景二 range引用结构的数据的时候 不会复制底层数据

```golang
	s := []{1, 2, 3, 4, 5}

	for i, v := range s {    //复制struct slice {pointer, len, cap}
		if i == 0 {
			s = s[:3]     //对slice的修改，不会影响range
			s[2] = 100
		}
		fmt.Ptintln(i, v)
	}
	0 1
	1 2
	2 100
	3 4
	4 5
```

## 场景三 range在循环遍历输出chan c类型的时候 注意c为nil的时候仍然也会进行遍历输出 除非手动close掉c close(c) range缓冲队列逐个获取的方法

```golang
	package main

	import (
		"fmt"
		"time"
	)

	var message = make(chan string, 3) //FIFO先进先出

	func sample() {
		message <- "hello gorutine1!"
		message <- "hello gorutine2!"
		message <- "hello gorutine3!"
		message <- "hello gorutine4!"
	}

	func sample2() {
		time.Sleep(2 * time.Second)
		str := <-message
		str = str + "I'm gorutine"
		message <- str
		//close(message)
	}
	func main() {
		go sample()
		go sample2()
		time.Sleep(3 * time.Second)
		for str := range message {
			fmt.Println(str)
		}
		fmt.Println("Hello World!")
	}
	hello gorutine2!
	hello gorutine3!
	hello gorutine4!
	hello gorutine1!I'm gorutine
	
	fatal error: all goroutines are asleep - deadlock!
	
	goroutine 1 [chan receive]:
	main.main()
		/Users/fengjun/workspace/go_workspace/go_mooc/test.go:28 +0x123
	
	exit status 2
```
