# goroutine中间的通道channel 通信的相关知识
* channel通道的定义
* channel通道的使用
* channel通道的意义

## 
![Image text]()
> 用于不同goroutine之间的消息通信切记 如果在同一个goroutine中间 <- 和  <- 操作的话 就会出现所有的协程都会deadblock