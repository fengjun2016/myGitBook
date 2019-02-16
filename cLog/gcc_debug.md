# gcc linux 下手动debug命令总结
#### 结构体字节内存对齐基础知识扫盲
> 就是按字节对齐的方式存储的！即以结构体成员中占内存最多的数据类型所占的字节数为标准，所有的成员在分配内存时都要与这个长度对齐。我们举一个例子：我们以上面这个程序为例，结构体变量 data 的成员中占内存最多的数据类型是 int 型，其占 4 字节的内存空间，那么所有成员在分配内存时都要与 4 字节的长度对齐。也就是说，虽然 char 只占 1 字节，但是为了与 4 字节的长度对齐，它后面的 3 字节都会空着

> 测试程序如下所示
  ``` c 
  #include<stdio.h>
  int main(){
  	struct _s{  //结构体 空占并且字节对齐
		char a;  //64位机 占1个字节
		int b;   //4个字节
		long c;  //8个字节
		void* d; //8个字节
		int e;   //4个字节
		char* f; //8个字节
  	}s;
  	s.a = 'a';
  	s.b = 1;
  	s.c = 2;
  	s.d = NULL;
  	s.e = 3;
  	s.f = &s.a;
  	printf("size of struct s is %d\n", sizeof(s));
	return 0;
  }
  ```

  ``` c
  #include<stdio.h>
  int main(){
  	union _s{ //联合体 覆盖8个字节 总共只占有8个字节 元素共享同一块内存
		char a;
		int b;
		long c;
		void* d;
		int e;
		char* f;
  	}s;
  	s.a = 'a';
  	s.b = 1;
  	s.c = 2;
  	s.d = NULL;
  	s.e = 3;
  	s.f = &s.a;
  	printf("size of struct s is %d\n", sizeof(s));
	return 0;
  }
  ```

## 带gdb模式下的手动命令调试 生成二进制文件
> gcc test.c -g -o test

### 执行完上面的命令之后 进行gdb调试运行
> gdb ./test

### 使用命令b进行断点调试 比如下面在main函数处进行断点
> b main

### 然后使用命令r运行断点调试 从断点处一步一步调试运行
> r

### 使用命令p打印断点处的结构体变量s
> p s

### 然后使用命令n运行下一步
> n

### 再次打印结构体变量s
> p s

### 再次运行命令n 继续执行下一步 打印&s.a的地址
> n
  p &s.a

