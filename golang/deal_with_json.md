# 关于最近在golang语言中使用json处理和解析数据所遇到的坑点总结
* tag 注意结构体里面的属性必须首字母要大写才能被导出和解析
* 在使用for range 循环时 应该每一次都声明一个结构体 传入 不能使用一个全局变量 不然会到最后解析出来全是空的值

## tag  注意下面需要导出的属性名称首字母必须都要大写 

```golang
//HTTP接口应答
type Response struct {
	Errno int         `json:"errno"`
	Msg   string      `json:"msg"`
	Data  interface{} `json:"data"`
}
```

> 下面这样的json对象中只会有两种属性 Name Class
```golang
type Stu struct {
    Name  string `json:"name"`
    age   int
    hIgh  bool
    sex   string
    Class *Class `json:"class"`
}
```


## 循环时的注意事项 下面看具体代码 注意观察下面的job变量
```golang
//获取任务列表
func (jM *JobMgr) ListJobs() (jobLists []*common.Job, err error) {
	var (
		dirKey  string
		getResp *clientv3.GetResponse
		kvPair  *mvccpb.KeyValue
		job     *common.Job
	)

	//获取目录key
	dirKey = common.JOB_SAVE_DIR

	//获取任务列表
	if getResp, err = jM.kv.Get(context.TODO(), dirKey, clientv3.WithPrefix()); err != nil {
		return
	}

	//初始化数组空间 len(jobListe) == 0
	jobLists = make([]*common.Job, 0)

	//遍历所有任务，进行反序列化
	for _, kvPair = range getResp.Kvs {
		log.Print(string(kvPair.Value))
		job = &common.Job{}  //如果没有这里 err报错显示 nil *common.Jo
		if err = json.Unmarshal(kvPair.Value, job); err != nil {
			log.Printf("unmarshal err : %v", err)
			err = nil
			continue
		}
		jobLists = append(jobLists, job)
	}

	log.Printf("jm jobs list: %v", getResp.Kvs)
	return
}
```
