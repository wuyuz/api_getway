## Lua发http请求

​	有些场景是需要nginx进行请求转发的，用户浏览器请求url访问到nginx服务器，但此请求业务需要再次请求其他业务，如用户请求订单服务获取订单详情，可订单详情中需要返回商品信息，也就需要再请求商品服务获取信息；这样nginx需要有发起http请求的能力。

nginx服务器发起http请求分为内部请求和外部请求

![1583482441866](D:\知识点复习\api网关\assets\1583482441866.png)

- 内部请求： capture函数

  ```lua
  res = ngx.location.capture(uri,{参数})
  
  --options可以传参和设置请求方式
  local res = ngx.location.capture("/product",{
      method=ngx.HTTP_GET --请求方式
      args = {a=2,b=3}  -- 参数
      body = "q=3&d=5" -- post方式传参
  })
  
  -- res的数据结构
  res.status -- 状态响应
  res.header --响应头
  res.body -- 响应体
  ```

  内部请求案例：配置nginx.conf文件

  ```lua
  		location /product {
  			content_by_lua_block {
  				ngx.req.read_body();
  				local args = ngx.req.get_post_args();
  				ngx.print(tonumber(args.a) + tonumber(args.b))
  			}
  		}
  
  		location /order {
  			content_by_lua_block {
  				local res = ngx.location.capture("/product",{
  						method = ngx.HTTP_POST,
  						args = {a=2,b=3}, 
  						body = "a=3&b=5" 
  					});
  			   ngx.say(res.status);
  			   ngx.say(res.body);
  			}
  		}
  ```

- 并发请求： capture_multi(), 多个请求同时发出 

  ```
  语法： res1, res2,... = ngx.location.capture_multi({
      {uri,option?（参数）},
      {uri,option?（参数）},
  })
  ```

  案例： 并发请求，在nginx.conf中

  ```nginx
  location = /sum {
      internal;
      content_by_lua_block {
          ngx.sleep(0.1)
          local args = ngx.req.get_uri_args()
          ngx.print(tonumber(args.a) + tonumber(args.b))
      }
  }
  
  location = /subduction {
      internal;
      content_by_lua_block {
          ngx.sleep(0.1)
          local args = ngx.req.get_uri_args()
          ngx.print(tonumber(args.a) - tonumber(args.b))
      }
  }
  
  location = /app/test_multi {
      content_by_lua_block {
          local start_time = ngx.now()
          local res1, res2 = ngx.location.capture_multi({
              {"/sum",{args={a=3,b=8}}},
              {"/subduction",{args={a=3,b=8}}}
          })
          ngx.say("status:",res1.status, "response:",res1.body)
          ngx.say("status:",res2.status, "response:",res2.body)
          ngx.say("time used: ",ngx.now() - start_time)
      }
  }
  
  location = /app/test_queue {
      content_by_lua_block {
          local start_time = ngx.now()
          local res1 = ngx.location.capture("/sum",{
          	args={a=3,b=8}
          })
          local res2 = ngx.location.capture("/subduction",{
          	args={a=3,b=8}
          })
          ngx.say("status:",res1.status, "response:",res1.body)
          ngx.say("status:",res2.status, "response:",res2.body)
          ngx.say("time used: ",ngx.now() - start_time)
      }
  }
  ```

- 外部请求： ngx.location.capture 不能直接发起外部请求，我们需要通过内部请求中用反向代理请求处理外部请求, 使用proxy_pass相外部发起请求

  ```nginx
  location /product {
      internal;
      proxy_pass "https://s.taobao.com/search?q=hello"; # 外部请求转发
  }
  
  location /order {
      content_by_lua_block {
          local res = ngx.location.capture("/produce");
          ngx.say(res.status)
          ngx.say(res.body)
      }
  }
  ```

  动态传参

  ```nginx
  location /product {
      internal;
      proxy_set_header Accept_Encoding ' '  # 不压缩
      proxy_pass "https://s.taobao.com/search?q=$arg_q";  # 使用全局变量
  }
  
  location /order {
      content_by_lua_block {
          local get_args = ngx.req.get_uri_args();
          local res = ngx.location.capture("/produce",{
              method = ngx.HTTP_GET,
                  args = {q=get_args["q"]}
          });
          ngx.say(res.status)
          ngx.say(res.body)
      }
  }
  ```



#### http模块

openresty默认没有提供http客户端，需要使用第三方提供，我们可以从github上搜索相应的客户端，比如：https://github.com/pintsized/lua-resty-http, 只要将lua-resty-http/lib/resty目录下的http.lua和http_headers.lua两个文件拷贝到/usr/local/openresty/lualib/resty目录下即可（假设openresty目录在此）

```shell
cd /usr/local/openresty/lualib/resty

wget https://raw.githubusercontent.com/ledgetech/lua-resty-http/tree/master/lib/resty/http_headers.lua
wget https://github.com/pintsized/lua-resty-http/master/lib/resty/http.lua

# 上面的地址找不到，可以再：https://github.com/ledgetech/lua-resty-http.git
```

- 实例： 使用http模块

  ```lua
  -- 引入http模块
  local http = require("resty.http")
  --创建http客户端
  local httpc = http.new()
  
  --request_uri 函数请求淘宝
  local resp, err = httpc:request_uri("https://list.tmall.com",{
  	method = "GET",    ---请求方式
      --path="/search_product.htm?q=ipone",
      query="q=iphone",    ---get方式传参数
      body="name='jack'&age=18",    ---post方式传参数
      path="/search_product.htm",    ----路径
      ---header参数
  	keepalive=false,
      headers = {["User-Agent"]="Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.111 Safari/537.36"}
  })
  
  if not resp then
  	ngx.say("request error: ",err)
  	return
  end
  
  -- 获取状态码
  ngx.status = resp.status
  if ngx.status ~= 200 then
  	ngx.log(ngx.WARN, "非200状态"..ngx.status)
  	return resStr
  end
  
  -- 获取遍历返回的头信息
  for k, v in pairs(resp.headers) do
  	if type(v) == "table" then
  		ngx.log(ngx.WARN, "table:"..k,table.concat(v,","))
  	else
  		ngx.log(ngx.WARN, "one:"..k,table.concat(v,","))
  	end
  end
  
  --响应体
  ngx.say("ok")
  ngx.say(resp.body)
  httpc.close()
  ```

  注意：报错情况

  -  报错：no resolver defined to resolve "taobao.com"，这是因为我们访问淘宝，没有配置DNS：

    ```
    需要再nginx.conf中http模块上加 resolver 8.8.8.8
    ```

  - 报错：unable to get local issuer certificate；访问https时发生错误，因为我们访问的时https，设置了ssl证书

    ```nginx
    在nginx.conf文件中，server虚拟主机模块设置
    lua_ssl_verify_depth 2;
    lua_ssl_trusted_certificate "/etc/ssl/certs?ca-bundle.crt";
    ```

    