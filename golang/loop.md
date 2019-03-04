
```golang
package main
import (
	"fmt"
	"strconv"
	"os"
	"bufio"
)

//10进制转换成为二进制的函数
func convertToBin(n int) string {
	var result string
	for ; n > 0; n /= 2{
		lsb := n%2
		result = strconv.Itoa(lsb) + result;
	}
	return result
}

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

func main() {
	fmt.Println(
		convertToBin(5),  //101
		convertToBin(13),  //1011 -> 1101 
	)
}
```