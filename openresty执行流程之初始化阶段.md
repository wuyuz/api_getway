## openresty执行流程之初始化阶段



#### 初始化阶段

- 初始化阶段：引入全局变量

  init_by_lua   init_by_lua_block     init_by_lua_file
  语法：init_by_lua <lua-script-str>
  语境：http
  阶段：loading-config
  当nginx master进程在加载nginx配置文件时运行指定的lua脚本，通常用来注册lua的全局变量或在服务器启动时预加载lua模块：nginx.conf配置

  ```nginx
  worker_processes  4;
  
  error_log logs/error.log;
  error_log logs/debug.log debug;
  
  events {
      worker_connections  1024;
  }
  
  http {
      include       mime.types;
      #default_type  application/octet-stream;
      default_type  text/html;
      charset utf-8;
      sendfile        on;
      # 关闭lua缓存，不需要每次重启才生效，会牺牲一定性能
      lua_code_cache on;
  
      lua_shared_dict shared_data 10m;
      keepalive_timeout  65;
  
      init_by_lua_block {  # 常用作引入全局变量
      	cjson = require "cjson"
      }
  
      server {
          listen       80;
          server_name  www.server1.com;
          resolver 8.8.8.8;
          lua_ssl_verify_depth 2;
      	lua_ssl_trusted_certificate "/etc/ssl/certs/ca-bundle.crt";
      	location = /api {
              content_by_lua_block {
                  ngx.say(cjson.encode({dog = 5, cat = 6}))
              }
          }
  
          location / {
              root html;
              index index.html;
          }
  
          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
                  root   html;
              }
          }
    }
  ```

  从这段配置代码，我们可以看出，其实这个指令就是初始化一些lua的全局变量，以便后续的代码使用。

  还可以初始化lua_shared_dict共享数据：

  ```nginx
  http {
      lua_shared_dict dogs 1m;
      init_by_lua_block {
          local dogs = ngx.shared.dogs;
          dogs:set("Tom", 50)
  		dogs:set("flag",1)
       }
      server {
          location = /api {
              content_by_lua_block {
                  local dogs = ngx.shared.dogs;
                  ngx.say(dogs:get("Tom"))
              }
          }
  	}
  }
  ```

  lua_shared_dict的内容不会在nginx reload时被清除。所以如果你不想在init_by_lua中重复初始化共享数据，
  那么你需要在你的共享内存中设置一个标志位并在init_by_lua中进行检查。

  因为这个阶段的lua代码是在nginx forks出任何worker进程之前运行，
  数据和代码的加载将享受由操作系统提供的copy-on-write的特性，从而节约了大量的内存。
  不要在这个阶段初始化你的私有lua全局变量，因为使用lua全局变量会照成性能损失，
  并且可能导致全局命名空间被污染。这个阶段只支持一些小的LUA Nginx API设置：ngx.log和print、ngx.shared.DICT；

  

- init_worker_by_lua
  语法：init_worker_by_lua <lua-script-str>
  语境：http
  阶段：starting-worker
  在每个nginx worker进程启动时调用指定的lua代码。常用在定时任务

  用于启动一些定时任务，比如心跳检查，定时拉取服务器配置等等；此处的任务是跟Worker进程数量有关系的，比如有2个Worker进程那么就会启动两个完全一样的定时任务。

  - 在nginx.conf中的http模块中

    ```
    init_worker_by_lua_file /usr/local/lua/init_worker.lua;
    ```

  - /usr/local/lua/init_worker.lua 文件如下

    ```lua
    local count = 0
    local delayInSeconds = 3
    local heartbeatCheck = nil
    
    heartbeatCheck = function(args)
       count = count + 1
       ngx.log(ngx.ERR, "do check ", count)
    
       local ok, err = ngx.timer.at(delayInSeconds, heartbeatCheck)
    
       if not ok then
          ngx.log(ngx.ERR, "failed to startup heartbeart worker...", err)
       end
    end
    
    heartbeatCheck()
    ```

  - 观察日志：

    ```
    2019/08/23 19:09:21 [error] 77620#0: *436 [lua] init_worker.lua:7: do check 2, context: ngx.timer
    2019/08/23 19:09:21 [error] 77617#0: *438 [lua] init_worker.lua:7: do check 2, context: ngx.timer
    2019/08/23 19:09:21 [error] 77618#0: *437 [lua] init_worker.lua:7: do check 2, context: ngx.timer
    2019/08/23 19:09:21 [error] 77619#0: *435 [lua] init_worker.lua:7: do check 2, context: ngx.timer
    2019/08/23 19:09:24 [error] 77619#0: *439 [lua] init_worker.lua:7: do check 3, context: ngx.timer
    2019/08/23 19:09:24 [error] 77617#0: *442 [lua] init_worker.lua:7: do check 3, context: ngx.timer
    2019/08/23 19:09:24 [error] 77620#0: *441 [lua] init_worker.lua:7: do check 3, context: ngx.timer
    ```

    ngx.timer.at：延时调用相应的回调方法；ngx.timer.at(秒单位延时，回调函数，回调函数的参数列表)；
    可以将延时设置为0即得到一个立即执行的任务，任务不会在当前请求中执行不会阻塞当前请求，
    而是在一个轻量级线程中执行。

    ```
    另外根据实际情况设置如下指令
    lua_max_pending_timers 1024;  #最大等待任务数
    lua_max_running_timers 256;    #最大同时运行任务数
    
    ```

    



- 
  lua_package_path

  语法：lua_package_path <lua-style-path-str>

  默认：由lua的环境变量决定，我们之前在系统环境变量中设置过了
  适用上下文：http
  设置lua代码的寻找目录。
  例如：lua_package_path "/opt/nginx/conf/www/?.lua;;";