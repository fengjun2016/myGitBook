# 关于golang当中文件读写的打开的两种方式总结如下:

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