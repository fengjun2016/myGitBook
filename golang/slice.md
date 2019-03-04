# slice 切片相关概念厘清 

* 定义方式 是 []int([]type) 这种形式发的定义
* 一个slice 有len 和 cap  slice是对arr的一种view的实现 cap是指从左边开始到数组结束的长度 len则是slice当前长度
* 对于一个slice中数据的修改可以影响到底层数组的数据
* s[i]不可以超过len(s) 
* slice可以向后扩展 而不能向前扩展 但是向后扩展不能超越底层数组cap(s)
* 向slice中添加元素 使用append s2 := append(s1, 12) 系统会自动扩充cap
* 如果想向slice中一次性加入多个元素 可以采取的方法是 s2 = append(s2, s1[:]...)

```golang
arr1 := [6]{1,2,3,4,5,6}
slice := arr1[1:4]
slice1 := arr1[:]
slice2 := arr1[1:]
slice3 := arr1[:6]   //包括左边不包括右边
slice4 := slice[2:5]   // reslice 也是成立的 因为cap是从左边 开始一直可以看到数组最后的
```



> 注意点:
* 关于最后一点追加元素的注意点 追加的元素如果没有超过原底层数组的长度 也就是没有超过cap(slice) 都会以原数组为view 并进行修改
例如原来上面的arr1  声明的slice 如果append(slice, 7) 的话 就会影响到原来的6 数组也会变成最后一个数据为7

* 如果追加到数据个数超过了cap 则不再以原来的数组为view 系统会重新分配 但是具体是哪一个我们这里无法知道

* 添加元素的时候如果超越cap 系统会重新分配更大的底层数组

* 由于值传递的关系， 必须有变量接收append的返回值 因为ptr len cap都会发生变化 例如s = append(s, val)


> 新建slice的定义声明形式
```golang
var s []int;    // Zero value for slice is nil

for i := 0; i < 100; i++ {
	s = append(s, 2*i+1)
}


s1 := []int{2, 4, 6, 8}   //方式2

s2 := make([]int, 16)    //方式3 只知道长度 但是不知道里面具体的内容的时候这里16是len

s3 := make([]int, 16, 32)  //方式4 分别是长度 和 容量 len cap
```

> copy 复制操作 复制slice操作
```golang
s1 := []int{2, 4, 6, 8} 

s2 := make([]int, 16)   

s3 := make([]int, 16, 32) 
copy(s3, s1)   //将s1拷贝进s3
```

> delete操作  例如这路要删除4
```golang 
s1 := []int{2, 4, 6, 8} 
s1 = append(s1[:1], s1[2:]...)
```

> poping操作 弹出操作
```golang
//poping from front
s1 := []int{2, 4, 6, 8} 
front := s1[0]
s1 = s1[1:]

//poping from back
tail := s1[len(s1)-1]
s1 = s1[:len(s1)-1]
```