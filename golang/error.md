# golang 中的错误处理相关总结以及defer调用相关总结

## defer 调用时程序结束退出时才会调用
* 在程序调用结束时才会执行
* 同一个范围内的defer是一个栈 先声明的后执行

## golang错误处理相关代码演示

```golang
func WriteFile(filename string) {
	file, err := os.OpenFile(filename, os.O_APPEND|os.O_WDWR, 0644)
	if err != nil {
		if pathError, ok := err.(*os.PathError); !ok {
			panic(err)
		} else {
			fmt.Printf("%s, %s, %s\n",
				pathError.Op,
				pathError.Path,
				pathError.Err)
		}
	}

	defer file.Close()
	writer := bufio.NewWriter(file)
	defer writer.Flush()
	fmt.Fprintf(writer, contents)
}
```

## 集中处理错误代码实例
```golang
package filelisting

import (
	"io/ioutil"
	"net/http"
	"os"
)

func MyHandler(writer http.ResponseWriter, request *http.Request) error {
	path := request.URL.Path[len("/list/"):]
	file, err := os.Open(path)
	if err != nil {
		return err
	}
	defer file.Close()
	all, err := ioutil.ReadAll(file)
	if err != nil {
		return err
	}
	writer.Write(all)
	return nil //最后如果没有错误的话就返回nil
}
```


```golang
package main

import (
	"github.com/gpmgo/gopm/modules/log"
	"go_mooc/filelistingserver/filelisting"
	"net/http"
	"os"
)

type appHandler func(writer http.ResponseWriter, request *http.Request) error

//集中处理错误 函数式编程 函数作为参数
func errWrapper(handler appHandler) func(writer http.ResponseWriter, request *http.Request) {
	return func(writer http.ResponseWriter, request *http.Request) {
		err := handler(writer, request)
		if err != nil {
			log.Warn("Error handling request: %s", err.Error())
			code := http.StatusOK
			switch {
			case os.IsNotExist(err):
				code = http.StatusNotFound
			case os.IsPermission(err):
				code = http.StatusForbidden
			default:
				code = http.StatusInternalServerError
			}
			http.Error(writer, http.StatusText(code), code)

		}
	}
}

func main() {
	//注意函数当参数的时候 传递只用传函数名称即可
	http.HandleFunc("/list/", errWrapper(filelisting.MyHandler))
	http.ListenAndServe("localhost:8887", nil)
}
```




## panic和recover

> panic 特性和特点
* panic会停止当前函数执行 所以建议少用 因为会直接down掉整个程序的执行
* 一直向上返回，会执行每一层的defer
* 如果没有遇见recover,程序就会像最上面的一条一样直接退出 相当于catch

> recover 特性和特点
* 仅在defer调用中调用
* 获取panic的值
* 如果无法处理，可重新panic

```golang
package main

import (
	"errors"
)

func tryRecover() {
	//匿名函数 加()调用执行 捕捉panic抛出来的错误
	defer func() {
		r := recover()
		//检查是否是error类型 如果是则打印错误信息
		if err, ok := r.(error); ok {
			fmt.Println("Error occurred:", err)
		} else {
			panic(err)
		}
	}()
	panic(errors.New("this is an errors"))
}

func main() {
	tryRecover()
}
```