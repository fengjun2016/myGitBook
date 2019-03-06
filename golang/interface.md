# golang 中关于接口编程 要记住的就是golang是面向接口编程 继承和多态是由接口来实现和完成的 面向对象由封装来完成

## duck typing概念基础知识扫盲 --来自维基百科
> 在程序设计中，鸭子类型（英语：duck typing）是动态类型的一种风格。在这种风格中，一个对象有效的语义，不是由继承自特定的类或实现特定的接口，而是由"当前方法和属性的集合"决定。这个概念的名字来源于由James Whitcomb Riley提出的鸭子测试（见下面的“历史”章节），“鸭子测试”可以这样表述：

> “当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。”[1][2]
在鸭子类型中，关注点在于对象的行为，能作什么；而不是关注对象所属的类型。例如，在不使用鸭子类型的语言中，我们可以编写一个函数，它接受一个类型为"鸭子"的对象，并调用它的"走"和"叫"方法。在使用鸭子类型的语言中，这样的一个函数可以接受一个任意类型的对象，并调用它的"走"和"叫"方法。如果这些需要被调用的方法不存在，那么将引发一个运行时错误。任何拥有这样的正确的"走"和"叫"方法的对象都可被函数接受的这种行为引出了以上表述，这种决定类型的方式因此得名。

> 鸭子类型通常得益于"不"测试方法和函数中参数的类型，而是依赖文档、清晰的代码和测试来确保正确使用。

> 描述事物的外部行为而非内部结构

> 严格来说golang属于结构化型系统，类似duck typing, 但它不是duck typing系统, 因为golang 不是动态绑定

> 接口的实现是隐式的

> 只要实现接口里的方法 就代表实现了该接口


## 实现接口实例展示
```golang
package mock

type Retriever struct {
	Contents string
}

func (r Retriever) Get(url string) string {
	return r.Contents
}
```

```golang
package real

import (
	"net/http"
	"net/http/httputil"
	"time"
)

type Retriever struct {
	UserAgent string
	Timout    time.Duration
}

func (r Retriever) Get(url string) string {
	resp, err := http.Get(url)
	if err != nil {
		panic(err)
	}

	result, err := httputil.DumpResponse(resp, true)
	resp.Body.Close()
	if err != nil {
		panic(err)
	}

	return string(result)
}
```

```golang
package main

import (
	"fmt"
	"go_mooc/retriever/mock"
	"go_mooc/retriever/real"
)

type Retriever interface {
	Get(url string) string
}

func download(r Retriever) string {
	return r.Get("http://www.imooc.com")
}

func main() {
	var r Retriever
	r = mock.Retriever{"this is a fake imooc.com"}
	r = real.Retriever{}
	fmt.Println(download(r))
}
```

## 获得接口的类型的方法
* 方法一: switch obj{}
```golang
func inspect(r Retriever) {
	fmt.Printf("%T %v\n", r, r)
	switch v := r.(type) {
	case mock.Retriever:
		fmt.Println("Contents:", v.Contents)
	case *real.Retriever:
		fmt.Println("UserAgent:", v.UserAgent)
	}
}
```

* 方法二: Type assertion
```golang
// Type assertion
if realRetriever, ok := r.(*real.Retriever); ok {
	fmt.Println(realRetriever.TimeOut)
} else {
	fmt.Println("not a *real.Retriever")
}
```


## 接口变量里面有啥？
* 实现者的类型
* 实现者的值(或者说是实现者的指针)
* 接口变量同样采用值传递，几乎不需要使用接口的指针

## 查看接口变量
* 表示任何类型: interface{}
* Type Assertion
* Type Switch

### 支持任何类型 但是里面限定int
```golang
package queue

//定义别名 扩充已有类型的方法
type Queue []interface{}

//还有因为这里传入的是指针 所以里面访问元素的时候 需要前面加*号
func (queue *Queue) Push(value interface{}) {
	*queue = append(*queue, value.(int))
}

//特别注意下面的括号(*queue)[0] 不然会出现相应的报错信息出现
func (queue *Queue) Pop() interface{} {
	head := (*queue)[0]
	*queue = (*queue)[1:]
	return head.(int)
}

func (queue *Queue) IsEmpty() bool {
	return len(*queue) == 0
}
```

## 接口的组合
* 注意与结构体的组合写法相区别开来才行
* 只要实现了某个接口的方法 就算是实现了该接口 某种类型结构体可以同时实现多个接口的方法 具体怎么用则由使用情况来决定 具体看下面mock.Retrivever

```golang
package mock

type Retriever struct {
	Contents string
}

func (r *Retriever) Get(url string) string {
	return r.Contents
}

func (r *Retriever) Post(url string, form map[string]string) string {
	r.Contents = form["contents"]
	return "ok"
}

```
```golang
package real

import (
	"net/http"
	"net/http/httputil"
	"time"
)

type Retriever struct {
	UserAgent string
	TimeOut   time.Duration
}

func (r *Retriever) Get(url string) string {
	resp, err := http.Get(url)
	if err != nil {
		panic(err)
	}

	result, err := httputil.DumpResponse(resp, true)
	resp.Body.Close()
	if err != nil {
		panic(err)
	}

	return string(result)
}
```

```golang
package main

import (
	"fmt"
	"go_mooc/retriever/mock"
	"go_mooc/retriever/real"
	"time"
)

type Retriever interface {
	Get(url string) string
}

type Poster interface {
	Post(url string, form map[string]string) string
}

//接口的组合
type RetrieverPoster interface {
	Retriever
	Poster
}

const url = "http://www.imooc.com"

func Session(s RetrieverPoster) string {
	s.Post(url, map[string]string{
		"contents": "another a fake imooc.com",
	})
	return s.Get(url)
}

func download(r Retriever) string {
	return r.Get(url)
}

func inspect(r Retriever) {
	fmt.Printf("%T %v\n", r, r)
	switch v := r.(type) {
	case *mock.Retriever:
		fmt.Println("Contents:", v.Contents)
	case *real.Retriever:
		fmt.Println("UserAgent:", v.UserAgent)
	}
}

func main() {
	var r Retriever
	var retriever RetrieverPoster
	retriever := &mock.Retriever{"this is a fake imooc.com"}
	fmt.Printf("%T %v\n", retriever, retriever)
	// inspect(r)

	r = &real.Retriever{
		UserAgent: "Mozilla/5.0",
		TimeOut:   time.Minute,
	}
	inspect(r)

	// Type assertion
	if realRetriever, ok := r.(*real.Retriever); ok {
		fmt.Println(realRetriever.TimeOut)
	} else {
		fmt.Println("not a *real.Retriever")
	}
	// fmt.Println(download(r))

	fmt.Println("Try a session")
	fmt.Println(Session(retriever))
}
```

## 几个系统常用的接口
* String()接口 使用fmt.Println()打印的时候会自动打印输出字符串

### 官方文档实例代码
```golang
package main

import "fmt"

type Person struct {
	Name string
	Age  int
}

func (p Person) String() string {
	return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
}

func main() {
	a := Person{"Arthur Dent", 42}
	z := Person{"Zaphod Beeblebrox", 9001}
	fmt.Println(a, z)
}
运行结果:
Arthur Dent (42 years) Zaphod Beeblebrox (9001 years)
```
```golang
package mock

import (
	"fmt"
)

type Retriever struct {
	Contents string
}

func (r *Retriever) Get(url string) string {
	return r.Contents
}

func (r *Retriever) Post(url string, form map[string]string) string {
	r.Contents = form["contents"]
	return "ok"
}

func (r *Retriever) String() string {
	return fmt.Sprintf("Retriever: {Contents=%s}", r.Contents)
}
运行结果:
Inspecting Retriever: {Contents=this is a fake imooc.com}
```

