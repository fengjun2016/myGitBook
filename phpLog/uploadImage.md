# php laravel54框架和wangeditor配合使用来解决图片上传功能

* 第一步引入wangeditor相关的css和js，按照官方文档要求

* 第二步 自己写js中定义代码如下:

```javascript
var editor = new wangEditor('content');

editor.config.uploadImgUrl = '/posts/image/upload';

// 设置 headers（举例）
editor.config.uploadHeaders = {
    'X-CSRF-TOKEN' : $('meta[name="csrf-token"]').attr('content')
};

editor.create();
```

* 第三步 按照官方文档要求解决图片上传之后保存的路径问题

> 由于一般在laravel中一般是存储在storage\app\public目录里面,这里需要修改laravel相关的配置

> 修改config/filesystems.php中默认驱动为public,'default' => env('FILESYSTEM_DRIVER', 'public'),
然后执行php artisan storage:link 建立public目录到storage\app\public的软连接，因为真正的图片是存储在后面这个目录里面的