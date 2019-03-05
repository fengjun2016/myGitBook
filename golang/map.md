# 关于map的相关操作和定义
* map的声明定义方法

> 声明方式一:
```golang
m := map[string]string{
	"name":"ccmouse",
	"course":"golang",
	"site":"imooc",
	"quality":"notbad",
}
```

> 声明方式二: m2 == empty map
```goalng
m2 := make(map[string]int) //空map 
```

> 声明方式三: m3 == nil
```golang
var m3 map[string]int //也是声明一个空的map 
```

* map的定义方式 使用 for k,v := range map
```golang
m := map[string]string{
	"name":"ccmouse",
	"course":"golang",
	"site":"imooc",
	"quality":"notbad",
}
for k, v := range m{
	fmt.Println(k, v)
}
```

* 获取某个key的val属性值
```golang
m := map[string]string{
	"name":"ccmouse",
	"course":"golang",
	"site":"imooc",
	"quality":"notbad",
}
courseName := m["course"]
fmt.Println(courseName)
```

* 如果访问不存在的key的属性值的时候 是会返回一个zero value, 但是并不会报错,所以正常访问的时候最好加一个ok标志
```golang
m := map[string]string{
	"name":"ccmouse",
	"course":"golang",
	"site":"imooc",
	"quality":"notbad",
}
courseName, ok := m["course"]
if ok {
	fmt.Println(courseName)
} else {
	fmt.Println("key dose not exist.")
}
```

* 删除map中的某个key的元素 使用delete方法delete(m, key)
```golang
m := map[string]string{
	"name":"ccmouse",
	"course":"golang",
	"site":"imooc",
	"quality":"notbad",
}
delete(m, "name")
```

* map的重要特性之在golang当中是一个hashmasp 是无序的一个map 每次遍历的时候出现的顺序并不是固定的 如需排序 需手动加入到一个slice当中，手动对其进行排序操作

* 使用len(map)来获得元素个数

* map当中key的注意事项
> * map使用哈希表，必须可以比较相等
  * 除了slice,map, function的内建类型都可以作为key
  * 自定义Struct类型不包含上述字段，也可以作为key



* map当中的例题练习:寻找最长不含有连续重复字符的子串 但是目前无法支持中文字符的操作 对于修正见string_control.md
```golang
package main

func lengthOfNonRepeatingSubstr(s string) int{
	lastOccurred := make(map[byte]int)
	start := 0
	maxLength := 0

	for i, ch := range []byte(s) {
		lastI, ok := lastOccurred[ch]
		if ok && lastI >= start {
			start = lastI + 1
		}
		if i - start + 1 > maxLength {
			maxLength = i - start + 1
		}
		lastOccurred[ch] = i
	}
	return maxLength
}

func main() {
	fmt.Println(
		lengthOfNonRepeatingSubstr("abcabcbb"))
	fmt.Println(
		lengthOfNonRepeatingSubstr("bbbbb"))
	fmt.Println(
		lengthOfNonRepeatingSubstr("pwwkew"))
}
```

