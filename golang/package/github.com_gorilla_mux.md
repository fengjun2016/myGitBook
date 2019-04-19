# 关于github.com/gorilla/mux包学习记录
* 包相关简介
* 包安装命令
* 包相关用法
* 包相关注意事项

## 包相关简介
> gorilla/mux
	Package gorilla/mux implements a request router and dispatcher for matching incoming requests to their respective handler.

	The name mux stands for "HTTP request multiplexer". Like the standard http.ServeMux, mux.Router matches incoming requests against a list of registered routes and calls a handler for the route that matches the URL or other conditions. The main features are:

	It implements the http.Handler interface so it is compatible with the standard http.ServeMux.
	Requests can be matched based on URL host, path, path prefix, schemes, header and query values, HTTP methods or using custom matchers.
	URL hosts, paths and query values can have variables with an optional regular expression.
	Registered URLs can be built, or "reversed", which helps maintaining references to resources.
	Routes can be used as subrouters: nested routes are only tested if the parent route matches. This is useful to define groups of routes that share common conditions like a host, a path prefix or other repeated attributes. As a bonus, this optimizes request matching.

## 包安装命令
> go get -u github.com/gorilla/mux 

## 包相关用法

### 用法举例
```golang
func main() {
	r := mux.NewRouter()
	r.HandleFunc("/", HomeHandler)
	r.HandleFunc("/products", ProductsHandler)
	r.HandleFunc("/articles", ArticlesHandler)
	http.Handle("/", r)
}
```
> 上面这里注册了很多url到对用的handlers, 这个做法是和http.HandleFunc()工作情况是类似的:如果一个进来的url请求匹配到了路径当中其中的一个,相关的handler就会使用http.ResponseWriter和`*http.Resqurst`作为参数被调用执行;

> 路径里面也可以传递变量参数, 他们可以使用{name}或者{name:pattern}这样的格式来定义变量传递的参数.如果一个正则表达式没有被定义, 这个匹配的变量将会是任何除了next slash, 例如下面的代码演示:
```golang
r := mux.NewRouter()
r.HandleFunc("/products/{key}", ProductHandler)
r.HandleFunc("/articles/{category}/", ArticlesCategoryHandler)
r.HandlerFunc("/articles/{category}/{id:[0-9]+}", ArticlesHandler)
```
> 如果想要获取URL 请求里面的变量参数 需要使用mux.Vars(`*http.Request`) 来将参数变量创建和存储在一个map当中 请看下面的代码演示:
```golang
func ArticlesCategoryHandler(w http.ResponseWriter, r *http.Request) {
	vars := mux.Vars(r)
	w.WriteHeader(http.StatusOk)
	fmt.Fprintf(w, "Category: %v\n", vars["category"])
}
```

#### 这里路由还可以限制url请求的域名地址或者使用正则表达来限制符合规则的域名才可以请求它 比如请看下面的代码演示:
```golang
r := mux.NewRouter()
// only matches if domain is "www.example.com"
r.Host("www.example.com")
//Matches a dynamic subdomain
r.Host("{subdomain:[a-z]+}.example.com")
```

> 还可以设置统一的一个路由前缀 请看下面的代码演示:
```golang
r := mux.NewRouter()
r.pathPrefix("/products/")
```

> 还可以设置http的请求方法
r.Methods("GET", "POST")

> 或者还可以设置url http协议模式
r.Schemes("https")

> 或者可以设置 header values 
r.Headers("X-Requested-With", "XMLHttpRequest")

> 或者还可以设置query values 
r.Queries("key", "value")

> 或者还可以使用自定义的handler处理函数
```golang
r.MatcherFunc(func(r *http.Request, rm *RouteMatch) bool {
	return r.ProtoMajor == 0
})
```

> 并且也可以将多个匹配器设置在一个单一的路由上面, 代码演示如下:
```golang
r.HandleFunc("/products", ProductsHandler).
	Host("www.example.com").
	Methods("GET").
	Schemes("http")
```

> 并且还可以设置一个group路由设置规则 因为在某种情况下 可能会有多个路由需要相同的依赖设置,在官方文档里这被叫做 subrouting
```golang
r := mux.NewRouter()
s := r.Host("www.example.com").Subrouter()

//然后在s上面注册多个路由规则 比如下面这些的路由规则
s.HandleFunc("/products/", ProductsHandler)
s.HandleFunc("/products/{key}", ProductsHandler)
s.HandleFunc("/articles/{category}/{id:[0-9]+}", ArticleHandler)
```