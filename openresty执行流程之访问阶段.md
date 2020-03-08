## openresty执行流程之访问阶段

#### 访问阶段

用途：访问权限限制 返回403

nginx的指令：allow 允许，deny 禁止

```
allow ip
deny ip
```

涉及到的网关，有很多的业务 都是在access阶段处理的，有复杂的访问权限控制；nginx：allow deny 功能太弱

- access_by_lua
  语法：access_by_lua <lua-script-str>
  语境：http,server,location,location if
  阶段：access tail
  为每个请求在访问阶段的调用lua脚本进行处理。主要用于访问控制，能收集到大部分的变量。

  用于在 access 请求处理阶段插入用户 Lua 代码。这条指令运行于 access 阶段的末尾，
  因此总是在 allow 和 deny 这样的指令之后运行，虽然它们同属 access 阶段。

  ```nginx
  location /foo {
    access_by_lua_block {
      ngx.log(ngx.DEBUG,"12121212");
    }
    allow 10.11.0.215;
    echo "access";
  }
  ```

  access_by_lua 通过 Lua 代码执行一系列更为复杂的请求验证操作，比如实时查询数据库或者其他后端服务，
  以验证当前用户的身份或权限。

  案例：利用 access_by_lua 来实现 ngx_access 模块的 IP 地址过滤功能：

  ```nginx
  location /access {
      access_by_lua_block {
          if ngx.var.arg_a == "1" then
            return
          end
          if ngx.var.remote_addr == "10.11.0.215" then
            return
          end
          ngx.exit(403)
      }
  
      echo "access";
  }
  
  #对于限制ip的访问，等价于
  location /hello {
    allow 10.11.0.215;
    deny all;
    echo "hello world";
  }
  ```

  

- access_by_lua_file

  nginx.conf 配置文件

  ```
  location /lua_access {
    access_by_lua_file /usr/local/luajit/test_access.lua;
    echo "access";
  }
  ```

- test_access.lua

  ```lua
  if ngx.req.get_uri_args()["token"] ~= "123" then
     return ngx.exit(403)
  end
  
  --即如果访问如http://10.11.0.215/lua_access?token=234将得到403 Forbidden的响应。这样我们可以根据如cookie/用户token来决定是否有访问权限。  
  ```

  
  