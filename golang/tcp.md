# golang实现tcp简单服务器的记录

## server端代码实现
```golang
package main

import (
	"fmt"
	"log"
	"net"
)

func main() {
	//监听
	listener, err := net.Listen("tcp", ":8001") //不写IP地址代表监听的是本机
	if err != nil {
		fmt.Println("listener errinfo = ", err)
		return
	}
	defer listener.Close()

	for {
		conn, err := listener.Accept() //阻塞，等待用户连接
		if err != nil {
			log.Fatal(err)
		}

		// 开一个新协程来处理连接信息
		go func(conn net.Conn) {
			// 关闭连接
			defer conn.Close()
			b := make([]byte, 1024)
			n, _ := conn.Read(b)
			fmt.Println(string(b[:n]))
		}(conn)
	}
}
```

## client端代码
```golang
package main

import (
	"fmt"
	"net"
)

func main() {
	//主动连接服务器
	conn, err := net.Dial("tcp", "127.0.0.1:8001")
	if err != nil {
		fmt.Println("err = ", err)
		return
	}
	defer conn.Close()

	conn.Write([]byte("are you ok"))
}
```


> 上面的客户端的代码只能完成一次性连接就会退出整个程序的执行 下面对上面的代码进行优化 实现并发

## server端
```golang
package main

import (
	"fmt"
	"log"
	"net"
)

func main() {
	//监听
	listener, err := net.Listen("tcp", ":8001") //不写IP地址代表监听的是本机
	if err != nil {
		fmt.Println("listener errinfo = ", err)
		return
	}
	defer listener.Close()

	for {
		conn, err := listener.Accept() //阻塞，等待用户连接
		if err != nil {
			log.Fatal(err)
		}

		// 开一个新协程来处理连接信息  支持并发不同的客户端连接服务器
		go func(conn net.Conn) {
			for {         // for 循环实现服务器与客户端的长连接
				conn.Write([]byte("I'm server can you listen to me"))
				b := make([]byte, 1024)
				n, _ := conn.Read(b)
				if n >= 1 {
					fmt.Println(string(b[:n]))
				}
			}
			defer conn.Close()
		}(conn)
	}
}
```

## client端代码 注意for循环一定要放在后面 不然主程序会很容易立马结束
```golang
package main

import (
	"fmt"
	"net"
	// "os"
)

func main() {
	//连接服务器
	conn, err := net.Dial("tcp", "127.0.0.1:8001")
	if err != nil {
		fmt.Println("net.Dial error = ", err)
		return
	}

	//接收服务器信息
	go func() {
		buf := make([]byte, 2048)
		for {    // 实现客户端与服务器的长连接
			n, err := conn.Read(buf)
			if err != nil {
				fmt.Println("net.read error = ", err)
				return
			}
			fmt.Println(string(buf[:n]))
		}
	}()

	//发信息给服务器
	for {
		str := make([]byte, 2048)
		for {
			n, err := os.Stdin.Read(str) //从键盘一直不停的获取回复内容
			if err != nil {
				fmt.Println("stdin error = ", err)
				return
			}
			conn.Write(str[:n])
		}
	}
}
```