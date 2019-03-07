# 源码编译安装常见步骤如下
* 解压源码
* configure
* make
* make install

## 编译前需要安装gcc autoconfig软件 做好准备

## 对于php的源码编译安装命令如下所示 主要是编译成.so文件 
* 解压源码命令
> tar -xjvf php-7.7.7.tar.bz2(你之前下载的当前目录下面的php源码名称)

* configre 进行一些安装的配置
> cd php-7.2.2
  然后找到里面的configure 主要是配置安装目录和一些扩展
  ./confiure --prefix=/usr/local/etc/php(安装目录) 还可以加一些其他的安装配置
> 我在这里出现了build test failed 
  搜了一下发现：网友相关答案如下所列
  * gcc编译系统出了问题 需要重新安装一下gcc *尝试过无用*
  	> brew install gcc -y
  	 
  * 发现是依赖的问题 因为在输出信息的最后发现有一条 说没有libxml2依赖的安装目录 
  	> checking libxml2 install dir... no
	  checking for xml2-config path... /usr/bin/xml2-config
	  checking whether libxml build works... no
	  configure: error: build test failed.  Please check the config.log for details.

    > 解决办法 
      安装依赖 [参考链接](https://mengkang.net/1080.html) https://mengkang.net/1080.html
	  相关依赖有:
	  搜索出来有这些:brew install libjpeg libpng libxml gettext openssl freetype pcre  
	  安装完之后执行
	  ./configure --prefix=/usr/local/etc/php --with-libxml-dir=/usr/local/opt/libxml2 

	 > 其实上面的问题不是没有安装这个依赖的问题 而是可能你安装了这个依赖 但是PHP  configure程序找不到该依赖安装坐在的目录 所以必须要在这里指定最后依赖安装的目录
	  如果你是安装了这个依赖 但是不在path环境变量设置的目录里面也会发生这个问题
	  如果安装在了/usr/local/bin/或者/usr/bin/就不用特殊制定了。
	  所以要注意检查环境变量path有没有将你安装的目录加在path里面 没有则需要自己指定特殊目录
	  echo 'export PATH="/usr/local/opt/libxml2/bin:$PATH"' >> ~/.bash_profile
	  如果安装了zsh的话 则将.bash_profile 换成 .zshrc文件

  
* 执行make命令 编译安装一下 有时候会需要加上sudo 如果此目录用户等级
> make

* 执行make install 
> make install

> 上面这些步骤执行完毕后 一般最后会告诉php相关最后安装的目录以及bin所在的目录
  由于我这里是mac系统 所以系统自带php5.6版本 所以需要将php7安装的bin/php目录加到.bashrc或在.bash_profile文件中
  可以直接使用alias命令来指定一个别名即可 
  export PATH=$HOME/bin:/usr/local/bin:/usr/local/sbin:$PATH
  //下面添加这条语句
  alias php=/usr/local/etc/php/bin/php
  alias php56=/usr/bin/php (这一句其实也可以不需要)
  记得修改之后要执行
  source ~/.zshrc 命令 修改的设置才会生效

 > 这样之后 如果 php -v 就是php7了
   php56 -v 就是php56的版本了

## 注意事项及踩坑
> 安装完后 发现安装目录里面并没有 php.ini文件 需要去源码目录里面cp到安装目录里面才行的
  回答下载解压后的目录
  
  执行 php -i | grep php.ini 查找php.ini应该放在的目录
  Configuration File (php.ini) Path => /usr/local/etc/php/lib 应该放在此目录里面
  要么在源码编译configure的时候加上一行指定ini目录

  cp php.ini-development /usr/local/etc/php/lib/php.ini


## mac上编译安装php扩展记录
> 总的就是进入php源码解压后的ext目录里面 然后找到你要安装的扩展，并cd进去,然后执行
* phpize
* ./configure --with-php-config=/usr/local/etc/php/bin/php-config (备注:有的这里还需要加其他的配置)
* make
* sudo make install
* 然后使用php --ini 找到php.ini文件 在文件使用shift+g定位至文件末尾 添加extension = "扩展名.so"


### php编译安装mbstring.so扩展 命令
* 找到之前下载解压后的php源码扩展目录 我的是/Users/fengjun/Downloads/php-7.3.2/ext cd进入该目录
* 然后cd mbstring
* 执行phpize
* 然后执行 ./configure --with-php-config=/usr/local/etc/php/bin/php-config
* make
* sudo make install
* 编辑php.ini 添加extendsion="mbstring.so"

### php编译安装openssl.so扩展
* 找到之前下载解压后的php源码扩展目录 我的是/Users/fengjun/Downloads/php-7.3.2/ext cd进入该目录
* 然后cd openssl
* mv config0.m4 config.m4 (这一步比较特殊)
* 执行phpize
* 然后执行 ./configure --with-php-config=/usr/local/etc/php/bin/php-config
> 如果只是单纯执行上面的命令会报错
./php_openssl.h:30:10: fatal error: 'openssl/opensslv.h' file not found
#include <openssl/opensslv.h>
         ^~~~~~~~~~~~~~~~~~~~
1 error generated.
make: *** [openssl.lo] Error 1

* mac上找不到这个头文件 注意只是mac上才有这个问题哦 解决办法是找你mac上openssl的安装目录 由于我是brew安装的 所以在/usr/local/opt里面可以找到

* 重新执行 ./configure --with-php-config=/usr/local/etc/php/bin/php-config --with-openssl=/usr/local/opt/openssl

* make
* sudo make install
* 编辑php.ini 添加extendsion="openssl.so"


### php编译安装pdo_mysql扩展
* 找到之前下载解压后的php源码扩展目录 我的是/Users/fengjun/Downloads/php-7.3.2/ext cd进入该目录
* 然后cd pdo_mysql
* 执行phpize
* 然后执行 ./configure --with-php-config=/usr/local/etc/php/bin/php-config --with-pdo-mysql=/usr/local/Cellar/mysql/5.7.20 (这里是我mysql在mac上的安装目录 一定要加上这一项要求)
* make
* sudo make install
* 编辑php.ini 添加extendsion="pdo_mysql.so"


