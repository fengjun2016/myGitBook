# 关于range使用的时候会复制对象的疑问 现在暂时不是很理解 暂时mark下

## 场景一 range数组的时候会出现影响原来的数据

```golang
	a := [3]int{0, 1, 2}

	for i, v := range a {    //此时index、value都是从复制品中取出
		if i == 0 {   //在修改前,我们先修改原数组
			a[1], a[2] = 999, 999
			fmt.Println(a)     //确认修改有效,输出[0, 999, 999]
		}
		a[i] = v + 100   //使用复制品中取出的value修改原数组
	}
	fmt.Println(a)    //输出[100, 101, 102]
```


## 场景二 range引用结构的数据的时候 不会复制底层数据

```golang
	s := []{1, 2, 3, 4, 5}

	for i, v := range s {    //复制struct slice {pointer, len, cap}
		if i == 0 {
			s = s[:3]     //对slice的修改，不会影响range
		}
		fmt.Ptintln(i, v)
	}
	0 1
	1 2
	2 100
	3 4
	4 5
```

