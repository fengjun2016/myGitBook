# gopath 相关环境变量知识设置

* 默认在~/go(unix,linux), windows操作系统省略
* 官方推荐：所有项目和第三方库都放在同一个GOPATH下
* 也可以将每个项目放在不同的GOPATH下面
* 在某个包里面使用go install 会编译该目录 并将编译后的二进制文件放入gopath下面的bin目录里面 这样的话在其他地方就可以随意进行访问了