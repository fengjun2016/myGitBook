# etcd在mac系统中的安装与使用

* 进入[etcd官网](https://coreos.com/etcd/)提供的连接， 点击GitHub Project, 然后选择release版本下载编译好的版本代码包
* 解压下载好的代码包
* cd 进入解压后的代码包
* 启动server端 nohup ./etcd --listen-client-urls 'http://0.0.0.0:2379' --advertise-client-urls 'http://0.0.0.0:2379' &
* client客户端连接使用 
* ETCDCTL_API=3 ./etcdctl put "name" "fengjun"
* ETCDCTL_API=3 ./etcdctl get "name"

## etcd特性总结 
* 对于key前缀相同的有序排列存储在系统中

> 例如:
> * ETCDCTL_API=3 ./etcdctl put "/cron/jobs/job1" "{...job1}"
  * ETCDCTL_API=3 ./etcdctl put "/cron/jobs/job2" "{...job2}"

* 然后可以按照key前缀设置来查找集合
> ETCDCTL_API=3 ./etcdctl get "/cron/jobs/" --prefix
	* /cron/jobs/job1
	* {...job1}
	* /cron/jobs/job2
	* {...job2}

* 在新开的终端会话界面使用watch命令检测 进入解压后的文件夹中
> ETCDCTL_API=3 ./etcdctl watch "/cron/jobs/" --prefix
	* ETCDCTL_API=3 ./etcdctl put "/cron/jobs/job2" "{...1111}" //修改
	* 新开的终端会显示:
				PUT
				/cron/jobs/job2
				{...1111}

* 获取etcd golang客户端的代码 在上面打开的etcd[github项目代码](https://github.com/etcd-io/etcd)里面 [clientv3](https://github.com/etcd-io/etcd/tree/master/clientv3)
> etcd/clientv3 is the official Go etcd client for v3.
	golang安装客户端代码方式
	Install
	go get github.com/coreos/etcd/client  //或者使用gopm工具 或者进入golang中国下载该包 然后入项目中


