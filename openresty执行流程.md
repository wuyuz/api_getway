## openresty执行流程



我们先看个例子

```nginx
# 在nginx.conf中
location /test {
    set $a 32;
    echo $a;
    set $a 56;
    echo $a;   # 字面上我们理解会输出32  56，但是实际上输出56 56
}
# echo nginx第三方模块，是用于做响应输出，只输出了两个 56
```

Nginx 处理每一个用户请求时，都是按照若干个不同阶段依次处理的。而不是根据配置文件上的顺序。

之上的例子 涉及到了 两个阶段  rewrite和content阶段

```nginx
set属于rewrite阶段
echo属于content阶段

而且 rewrite阶段的指令 在 content阶段指令 之前执行。实际的执行顺序应当是：
set $a 32;
set $a 56;
echo $a;
echo $a;

所以输出 56
```

在我们配置文件中执行的指令，可以理解为对我们的执行阶段进行入栈



#### 阶段划分

Nginx处理请求的过程一共划分为11个阶段，按照执行顺序依次是post-read、server-rewrite、
find-config、rewrite、post-rewrite、 preaccess、access、post-access、try-files、content、log.

1. **post-read**
   读取请求内容阶段，nginx读取并解析完请求头之后就立即开始运行；
   例如模块 ngx_realip 就在 post-read 阶段注册了处理程序，
   它的功能是迫使 Nginx 认为当前请求的来源地址是指定的某一个请求头的值。

2. **server-rewrite**
   server请求地址重写阶段；当ngx_rewrite模块的set配置指令直接书写在server配置块中时，
   基本上都是运行在server-rewrite 阶段

3. **find-config**
   配置查找阶段，这个阶段并不支持Nginx模块注册处理程序，
   而是由Nginx核心来完成当前请求与location配置块之间的配对工作。

4. **rewrite**
   location请求地址重写阶段，当ngx_rewrite指令用于location中，就是再这个阶段运行的；
   另外ngx_set_misc(设置md5、encode_base64等)模块的指令，
   还有ngx_lua模块的set_by_lua指令和rewrite_by_lua指令也在此阶段。

5. **post-rewrite**
   请求地址重写提交阶段，当nginx完成rewrite阶段所要求的内部跳转动作，如果rewrite阶段有这个要求的话；

6. **preaccess**
   访问权限检查准备阶段，ngx_limit_req和ngx_limit_zone在这个阶段运行，
   ngx_limit_req可以控制请求的访问频率，ngx_limit_zone可以控制访问的并发度；

7. **access**
   访问权限检查阶段，标准模块ngx_access、第三方模块ngx_auth_request以及第三方模块ngx_lua的access_by_lua
   指令就运行在这个阶段。配置指令多是执行访问控制相关的任务，如检查用户的访问权限，检查用户的来源IP是否合法；

8. **post-access**
   访问权限检查提交阶段；主要用于配合access阶段实现标准ngx_http_core模块提供的配置指令satisfy的功能。
   satisfy all(与关系),satisfy any(或关系)

9. **try-files**
   配置项try_files处理阶段；专门用于实现标准配置指令try_files的功能,
   如果前 N-1 个参数所对应的文件系统对象都不存在，
   try-files 阶段就会立即发起“内部跳转”到最后一个参数(即第 N 个参数)所指定的URI.

10. **content**
    内容产生阶段，是所有请求处理阶段中最为重要的阶段，
    因为这个阶段的指令通常是用来生成HTTP响应内容并输出 HTTP 响应的使命；

11. **log**
    日志模块处理阶段；记录日志

##### 总结

```nginx
NGX_HTTP_POST_READ_PHASE:
    #读取请求内容阶段
NGX_HTTP_SERVER_REWRITE_PHASE:
    #Server请求地址重写阶段
NGX_HTTP_FIND_CONFIG_PHASE:
    #配置查找阶段:
NGX_HTTP_REWRITE_PHASE:
    #Location请求地址重写阶段，常用
NGX_HTTP_POST_REWRITE_PHASE:
    #请求地址重写提交阶段
NGX_HTTP_PREACCESS_PHASE:
    #访问权限检查准备阶段
NGX_HTTP_ACCESS_PHASE:
    #访问权限检查阶段，常用
NGX_HTTP_POST_ACCESS_PHASE:
    #访问权限检查提交阶段
NGX_HTTP_TRY_FILES_PHASE:
    #配置项try_files处理阶段
NGX_HTTP_CONTENT_PHASE:
    #内容产生阶段 最常用
NGX_HTTP_LOG_PHASE:
    #日志模块处理阶段 常用
```

到此，应该明白Nginx 的conf中指令的书写顺序和执行顺序是两码事，切记。有些阶段是支持 Nginx 模块注册处理程序，有些阶段并不可以。最常用的是 rewrite阶段，access阶段 以及 content阶段；

不支持： Nginx 模块注册处理程序的阶段 find-config, post-rewrite, post-access,
主要是 Nginx 核心完成自己的一些逻辑。



#### Nginx下Lua处理阶段与使用范围

openresty发起一个请求时，会有相应的执行流程

![1583496384048](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1583496384048.png)

从图中可知，OpenResty 处理请求大致分为四个阶段：

```
1）初始化阶段（Initialization Phase）
2）重写与访问阶段（Rewrite / Access Phase）
3）内容生成阶段（Content Phase）
4）日志记录阶段（Log Phase）
```



#### 案例分析

我们OpenResty做个测试，示例代码如下：

```nginx
/* 我们通过下面的代码查看执行流程：*/

# 在nginx.conf中配置：
http {
    ...
    init_by_lua 'ngx.log(ngx.ERR, "init_by_lua")'; 
	init_worker_by_lua 'ngx.log(ngx.ERR, "init_worker_by_lua")';
    server {
        listen 8001;
         location /exec {
            rewrite_by_lua 'ngx.log(ngx.ERR, "rewrite_by_lua")';
            set_by_lua $a 'ngx.log(ngx.ERR, "set_by_lua")';
            access_by_lua 'ngx.log(ngx.ERR, "access_by_lua")';
            header_filter_by_lua 'ngx.log(ngx.ERR, "header_filter_by_lua")';
            body_filter_by_lua 'ngx.log(ngx.ERR, "body_filter_by_lua")';
            log_by_lua 'ngx.log(ngx.ERR, "log_by_lua")';
            content_by_lua 'ngx.log(ngx.ERR, "content_by_lua")';
        }
    } 
}
```

这几个阶段的存在，应该是openresty不同于其他多数Web server编程的最明显特征了。
由于nginx把一个会话分成了很多阶段，这样第三方模块就可以根据自己行为，挂载到不同阶段进行处理达到目的。

这样我们就可以根据我们的需要，在不同的阶段直接完成大部分典型处理了。



指令可以在http、server、server if、location、location if几个范围进行配置：

**指令**                         **所处处理阶段**         **使用范围**             **解释**

init_by_lua
init_by_lua_file             loading-config         http                 
nginx Master进程加载配置时执行；通常用于初始化全局配置/预加载Lua模块（Master进程启动时）



init_worker_by_lua
init_worker_by_lua_file     starting-worker     http                 
每个Nginx Worker进程启动时调用的计时器，如果Master进程不允许则只会在init_by_lua之后调用；
通常用于定时拉取配置/数据，或者后端服务的健康检查（工作进程启动时）



set_by_lua
set_by_lua_file             rewrite         server,server if,location,location if
设置nginx变量，可以实现复杂的赋值逻辑；此处是阻塞的，Lua代码要做到非常快；



rewrite_by_lua
rewrite_by_lua_file         rewrite tail     http,server,location,location if
rrewrite阶段处理，可以实现复杂的转发/重定向逻辑；



access_by_lua
access_by_lua_file             access tail     http,server,location,location if     
请求访问阶段处理，用于访问控制（网关、限流、等）



content_by_lua
content_by_lua_file         content         location，location if
内容处理器，接收请求处理并输出响应



header_filter_by_lua
header_filter_by_lua_file     output-header-filter     http，server，location，location if
响应 HTTP过滤处理(例如添加头部信息)，设置header和cookie



body_filter_by_lua
body_filter_by_lua_file     output-body-filter         http，server，location，location if
响应 BODY过滤处理(例如完成应答内容统一成大写) 对响应数据进行过滤，比如截断、替换。




log_by_lua
log_by_lua_file             log                     http，server，location，location if
响应完成后本地异步完成日志记录(日志可以记录在本地，还可以同步到其他机器)
阶段处理，比如记录访问量/统计平均响应时间

实际上我们只使用其中一个阶段content_by_lua，也可以完成所有的处理。但这样做，
会让我们的代码比较臃肿，越到后期越发难以维护。把我们的逻辑放在不同阶段，分工明确，代码独立