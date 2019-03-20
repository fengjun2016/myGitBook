# golang http 中间件实现思路分析以及总结
* 方式一: 声明一个新的结构体 实现http.ServeHTTP(w http.ResponseWriter, r *http.Request)方法 
* 方式二: 重新写一个函数 将http.Handler 作为参数传入该方法


## 实现方式一: http.HandlerFunc() 将MyHandler 强制转换成http.Handler类型 查看文档能够知道:
> The HandlerFunc type is an adapter to allow the use of ordinary functions as HTTP handlers. If f is a function with the appropriate signature, HandlerFunc(f) is a Handler that calls f.

type HandlerFunc func(ResponseWriter, *Request)

> 看到上面最后一句就可以知道: 强制转换f 成为http.Handler 类型 注意这个f必须要跟http.Handler参数列表一致
  而在go源码里面可以知道: 只要是被强转的函数f 都实现了ServeHTTP方法 并在方法里面调用它自己, 这就解释了这个原理
  ```golang
  func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
  	f(w, r)
  }
  ```
	
```golang
package main 

import (
	"net/http"
)

type SingleHost struct {
	handler http.Handler
	allowedHost string
}

func (sh *SingleHost) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	if r.Host == sh.allowedHost {
		sh.handler.ServeHTTP(w, r)   //下面一定要实现一下这个底层的方法
	} else {
		w.WriteHeader(403)
		w.Write([]byte("Not allowed http request host"))
	}
}

func MyHandler(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(200)
	w.Write([]byte("Hello cute handsome boy"))
}

func main() {
	sh := &Single{http.HandlerFunc(MyHandler), allowedHost:"example.com"}
	http.ListenAndServe(":1234", sh)
}
```

## 实现方式二: 只需要写一个函数，传入一个http.Handler 作为参数即可, 返回一个http.Handler
```golang
package main

import (
	"net/http"
)

func SingleHost(handler http.Handler, allowedHost string) http.Handler {
	fn := func(w http.ResponseWriterm, r *http.Request) {
		if r.Host == allowedHost {
			handler.ServeHTTP(w, r)
		} else {
			w.WriteHeader(403)
		}
	}
	return http.HandlerFunc(fn)
}

func MyHandler(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello"))
}

func main() {
	handler := SingleHost(http.HandlerFunc(MyHandler), "example.com")
	http.ListenAndServe(":1234", handler)
}
```