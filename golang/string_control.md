# 字符和字符串处理相关常见操作 特别是对于中文字符的处理以及全球不同字符的处置

## golang中string实现的背景介绍
> golang中string底层是通过byte数组实现的。中文字符在unicode下占2个字节，在utf-8编码下占3个字节，而golang默认编码正好是utf-8。

> 关于golang中的rune:
// rune is an alias for int32 and is equivalent to int32 in all ways. It is
// used, by convention, to distinguish character values from integer values.

//int32的别名，几乎在所有方面等同于int32
//它用来区分字符值和整数值

type rune = int32

* 获取字节数长度 len 特别记住这一点 len(s)返回的是底层的字节数 英文占一个字节 中文字符在golang中占有两个字节 并不是获得字符串长度
* 关于utf-8和rune的互相转换
* 关于含有中文字符串的对于每一个中文字符的遍历处理 强制转换该字符串变成一个[]rune数组
* 计算字节数
* 使用range遍历pos, rune pos会不连续
* 使用[]byte转换获得字节
* 使用utf8.RuneCountInStriung获得字符数量

# golang中海油一个byte数据类型与rune相似，它们都是用来表示字符类型的变量类型。它们的不同在于：

> * byte 等同于int8，常用来处理ascii字符
  * rune 等同于int32,常用来处理unicode或utf-8字符


```golang
package main 
import (
	"fmt"
	"unicode/utf8"
)

func main() {
	s := "Yes我爱慕课网!"  //UTF-8编码方式 中文 可变长编码方式 英文是一字节方式 中文是3字节方式存储
	fmt.Println(s)
	//查看字符的具体存储 X打出字节的具体数字 16进制 每个字符的ascii码值
	for _, b := range []byte(s) { //utf-8编码
		fmt.Println("%X\n", b)   	
	}
	fmt.Println()

	for i, ch := range s {  //unicode编码 i是每个字符的开始字节数 ch is a rune(int32) 一个四字节的整数
		fmt.Printf("(%d %X)", i , ch)
	}
	fmt.Println()

	fmt.Println("Rune count:",
		utf8.RuneCountInString(s))   //获取rune个数 也是获取字符串长度的方法之一

	//常用方法
	for i, ch := range []rune(s) {  //获得一个rune数组 一个下标一个英文字符或者一个中文字符 获取字符串长度方法二
		fmt.Println("(%d %c"), i, ch)
	}
	fmt.Println()

	bytes := []bytes(s)   //
	for len(bytes) > 0 {
		ch, size := utf8.DecodeRune(bytes)
		fmt.Printf("(%c %v)", ch, size)
		bytes = bytes[size:]
	}
}

运行结果:
Yes我爱慕课网!
59	65	73	E6	88	91	E7	88	B1	E6	85	95	E8	AF	BE	E7	BD	91	21	//utf-8编码
(0 59)(1 65)(2 73)(3 6211)(6 7231)(9 6155)(12 8BFE)(15 7F51)(18 21)   //unicode编码
(0 Y)(1 e)(2 s)(3 我)(4 爱)(5 慕)(6 课)(7 网)(8 !)    //第几个字符是谁
(Y 1)(e 1)(s 1)(我 3)(爱 3)(慕 3)(课 3)(网 3)(! 1)
```


```golang
package main

import (
    "fmt"
    "unicode/utf8"
)

func main() {

    var str = "hello 你好"

    //golang中string底层是通过byte数组实现的，座椅直接求len 实际是在按字节长度计算  所以一个汉字占3个字节算了3个长度
    fmt.Println("len(str):", len(str))
    
    //以下两种都可以得到str的字符串长度
    
    //golang中的unicode/utf8包提供了用utf-8获取长度的方法
    fmt.Println("RuneCountInString:", utf8.RuneCountInString(str))

    //通过rune类型处理unicode字符
    fmt.Println("rune:", len([]rune(str)))
}
运行结果:
len(str): 12
RuneCountInString: 8
rune: 8
```
