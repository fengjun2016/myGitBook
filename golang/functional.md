# 函数式编程相关

* 函数式编程

> 在维基百科中，对函数式编程有很详细的介绍。Wiki上对Functional Programming的定义：

In computer science, functional programming is a programming paradigm that treats computation as the evaluation of mathematical functions and avoids state and mutable data.

简单地翻译一下，也就是说函数式编程是一种编程模型，他将计算机运算看做是数学中函数的计算，并且避免了状态以及变量的概念。

* 闭包概念

> 在函数编程中经常用到闭包，闭包是什？它是怎么产生的及用来解决什么问题呢?先给出闭包的字面定义：闭包是由函数及其相关引用环境组合而成的实体(即：闭包=函数+引用环境)。这个从字面上很难理解，特别对于一直使用命令式语言进行编程的程序员们。

闭包只是在形式和表现上像函数，但实际上不是函数。函数是一些可执行的代码，这些代码在函数被定义后就确定了，不会在执行时发生变化，所以一个函数只有一个实例。闭包在运行时可以有多个实例，不同的引用环境和相同的函数组合可以产生不同的实例。所谓引用环境是指在程序执行中的某个点所有处于活跃状态的约束所组成的集合。其中的约束是指一个变量的名字和其所代表的对象之间的联系。那么为什么要把引用环境与函数组合起来呢？这主要是因为在支持嵌套作用域的语言中，有时不能简单直接地确定函数的引用环境。这样的语言一般具有这样的特性：

函数是一等公民（First-class value），即函数可以作为另一个函数的返回值或参数，还可以作为一个变量的值。
函数可以嵌套定义，即在一个函数内部可以定义另一个函数。

在面向对象编程中，我们把对象传来传去，那在函数式编程中，要做的是把函数传来传去，说成术语，把他叫做高阶函数。在数学和计算机科学中，高阶函数是至少满足下列一个条件的函数:

接受一个或多个函数作为输入
输出一个函数

在函数式编程中，函数是基本单位，是第一型，他几乎被用作一切，包括最简单的计算，甚至连变量都被计算所取代。

闭包小结:
函数只是一段可执行代码，编译后就“固化”了，每个函数在内存中只有一份实例，得到函数的入口点便可以执行函数了。在函数式编程语言中，函数是一等公民（First class value）：第一类对象，我们不需要像命令式语言中那样借助函数指针，委托操作函数，函数可以作为另一个函数的参数或返回值，可以赋给一个变量。函数可以嵌套定义，即在一个函数内部可以定义另一个函数，有了嵌套函数这种结构，便会产生闭包问题。如：

## 闭包代码展示：
```golang
package main 

import (
	"fmt"
)

func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos := adder()
	for i := 0; i < 10; i++ {
		fmt.Println(pos(i))
	}
}
```


## 即函数作为一等公民能够作为返回值或参数进行传递或者返回 传参时只用写上函数名称即可 代码实例如下:
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

//集中处理错误
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
	http.HandleFunc("/list/", errWrapper(filelisting.MyHandler))
	http.ListenAndServe("localhost:8887", nil)
}
```

