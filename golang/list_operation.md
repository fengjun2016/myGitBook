# 小技巧之 实现golang的链式操作

*  实例代码演示
```golang
package main

import (
	"fmt"
)

type Stu struct {
	Name string
	Age int 
}

func (s *Stu) SetName(name string) *Stu {
	s.Name = name
	return s 
}

func (s *Stu) SetAge(age int) *Stu {
	s.Age = age
	return s
}

func (s *Stu) Print() {
	fmt.Printf("age:%d, name:%s\n", s.Age, s.Name)
}

func main() {
	stu := &Stu{}
	stu.SetAge(12).SetName("jornfeng").Print()  //这里就可以实现链式调用
}
```