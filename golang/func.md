# 关于 golang中函数一个特殊的地方 在这里mark一下  函数式编程 以及 可变参数列表 
>  语法要点有
* 可返回多个值
* 返回值类型写在最后面
* 函数可以作为参数
* 没有默认参数， 有可变参数


```golang
package main
import (
	"fmt"
	"runtime"
	"reflect"
	"runtime"
)

//多多关注一下下面这里default里面返回error信息
func zval(a, b int, op string) (int, error){
	switch op {
	case "+":
		return a+b, nil
	case "-":
		return a-b, nil
	case "*":
		return a*b, nil
	case "/":
		return a/b, nil
	default:
		//panic("unsupported operation: " + op)   要记住的是 panic会中断整个程序的执行 所以这里可以改成抛错误
		return 0, fmt.Errorf("unsupported operation: %s", op)
	}
}

//除法返回两个值 返回商和余数 多返回值的函数使用方法
func div(a, b int) (q, r int) {
	return a/b, a%b
}

//但是这个函数形式并不推荐使用 推荐使用的还是上面的这种写法
func div1(a, b int) (q, r int) {
	q = a /b
	r = a % b
	return
}

//函数式编程  也叫匿名函数编程 这里使用了反射来完成
func apply(op func(int, int) int, a, b int) int {
	fmt.Printf("Calling %s with %d, %d\n",
		runtime.FuncForPc(reflect.ValueOf(op).Pointer()).Name(), a, b)
	return op(a, b)
}

//可变参数列表  ...int机是代表可以传入多个int 类型的参数
func sum(numbers ...int) int{
	s := 0
	for _, v := rang numbers {
		s += v
	}
	return s
}

func main() {
	fmt.Println(eval(3, 4, "*"))
	q, r := div(13, 3)
	fmt.Println(q, r)
	//多多注意这里匿名函数的写法
	fmt.Println(apply(func(a, b int) int {
		return int(math.Pow(float64(a), float64(b)))
		}, 3, 4))
}
```