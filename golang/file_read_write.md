# 关于golang当中文件读写的打开的两种方式总结如下:
* 特别要注意下面golang操作文件的坑

* 方式一:采用 ioutil.ReadFile(filename) 方式直接读写文件
```golang
if contents, err := ioutil.ReadFile(filename); err == nil {
	fmt.Printf(string(contents))
} else {
	fmt.Println("cannot print file contents:", err)
}
```
> 备注: 上面的方法用于读文件内容的时候 比较方便 但是不好处理写文件的追加还是覆盖

* 方式二:采用os包来打开文件 os.OpenFile(filename) 但记得要关闭打开指针
```golang
if fd, err := os.OpenFile(filename, os.O_APPEND|os.O_RDWR, 0644); err == nil {
		//读文件 一行一行读文件内容
		reader := bufio.NewReader(fd)
		for {
			line, err := reader.ReadString('\n')
			fmt.Println(line)
			if err != nil {
				if err == io.EOF {
					break  //退出文件读
				} else {
					fmt.Println(err)
				}
			}	
		}
} else {
	fmt.Println("cannot open this file:", err)
}
defer fd.Close()
```

> 追加写文件实例
```golang
fd, _ := os.OpenFile(filename, os.O_APPEND|os.O_WDWR, 0644)
fd.Write([]byte(contents))
defer fd.Close()
```

```golang
fd, _ := os.OpenFile(filename, os.O_APPEND|os.O_WDWR, 0644)
defer fd.Close()
writer := bufio.NewWiter(fd)
defer writer.Flush()  //从内存导入文件
fmt.Fprintf(writer, contents)
```
 
* 方式三: 一行一行读文件 采用bufio.NewScanner()来一行一行读文件
```golang
//一行一行的读写文件的函数操作
func printFile(filename string) error {
	fd, err := os.OpenFile(filename, os.O_APPEND|os.O_RDWR, 0644)
	if err != nil {
		panic(err)
	}

	//创建一个scanner扫描器 一行一行读文件
	scanner := bufio.NewScanner(fd)
	for scanner.Scan() {
		fmt.Println(scanner.Text())	
	}
}
```

* 方式四: 一次性读取所有的文件内容
```golang
file, err := os.Open(path)
if err != nil {
	panic(err)
}
defer file.Close()
all, err := ioutil.ReadAll(file)
if err != nil {
	panic(err)
}
```


# 关于golang在使用buf缓冲区读写文件的坑
> open file failed: invalid arguments
> 读写文件乱码

## [产生这些错误的背景分析](https://studygolang.com/articles/2911):
> 在C语言中，字符串的内存模型定义为以NUL(\x0)结尾的字节数组。这是为大家所熟知的。
  但是在Golang中并不是如此，Golang中的字符串abc和abc\x0\x0并不相当，所以说Golang明确规定了字符串的长度，而不是以\x0为结尾来判断的。

* 下面看示例代码：
```golang
package main

import (
    "fmt"
    "os"
)

func main() {
    var a[5]byte = [5]byte{'a','b','c'}
    var b[]byte = []byte{'a','b','c'}

    fmt.Printf("len(a): %d, %q\n", len(a), a)
    fmt.Printf("len(b): %d, %q\n", len(b), b)

    slice_a := a[:]
    str_a := string(slice_a)
    str_b := string(b)
    fmt.Printf("len(str_a): %d, %q:%s\n", len(str_a), str_a, str_a)
    fmt.Printf("len(str_b): %d, %q:%s\n", len(str_b), str_b, str_b)

    if str_a == str_b {
        fmt.Println("str_a == str_b")
    } else {
        fmt.Println("str_a != str_b")
    }

    file, err := os.Create(str_a)
    if err != nil {
        fmt.Println(err)
    }
    fmt.Println(file)
}
运行结果:
len(a): 5, "abc\x00\x00"
len(b): 3, "abc"
len(str_a): 5, "abc\x00\x00":abc
len(str_b): 3, "abc":abc
str_a != str_b

open abc: invalid argument

```

> 数组a长度为5，最后两个字节为\x00。切片b长度为3，有效内容为abc。 分别将a和b转换为string类型后，数组a中包含的\x00也被保留下来，所以在对比时，这两者的长度不可能相等的。

当然这还不是最关键的，问题处在当用str_a作为文件名创建文件时file, err := os.Create(str_a)，系统会报错：open abc: invalid argument。说明这不是一个有效的路径名称。

Golang是一种强类的语言，对于数组来说[3]byte和[5]byte并不是同一个类型，不能通用。 当然，数组和切片也更不可能是同一个类型。

解决办法是，迭代数组中的字节，跳过为0的字节：

```golang
func GetValidByte(src []byte) []byte {
    var str_buf []byte
    for _, v := range src {
        if v != 0 {
            str_buf = append(str_buf, v)
        }
    }
    return str_buf
}
```

## 对于上面的坑 下面以一个案例来分析和mark一下 一个远程tcp发送文件的服务器中 遇到os.Open 出现err invalid arguments  

* 最好在使用的时候带上具体的长度 [:n] 这样就不会把\0x00带上了 防止文件写入乱码和文件打不开的问题

### 发送文件端代码
```golang
package main

import (
	"fmt"
	"io"
	"log"
	"net"
	"os"
)

//发送文件内容
func sendFile(path string, conn net.Conn) {
	//以只读方式打开文件
	f, err := os.Open(path)
	if err != nil {
		log.Printf("file open failed: %v", err)
		fmt.Println("Open file err = ", err)
		return
	}

	defer f.Close()
	//核心功能， 读多少就发送多少 分段读
	buf := make([]byte, 2048)
	for {
		n, err := f.Read([]byte(buf))
		if err != nil {
			if err == io.EOF {
				fmt.Println("文件读取完毕")
				return
			} else {
				log.Printf("f.read failed: %v", err)
				fmt.Println("f.read err = ", err)
				return
			}
		}
		// 给服务器发送内容
		fmt.Println(string(buf[:n]))
		conn.Write(buf[:n])    // 坑的地方在这 不加[:n]的话 后面写入文件的时候就会乱码的 因为带有\0x00 
	}
}

func main() {
	//提示输入文件
	fmt.Println("请输入需要传输的文件: ")
	var path string
	fmt.Scan(&path)

	// 获取文件名
	info, err := os.Stat(path)
	if err != nil {
		log.Printf("获取文件名失败: %v", err)
		fmt.Println("获取文件名err = ", err)
		return
	}

	//文件连接服务器
	conn, err1 := net.Dial("tcp", ":1234")
	if err1 != nil {
		log.Printf("net.Dial err = ", err1)
		fmt.Println("net.Dial err = ", err1)
		return
	}
	defer conn.Close()

	//给接收方发送文件名
	_, err = conn.Write([]byte(info.Name()))
	if err != nil {
		fmt.Println("conn.Write err = ", err)
		log.Println("conn.Write err : %v", err)
		return
	}
	buf := make([]byte, 1024)
	n, err := conn.Read(buf)
	if err != nil {
		log.Printf("conn.Read err: %v", err)
		fmt.Println("conn.Read err = ", err)
		return
	}

	if "ok" == string(buf[:n]) {
		fmt.Println("开始发送文件了")
		//发送文件
		sendFile(path, conn)
	}
}
```

### 接收文件端代码
```golang
package main

import (
	"net"
	"io"
	"fmt"
	"log"
	"os"
)

//接收文件内容
func RecvFile(filename string, conn net.Conn) {
	fmt.Println(filename)
	//新建一个文件
	file, err := os.OpenFile(filename, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
	defer file.Close()

	if err != nil {
		fmt.Println("os.Create err = ", err)
		log.Printf("os create err: %v", err)
		return
	}

	for {
		buf := make([]byte, 4*1024)
		n, err := conn.Read(buf)
		if err != nil {
			if err == io.EOF {
				log.Printf("file contens has been received")
				fmt.Printf("%s has been received fully", filename)
				return
			} else {
				log.Printf("received file failed: %v", err)
				fmt.Println("received file failed: ", err)
				return
			}
		}
		fmt.Println(string(buf[:n]))
		file.WriteString(string(buf[:n]))    // 这里记得带上[:n]
	}
}

func main() {
	listener, err := net.Listen("tcp", ":1234")
	defer listener.Close()

	if err != nil {
		log.Printf("net.Listen tcp 1234 failed: %v", err)
		fmt.Println("net.Listen tcp 1234 failed:", err)
		return
	}
	for {
		conn, err := listener.Accept()
		defer conn.Close()
		if err != nil {
			log.Fatal("listener.Accept() tcp:1234 failed: %v", err)
			fmt.Println("listener.Accept() tcp:1234 failed: ", err)
			return
		}
        
        go func(conn net.Conn) {
    		buf := make([]byte, 1024)
    		//读取文件名
            n, _ := conn.Read(buf)
            filename := string(buf[:n])    //这里读取的时候 记得带上[:n]不然就会出现文件打不开的尴尬
            //回复ok
            conn.Write([]byte("ok"))
            //接收文件
            RecvFile(filename, conn)
        }(conn)
	}
}
```


