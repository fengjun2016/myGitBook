# golang中面向对象及结构体封装相关知识

* go语言仅支持封装，不支持继承和多态 而是面向接口编程
* go关键字没有class 只有struct
* go里面没有构造函数 但可以自己写工厂函数
* go里面默认是传值类型 所以需要改变数据的话 需要传指针
* 记住如果在写某个结构体的方法的时候 需要改值的话 接收者需要是指针类型 即是传*treeNode 这种的类型
* 为结构定义方法 显示定义和命名方法接收者
* go语言没有this和self指针
* 只有使用指针才可以改变结构内容 使用指针作为方法接收者
* nil指针也可以调用方法


```golang
package main

import (
	"fmt"
)

type treeNode struct {
	value       int
	left, right *treeNode
}

//使用自定义工厂函数  注意返回了局部变量的地址
func createNode(value int) *treeNode {
	return &treeNode{value: value}
}

// 为结构定义方法 需要定义接收者
func (node treeNode) print() {
	fmt.Print(node.value, " ")
}

//下面如果要修改原来的值的话 需要定义传指针
func (node *treeNode) setValue(value int) {
	if node == nil {
		fmt.Println("Setting value to nil node. Ignore")
		return
	}
	node.value = value
}

//遍历一棵树的方法 中序 递归遍历该树
func (node *treeNode) traverse() {
	if node == nil {
		return
	}
	node.left.traverse()
	node.print()
	node.right.traverse()
}

func main() {
	var root treeNode
	// fmt.Println(root)
	root = treeNode{value: 3}
	root.left = &treeNode{}
	root.right = &treeNode{5, nil, nil}
	root.right.left = new(treeNode)
	root.left.right = createNode(2)

	root.right.left.setValue(4)
	// root.right.left.print()
	// fmt.Println()

	// root.print()
	// root.setValue(100)

	// pRoot := &root
	// pRoot.print()
	// pRoot.setValue(200)
	// pRoot.print()

	// var lRoot *treeNode
	// lRoot.setValue(200)
	// lRoot = &root
	// lRoot.setValue(300)
	// lRoot.print()
	root.traverse()
}
```

## 值接收者 vs 指针接收者
* 要改变内容必须使用指针接收者
* 结构过大也考虑使用指针接收者
* 一致性:如有指针接收者，最好都是指针接收者
* 值接收者才是go语言特有的 指针接收者其他语言都有的
* 无论值/指针接收者均可接收值/指针



# go语言封装相关知识
* 名字一般使用驼峰命名方法
* 首字母大写:public 公有方法、属性、结构体
* 首字母小写:private 私有方法、属性、结构体
* 但是上面的公有和私有方法都是针对包而言的 例如package main
* 同一个结构体的方法不一定只写在同一个文件中 可以分散到两个文件中也是可以的

# go 里面包相关知识总结
* 每个目录只能有一个包 但是包名不一定和目录名相同
* main包包含可执行入口 有一个main函数 main函数必须放置在main包里面才行
* 为结构定义的方法必须放在同一个包内，但可以是不同文件
* 包外使用的时候 需要使用import命令引入该包文件 引用串从src下开始到该包文件的父目录即可


# 如何扩充已有类型的呢
* 定义别名
* 使用组合思想 即新定义一个结构体 里面含有已知类型作为其属性


> 使用组合思想 扩充已有类型 新增后序遍历子树的方法
```golang
//使用组合思想 从而来扩充已有类型 因为golang中没有继承和多态的思想
type myTreeNode struct {
	node *tree.Node
}

//扩充新的方法 使用组合思想 实现后序遍历该结构树
func (myNode *myTreeNode) postOrder() {
	if myNode == nil || myNode.node == nil {
		return
	}

	left := myTreeNode{myNode.node.Left}
	right := myTreeNode{myNode.node.Right} //还有记住只有变量才能够取到地址
	left.postOrder()
	right.postOrder()
	myNode.node.Print()
}
```

> 使用别名思想来扩充已有类型

```golang
package queue

//定义别名 扩充已有类型的方法
type Queue []int

//还有因为这里传入的是指针 所以里面访问元素的时候 需要前面加*号
func (queue *Queue) Push(value int) {
	*queue = append(*queue, value)
}

//特别注意下面的括号(*queue)[0] 不然会出现相应的报错信息出现
func (queue *Queue) Pop() int {
	head := (*queue)[0]
	*queue = (*queue)[1:]
	return head
}

func (queue *Queue) IsEmpty() bool {
	return len(*queue) == 0
}
```

```golang
package main

import (
	"fmt"
	"go_mooc/queue"
)

func main() {
	q := queue.Queue{1}

	q.Push(2)
	q.Push(3)
	fmt.Println(q.Pop())
	fmt.Println(q.Pop())
	fmt.Println(q.IsEmpty())
	fmt.Println(q.Pop())
	fmt.Println(q.IsEmpty())
}
运行结果:
1
2
false
3
true
```