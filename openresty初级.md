## openresty初级

#### Nginx 优势

​	Nginx设计为一个主进程多个工作进程的工作模型，每个进程是单线程处理多个连接，而且每个工作进程采用了非阻塞I/O模型来处理多个连接，从而减少了线程上下文切换，实现了公认的高性能、高并发；在生产环境中会通过把cpu绑定给Nginx工作进程从而提升其性能；另外因为单线程工作模式的特点，内存占用少。

Nginx模块也是非常多，功能也很强劲，不仅可以作为http负载均衡，还可以实现内容缓存，web服务器，反向代理，访问控制等功能（Nginx的rewrite模块经常用到）



#### 什么是ngx_lua

​	ngx_lua 是Nginx的一个模块，将lua嵌入到Nginx中，从而可以使用lua来编写脚本，这样可以使用lua编写应用脚本，部署到Nginx中运行，即Nginx变成一个web容器；这样开发人员可以使用lua语言开发  能web应用

​	lua是一个轻量级、可嵌入式的脚本语言，这样可以非常容易的嵌入到其他语言中使用。另外lua提供了协程并发，即以同步调用的方式进行异步执行，从而实现并发，比起回调机制的并发来说代码更容易编写和理解，排查问题也很容易，lua还提供了闭包机制，函数可以作为First class value 进行参数残敌，另外其是实现了标记清除垃圾收集（因为lua的小巧轻量级，可以在Nginx中嵌入Lua VM，请求的时候创建VM，请求结束回收VM）

```
我们在做web开发的时候，也有一个重要的脚本语言，javascript，也就是js文件；区别于node.js是服务器语言，而js文件是客户端脚本语言，每个浏览器中自带了javascript的引擎，可以解析js文件

lua -- javascript一样，常用于游戏应用中
```

##### ngx_lua模块的工作原理

​	ngx_lua将lua嵌入nginx中，能够让nginx运行lua脚本，而且高并发，非阻塞的处理各种请求，lua内建协程，这样就能够非常好的将异步回调转换成顺序调用的形式。ngx_lua在lua中进行的io操作会托付给nginx的事件模型。从而实现非阻塞调用。开发人员能后用串行的方式码，ngx_lua会自己主动的在进行io操作时中断，保存上下文；然后将io操作托付给nginx事件处理机制，在io操作完毕后，ngx_lua会恢复上下文，程序继续运行，每个nginxworker进程持有一个lua解释器或者luajit实列，被这个worker处理的全部请求共享这个实例。

​	每个请求的Context会被lua轻量级的协程切割，从而保证各自请求是独立的，nginx_lua采用"one-coroutine-per-request"的处理模型，对于每个用户请求，ngx_lua会唤醒一个协程用于执行用户代码处理请求，当请求处理完毕这个协程会被销毁。

​	每个协程都有一个独立的全局变量（变量空间），继承于全局共享的，仅仅读的”comman data"。所以被用户代码注入全局变量的不论什么变量都不会影响其他请求的处理，而且这些变量在请求处理完毕后会被释放，这样就保证了全部用户代码执行字一个“sandbox"(沙箱）,这个沙箱与请求具有同样的生命周期，得益于lua协程的支持。ngx_lua在处理10000个并发请求时仅仅需要非常少的内存，测试中，ngx_lua处理每个请求仅仅需要2kb内存。



##### ngx_lua的安装

- 类似于echo模块的安装，但是这种安装方式比较复杂

- 上面的方式是通过下载模块的源码文件，编译nginx。可是推荐使用openresty，openresty就是一个打包程序，包括大量的第三方nginx模块，比方HttpLuaModule、HttpRedis2Module、HttpEchoModule等，省去下载模块。

  ```
  openresty将nginx核心、luajIT(lua引擎)、许多有用的lua库和Nginx第三方模块打包在一起；这样开发人员只需要安装openresty，不需要了解nginx核心和复杂的C/C++模块就可以了，只需要使用lua语言进行web应用开发。
  
  openresty提供了一些常用的ngx_lua开发模块，如：
  	lua-resty-memcached
  	lua-resty-mysql
  	lua-resty-redis
  	lua-resty-dns
  	lua-resty-limit-traffic
  	lua-resty-template
  这些模块涉及到mysql、redis、限流、模块渲染常用组件
  ```



#### openresty安装

- 预备条件

  ```
  yum install readline-devel pcre pcre-devel openssl openssl-devel gcc curl GeoIP-devel
  ```

- 下载源码包

  ```shell
  wget https://openresty.org/download/ngx_openresty-1.9.7.1.tar.gz   # 下载
  tar xzvf ngx_openresty-1.9.7.1.tar.gz       # 解压
  cd ngx_openresty-1.9.7.1/ 
  
  ./configure --with-luajit --with-pcre --with-http_gzip_static_module --with-http_realip_module --with-http_geoip_module --with-http_ssl_module --with-http_stub_status_module   # 选择安装或不安装的模块
  
  
  make 
  make install
  
  [root@MyHost ngx_openresty-1.9.7.1]#./configure --help  # 可以查看提供了哪些模块
    --prefix=PATH        set the installation prefix (default to /usr/local/openresty)，安装位置
    --without-http_echo_module         disable ngx_http_echo_module 默认是开启的，这里可关闭
    --with-luajit                      enable and build the bundled LuaJIT 2.1 (the default) lua及时编译
  ```

  选择的模块介绍

  ```
  --with-luajit  lua及时编译模块
  --with-pcre    设置pcre库
  --with-http_gzip_static_module    静态文件压缩
  --with-http_realip_module      通过这个模块允许我们改变客户端请求头中的客户端ip地址（如x-real-IP或x-Forward-for 意义在于能够使得后台服务器记录原始客户端IP）
  
  --with-http_geoip_module   增加了根据ip获取城市信息，经纬度
  --with-http_ssl_module     使用https协议模块
  --with-http_stub_status_module    监控nginx状态
  ```

  安装好后，默认是在/usr/local/openresty目录下

  luajit 是采用C语言写的lua代码解释器 ---just in time  (即时编译)

  lualib  是编辑好的lua库

  nginx 其实我们openresty就是nginx，只是做了一些模块化工作，所以启动openresty就是启动nginx，直接进入nginx启动

- 测试openresty

  ```nginx
  在conf文件的nginx.conf文件中编写如下代码
  
  	server {
          listen 8080;
          location / {
              default_type text/html;
              content_by_lua '
                  ngx.say("<h1>Hello, World!</h1>")
              ';
          }
      }
  
  启动nginx，在openresty下的nginx下的sbin中启动；访问：http://47.95.217.144:8080/ 
  ```

- 性能测试

  ```
  -- 1. 安装压力测试工具
  $ yum install httpd-tools
  
  -- 2. 测试
  $ ab -c10 -n50000 http://localhost:8080/
  ...
  Concurrency Level:      10
  Time taken for tests:   2.825 seconds
  Complete requests:      50000
  Failed requests:        0
  Write errors:           0
  Total transferred:      8050000 bytes
  HTML transferred:       650000 bytes
  Requests per second:    17697.26 [#/sec] (mean)
  Time per request:       0.565 [ms] (mean)
  Time per request:       0.057 [ms] (mean, across all concurrent requests)
  Transfer rate:          2782.48 [Kbytes/sec] received
  ```

- 安装好后，设置环境变量

  ```shell
  vi /etc/profile
      export NGINX_HOME=/usr/local/openresty/nginx
      export PATH=$PATH:$NGINX_HOME/sbin
      
  source /etc/profile 
  
  
  使用 /etc/profile.d 而不是 /etc/profile 来配置环境变量 Linux?
  在 /etc/profile 这个文件中有这么一段 shell, 会在每次启动时自动加载 profile.d 下的每个配置:
  
  if [ -d /etc/profile.d ]; then
    for i in /etc/profile.d/*.sh; do
      if [ -r $i ]; then
        . $i
      fi
    done
    unset i
  fi
  
  区别:
  1、都用来设置环境变量文件
  
  2、/etc/profile.d/ 高度解耦, 比 /etc/profile 好维护，不想要什么变量直接删除 /etc/profile.d/ 下对应的 shell 脚本即可
  
  3、/etc/profile 和 /etc/profile.d 同样是登录（login）级别的变量，当用户重新登录 shell 时会触发。所以效果一致。
  
  4、设置登录级别的变量，重新登录 shell，或者 source /etc/profile。
  
  需要添加新的环境变量时:
  在 /etc/profile.d/ 目录下新建对应的 sh 文件即可，比如新建 java 的：
  vim /etc/profile.d/kafka.sh
      export KAFKA_HOME=/opt/module/kafka
      export PATH=$PATH:$KAFKA_HOME/bin
  立即刷新使变量可用
  回到上一次目录 source /etc/profile
  查看 echo $KAFKA_HOME
  
  ```

  

#### openresty的lua模块的Helloword

这里使用在conf文件中使用content_by_lua    语法使用lua输出，ngx.say() 函数为要输出的内容

```nginx
	server {
        listen 8080;
        location / {
            default_type text/html;
            content_by_lua '
                ngx.say("<h1>Hello, World!</h1>")
            ';
        }
    }
```

- 使用nginx的内部变量

  ```nginx
  	server {
          listen 8080;
  
          location / {
              default_type text/html;
              content_by_lua '
                  ngx.say("<h1>Hello, World!</h1>")
              ';
          }
  		location /url {
              default_type text/html;
              echo "url:$uri";
  			echo "arg:$args ---- arg-a:$arg_a";
          }
  		
      }
  
  浏览器中访问：http://47.95.217.144:8080/url?a=1&b=5
  url:/url arg:a=1&b=5 ---- arg-a:1
  ```

- set设置的变量，为局部变量，其他请求无法获取，但是可以结合到echo_exec，内部跳转来传递参数

  ```nginx
  location /test {
      set $name "rain";
      echo_exec /test_def;
  }
  
  location /test_def {
      echo $name;
  }
  ```

  

