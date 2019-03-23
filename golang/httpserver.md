# 详细来完成一个httpserver的过程


## 自己使用底层的库来实现一个httpServer服务器的过程
* 1.设置路由 mux
* 2.配置http.Server对象
* 3.使用net.Listen("tcp", ":port") 监听端口 返回一个net.Listener
* 4.使用go 开启协程 让server服务启动 等待客户端连接响应 go server.Serve(listener)
* 5.在main里使用time.Sleep()循环等待 防止主程序退出

```golang
package main

import (
	"log"
	"net"
	"net/http"
	"time"
)

func handleWorker(rw http.ResponseWriter, req *http.Request) {
	rw.Write([]byte("Hello cute boy"))
}

func init() {
	var (
		server   *http.Server
		mux      *http.ServeMux
		listener net.Listener
		err      error
	)

	//新建路由
	mux = http.NewServeMux()
	mux.HandleFunc("/index", handleWorker)

	server = &http.Server{
		ReadTimeout:  5000 * time.Millisecond,
		WriteTimeout: 5000 * time.Millisecond,
		Handler:      mux,
	}

	//监听端口 使用tcp协议
	listener, err = net.Listen("tcp", ":1234")
	if err != nil {
		log.Println(err)
		return
	}

	//开始等待客户端连接 传入监听器 新开一个协程 并发处理来自客户端的连接
	go server.Serve(listener)
}

func main() {
	for {
		time.Sleep(1 * time.Second) //防止http apiServer协程退出
	}
}
```

## 使用golang里面提供的封装好的方法来写一个httpServer的代码演示 http.ListenAndServe替我们完成了阻塞等待的过程
```golang
package main

import (
	"net/http"
)

func indexHandler(rw http.ResponseWriter, req *http.Request) {
	rw.Write([]byte("hello cute boy"))
}

func loginHandler(rw http.ResponseWriter, req *http.Request) {
	rw.Write([]byte("please login"))
}
func main() {
	http.HandleFunc("/index", indexHandler)
	http.HandleFunc("/login", loginHandler)
	http.ListenAndServe(":1234", nil)   //使用默认的mux 
}
```

```golang
package main 

import (
	"net/http"
)

func indexHandler(rw http.ResponseWriter, req *http.Request) {
	rw.Write([]byte("hello my son"))
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/index.html", indexHandler)
	http.ListenAndServe(":1234", mux)
}
```

```golang
package main 

import (
	"net/http"
)

func indexHandler(rw http.ResponseWriter, req *http.Request) {
	rw.Write([]byte("hello my son"))
}

func main() {
	http.ListenAndServe(":1234", http.HandlerFunc(indexHandler))  //这样一个端口就只能设置一个handler 和路由规则
}
```