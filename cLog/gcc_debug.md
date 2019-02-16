# gcc linux 下手动debug命令总结

> 测试程序如下所示
  ``` c 
  #include<stdio.h>
  int main(){
  	struct _s{
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
## 带gdb模式下的手动命令调试
> gcc test.c -g -o test

## 执行完上面的命令之后 进行运行
