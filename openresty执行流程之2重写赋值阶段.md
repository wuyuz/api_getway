## openresty执行流程之2重写赋值阶段



#### 重写赋值阶段

- set_by_lua

  语法：set_by_lua $res <lua-script-str> [$arg1 $arg2 …]
  语境：server、server if、location、location if
  阶段：rewrite

  设置nginx变量，我们用的set指令即使配合if指令也很难实现负责的赋值逻辑；传入参数到指定的lua脚本代码中执行，并得到返回值到res中。<lua-script-str>中的代码可以使从ngx.arg表中取得输入参数(顺序索引从1开始)。

  这个指令是为了执行短期、快速运行的代码因为运行过程中nginx的事件处理循环是处于阻塞状态的。
  耗费时间的代码应该被避免。
  禁止在这个阶段使用下面的API：
  1、output api（ngx.say和ngx.send_headers）；
  2、control api（ngx.exit）；
  3、subrequest api（ngx.location.capture和ngx.location.capture_multi）；
  4、cosocket api（ngx.socket.tcp和ngx.req.socket）；
  5、sleep api（ngx.sleep）

   

  ##### 案例：

  - 简单案例

    ```nginx
    #nginx.conf配置文件
    location /lua {
        set $jump "1";
        echo $jump;
    }
    ```

    set 命令对变量进行赋值，但有些场景的赋值业务比较复杂，需要用到lua脚本
    所以用到set_by_lua

  - 补充知识点：

    ```
    ngx.var.arg与ngx.req.get_uri_args的区别，都是能够获取请求参数
    
    ngx.var.arg_xx与ngx.req.get_uri_args["xx"]两者都是为了获取请求uri中的参数
    例如 http://pureage.info?strider=1
    为了获取输入参数strider，以下两种方法都可以：
    
    local strider = ngx.var.arg_strider
    local strider = ngx.req.get_uri_args["strider"]
    
    差别在于，当请求uri中有多个同名参数时,ngx.var.arg_xx的做法是取第一个出现的值
    ngx.req_get_uri_args["xx"]的做法是返回一个table，该table里存放了该参数的所有值
    
    例如,当请求uri为：http://pureage.info?strider=1&strider=2&strider=3&strider=4时
    
    ngx.var.arg_strider的值为"1",而ngx.req.get_uri_args["strider"]的值为table ["1", "2", "3", "4"]。
    因此，ngx.req.get_uri_args属于ngx.var.arg_的增强。
    ```

  - **案例需求**：书店网站改造把之前的skuid为8位商品，请求到以前的页面，为9位的请求到新的页面
    书的商品详情页进行了改造美化了一下；上线了时候，不要一下子切换美化页面；做AB概念
    把新录入的书的商品 采用 新的商品详情页，之前维护的书的商品详情页 用老的页面，以前书的id 为8位，新的书id为9位

    ```nginx
    # 在 nginx.conf 中
    location /book {
        # set_by_lua， 通过lua脚本文件赋值
        set_by_lua $to_type '
            local skuid = ngx.var.arg_skuid
            ngx.log(ngx.ERR,"skuid=",skuid)
            local r = ngx.re.match(skuid, "^[0-9]{8}$")
            local k = ngx.re.match(skuid, "^[0-9]{9}$")
            if r then
            return "1"
            end;
        if k then
            return "2"
            end;
        ';
    
            if ($to_type = "1") {
            echo "skuid为8位";
            proxy_pass http://127.0.0.1/old_book/$arg_skuid.html;
        }
        if ($to_type = "2") {
            echo "skuid为9位";
            proxy_pass http://127.0.0.1/new_book/$arg_skuid.html;
        }
     }
    ```

  - 编辑html文件

    ```shell
    [root@node5 data]# tree /data/www/html/
    /data/www/html/
    ├── new_book
    │   └── 123456789.html
    └── old_book
        └── 12345678.html
    
    2 directories, 2 files
    [root@node5 data]# cat /data/www/html/old_book/12345678.html
    old book
    [root@node5 data]# cat /data/www/html/new_book/123456789.html
    new book
    
    ------------------------------
    #测试访问：http://10.11.0.215/book?skuid=123456789  --> 会跳转到new_book/123456789.html
    
    访问：http://10.11.0.215/book?skuid=12345678
    会跳转到new_book/12345678.html
    ```

    

- set_by_lua_file

  语法set_by_lua_file $var lua_file arg1 arg2...;

  在lua代码中可以实现所有复杂的逻辑，但是要执行速度很快，不要阻塞；

  ```nginx
  # nginx.conf中
  location /lua_set_1 {
      default_type "text/html";
      set_by_lua_file $num /usr/local/luajit/test_set_1.lua;
      echo $num;
  }
  ```

  编写lua文件： test_set_1.lua

  ```lua
  local uri_args = ngx.req.get_uri_args()
  local i = uri_args["i"] or 0
  local j = uri_args["j"] or 0
  
  return i + j
  ```

  得到请求参数进行相加然后返回。

  访问如http://192.168.31.138/lua_set_1?i=1&j=10进行测试。 如果我们用纯set指令是无法实现的。

  **注意**，这个指令只能一次写出一个nginx变量，但是使用ngx.var接口可以解决这个问题：

  ```nginx
  location /foo {
      set $diff '';
      set_by_lua $sum '
          local a = 32
          local b = 56
          ngx.var.diff = a - b; --写入$diff中
          return a + b;  --返回到$sum中
      ';
      echo "sum = $sum, diff = $diff";
  }
  ```

  

