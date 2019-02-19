# 打点服务器实现 需要nginx开启相关的模块有
* ngx_http_empty_gif_module 模块
> 用过百度统计的兄弟有没有注意到百度使用1x1的空白图片传递统计参数，自己做异步统计的兄弟是否使用静态文件来传递参数。为什么使用空白图片呢而不是自己存
放一张小图呢，nginx里面的空白图片是保存在内存中的，速度绝对比硬盘上读取的快. 看下如何使用empty_gif生成响应1x1的空白图片吧.或许哪天ttlsa自己要做统计，咱们也可以使用empty_gif来传递参数，说归说，肯定性还是比较小，能用第三方的统计就用第三方统计。好了，进入正题吧。nginx默认内置ngx_http_empty_gif_module模块, 如何安装nginx我不在多讲.直接看下empty_gif的用法


> 这个模块，可以默认输出一个空白的gif，某个png透明的插件要用到这种透明的gif文件，简单看了下，配置方法很简单：
![Image text](https://raw.githubusercontent.com/fengjun2016/myGitBook/master/img_cut/empty_gif.png)
我在我本地的nginx服务器实际配置为:
![Image text](https://raw.githubusercontent.com/fengjun2016/myGitBook/master/img_cut/nginx_empty_gif.png)

> 在做打点服务器get请求的时候 发生了405 nginx NOT ALLOWED的错误
最后修改为的配置是：
![Image text]()

* 借助nginx access.log记录打点请求 
> 性能开销最小，最佳方案
  记住打开log_format main 建立一个main格式的log日志记录方式
  还有一点关于gzip on;是否打开  对于返回大的数据 压缩效果明显 而对于本次场景里面的nginx只返回1个像素点的大小的图片的时候 ，其实并没有多大的影响 所以这里是无需打开的 
  gzip对于返回大文件的时候 压缩效果明显 但是需要cpu消耗计算


> 备注 对于打点服务器 一台最好只做这一件事即可 因为本身打点在正常业务场景下面会有很大的访问量 如果同时还有其他程序和业务的时候 会导致程序和服务器承受不住这么大的压力