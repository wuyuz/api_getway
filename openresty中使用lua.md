## openresty中使用lua



#### openresty中nginx引入lua方式

- xxx_by_lua   : 字符串编写方式
- xxx_by_lua_block  : 代码块方式
- xxx_by_lua_file:  ： 代码的脚本文件方式



#### 编写nginx.conf文件

- 第一种：content_by_lua，在nginx.conf中

  ```nginx
  localtion /testlua {
      content_by_lua "ngx.say('hello world')";
  }  # 输出hello world
  ```

- 第二种：content_by_lua_block{} 表示内部为lua快，里面可以应用lua语句

  ```nginx
  localtion /testlua {
      content_by_lua_block {
          ngx.say('hello world');
      }
  }  # 输出hello world
  ```

- 第三种方式： content_by_lua_file  表示应用lua文件

  ```nginx
  -- nginx.conf 文件中
  location /testlua {
      content_by_lua_file /usr/local/lua/test.lua;
  }
  
  --test.lua文件如下(常用file和block)
  ngx.say("hello world")
  ```

  案例：

  ```nginx
  	server {
          listen 8080;
          location /testsay {
  			default_type   text/html;
  			content_by_lua_block {
  				-- 写响应头
  				ngx.header.a = "1"
  				-- 多个响应头可以使用table
  				ngx.header.b = "2"
  				-- 输出相应
  				ngx.say("a","b","<br/>")
  				ngx.print("c","d","<br/>")
  				--200 状态码退出
  				return ngx.exit(200)
  			}
  		}
  	}  
  
  /*
  Response Headers
      a: 1
      b: 2
      Connection: keep-alive
      Content-Type: text/html
      Date: Thu, 05 Mar 2020 06:02:09 GMT
      Server: openresty/1.9.7.1
      Transfer-Encoding: chunked
  */
  ngx.header: 输出响应头
  ngx.print:输出响应内容
  ngx.say: 通过ngx.print, 但是会最后输出一个换行符
  ngx.exit: 指定状态码返回
  ```



#### openresty使用lua常用的api

- ngx.var.xxx : 获取nginx变量 和内置变量

  ```nginx
  nginx内置的变量
      $args                      请求中的参数;
      $binary_remote_addr        远程地址的二进制表示
      $body_bytes_sent           已发送的消息体字节数
      $content_length            HTTP请求信息里的"Content-Length"
      $content_type              请求信息里的"Content-Type"
      $document_root             针对当前请求的根路径设置值
      $document_uri              与$uri相同
      $host                      请求信息中的"Host"，如果请求中没有Host行，则等于设置的服务器名;    
      $http_cookie               cookie 信息 
      $http_referer              来源地址
      $http_user_agent           客户端代理信息
      $http_via                  最后一个访问服务器的Ip地址
      $http_x_forwarded_for      相当于网络访问路径。    
      $limit_rate                对连接速率的限制          
      $remote_addr               客户端地址
      $remote_port               客户端端口号
      $remote_user               客户端用户名，认证用
      $request                   用户请求信息
      $request_body              用户请求主体
      $request_body_file         发往后端的本地文件名称      
      $request_filename          当前请求的文件路径名
      $request_method            请求的方法，比如"GET"、"POST"等
      $request_uri               请求的URI，带参数   
      $server_addr               服务器地址，如果没有用listen指明服务器地址，使用这个变量将发起一次系统调用以取得地址(造成资源浪费)
      $server_name               请求到达的服务器名
      $server_port               请求到达的服务器端口号
      $server_protocol           请求的协议版本，"HTTP/1.0"或"HTTP/1.1"
      $uri                       请求的URI，可能和最初的值有不同，比如经过重定向之类的
      
  案例：
  location /var {
      default_type   text/html;
      set $c 3;
      content_by_lua_block {
      	local a = tonumber(ngx.var.arg_a) or 0
      	local b = tonumber(ngx.var.arg_b) or 0
      	local c = tonumber(ngx.var.c)  or 0
      	ngx.say("sum：", a+b+c)  # sum：3
  	}
  }  #http://47.95.217.144:8080/var?a=3&b=3   sum:9
  ```

  注意：ngx.var.c 此变量必须提前声明，另外nginx location中使用正则捕获的捕获组可以使用ngx.var[捕获组数字] 来获取

  ```nginx
  # 捕获分组
  location ~ ^/var/([0-9]+)/([\w]+) {
  	default_type   text/html;
      xnice_by_lua_block {
          ngx.say("var[1]:",ngx.var[1])
          ngx.say("var[2]:",ngx.var[2])
      }  # http://47.95.217.144:8080/var/2020/my   ---var[1]:2020 var[2]:my
  }
  ```

- ngx.req 请求模块的常用api； 

   ngx.req.get_headers ：获取请求头； 获取带中划线的请求头时使用如： headers.user_agent 这种方式；如果一个请求头有多个值，则返回的是table

  编辑test.lua文件

  ```lua
  
  local headers = ngx.req.get_headers()
  
  ngx.say("=======header begin========","<br/>")
  ngx.say("host: ",headers['Host'],"<br/>")
  ngx.say("headers['user-agent']:",headers['user-agent'],"<br/>")
  ngx.say("headers.user_agent:",headers.user_agent,"<br/>")
  
  ngx.say("=======遍历headers========","<br/>")
  
  for k,v in pairs(headers) do
  	if type(v) == "table" then	
  		ngx.say(k,": ",table.concat(v," "),"<br/>")
  	else
  		ngx.say(k,': ',v,"<br/>")
  	end
  end
  ```

  编辑nginx.conf文件

  ```nginx
  		location /testlua {
  			default_type   text/html;
  			content_by_lua_file /usr/local/lua/test.lua;
  		}
  ```

- 获取请求参数

  ngx.req.get_uri_args: 获取url请求参数，其用法和get_headers类似

  ngx.req.get_post_args: 获取post请求内容体，其用法和get_headers类似，但是必须提前调用				         ngx.req.read_body()来读取body内容

  ngx.req.get_body_data: 为解析的请求体body内容字符串

  ```lua
  --test.lua文件编辑如下
  
  -- 获取GET请求参数
  ngx.say("========uri get args begin========")
  local uri_args = ngx.req.get_uri_args()
  for k, v in pairs(uri_args) do
  	if type(v) == "table" then
  		ngx.say(k, ": ", table.concat(v,", ","<br/>"))
      else
          ngx.say(k, ": ",v,"<br/>")
      end
  end
  
  -- 获取POST请求参数
  ngx.req.read_body()
  ngx.say("========uri post args begin========")
  local post_args = ngx.req.get_post_args()
  for k, v in pairs(post_args) do
  	if type(v) == "table" then
  		ngx.say(k, ": ", table.concat(v,", ","<br/>"))
      else
          ngx.say(k, ": ",v,"<br/>")
      end
  end
  ```

  重启nginx，查看效果，是否能打印对应的参数

  ```nginx
  相应的ngx.req其他的api：
  	ngx.say("ngx.req.http_version: ", ngx.req.http_version(), "<br/>")  # http版本
  	ngx.say("ngx.req.get_method: ", ngx.req.get_method(), "<br/>")    # 请求方式
  	ngx.say("ngx.req.raw_header: ", ngx.req.raw_header(), "<br/>")   #请求元信息
  ```

- 编解码

  ngx.escape_uri/ngx.unescape_uri : uri编解码

  ngx.encode_args/ngx.decode_args: 参数编解码

  ngx.encode_base64/ngx.decode_base64: BASE64编解码

  ```nginx
  location /var {
      default_type   text/html;
      content_by_lua_block {
  		local request_uri = ngx.var.request_uri
  		ngx.say("request_uri: ",request_uri,"<br/>") 
  		
  		local escape_uri = ngx.escape_uri(request_uri)
  		ngx.say("escape_uri: ",escape_uri,"<br/>")
  		
  
  		ngx.say("decode request_uri: ",ngx.unescape(escape_uri),"<br/>")
  	}
  } 
  
  /*
  request_uri: /url
  escape_uri: %2Furl
  decode request_uri: /url
  */
  ```

  案例： 如果想截取参数，对uri参数进行操作后返回新的uri

  ```lua
  -- test.lua 编辑
  local request_uri = ngx.var.request_uri
  ngx.say("request_uri: ",request_uri,"<br/>") 
  -- 编码
  local escape_uri = ngx.escape_uri(request_uri)
  ngx.say("escape_uri: ",escape_uri,"<br/>")
  -- 解码
  ngx.say("decode request_uri: ",ngx.unescape_uri(escape_uri),"<br/>")
  
  --参数编码
  local request_uri = ngx.var.request_uri
  local question_pos, _  = string.find(request_uri, '?')
  if question_pos > 0 then
      local uri = string.sub(request_uri, 1, question_pos-1)
      ngx.say("uri sub=", string.sub(request_uri, question_pos+1),"<br/>")
      
      --对字符串进行解码
      local args = ngx.decode_args(string.sub(request_uri,question_pos +1))
      for k, v in pairs(args) do
          ngx.say("k=",k," v=",v,"<br/>")
      end
      
      if args and args.userId then
          args.userId = args.userId + 1000
          ngx.say("args +1000: ",uri .. '?'..ngx.encode_args(args),"<br/>")
      end
  end
  
  [[
      request_uri: /testlua?a=1&b=3&userId=1
      escape_uri: %2Ftestlua%3Fa%3D1%26b%3D3%26userId%3D1
      decode request_uri: /testlua?a=1&b=3&userId=1
      uri sub=a=1&b=3&userId=1
      k=b v=3
      k=a v=1
      k=userId v=1
      args +1000: /testlua?b=3&a=1&userId=1001
  ]]
  ```



#### Md5加密

```lua
ngx.say("ngx.md5 :", ngx.md5("123", "<br/>"))
-- ngx.md5 :202cb962ac59075b964b07152d234b70
```



#### Nginx获取时间

​	之前接受的os.time() 会涉及系统调用，性能比较差推荐使用nginx的时间api

```lua
ngx.time()  -- 返回秒级精度时间戳
ngx.now()  -- 返回毫秒级精度时间戳

-- 但是通过这两种方式获取的只是nginx缓存起来的额时间戳，不是实际的，所以有时候会出现奇怪的现象，如下：
local t1= ngx.now()
for i=11000000 do
end
local t2 = ngx.now()
print(t1,", ", t2)  -- t1和t2相同
ngx.exit(200)

-- 为了避免上述情况发生，我们需要提前执行ngx.update_time() 更新缓存
local t1= ngx.now()
for i=11000000 do
end
ngx.update_time()
local t2 = ngx.now()
print(t1,", ", t2)  -- t1和t2相同
ngx.exit(200)
```



#### Nginx 正则

​	ngx.re模块中正则表达式相关的api

```lua
ngx.re.match
ngx.re.sub
ngx.re.gsub
ngx.re.find
ngx.re.gmatch

-- 我们介绍ngx.re.match， 它只有第一次匹配的结果被返回，如果没有返回nil，或者匹配过程发生错误，返回nil；当匹配的字符串找到时，一个lua table capture会被返回，capture[0] 中保存匹配到的字符串，capture[1] 保存的是用括号括起来的第一个子模式的结果，capture[2] 保存的是第二个子模式的结果,以此类推，换句话说可以写几个匹配模式

-- 编辑test.lua文件
-- 只有一个匹配模式
local m, err = ngx.re.match("hello, 1234","[0-9]+")
if m then
    ngx.say(m[0])
else
    if err then
        ngx.log(ngx.ERR, "error:",err)
        return
     end
    ngx.say("match not found")
end

-- 多个匹配模式
local m, err = ngx.re.match("hello, 1234","([0-9])[0-9]+")
if m then
    ngx.say(m[0],"<br/>")  -- 匹配第一个分组
	ngx.say(m[1],"<br/>")  --匹配更多的分组
	ngx.say(m[2])  --匹配更多的分组
else
    if err then
        ngx.log(ngx.ERR, "error:",err)
        return
     end
    ngx.say("match not found")
end
[[
	1234 1234
	1
	nil
]]
```

备注： 有没有注意到，我们每次修改都是要重启nginx，这样太过于麻烦，我们可以用content_by_lua_file引入外部lua，这样的话，只要修改外部lua，就可以了不用重启nginx（将lua_code_cache 设置为 off)

- 语法： lua_code_cache on| off

  默认：on

  适用于上下文： http 、server、 location、 这个指令是指定是否开启lua的代码编译缓存，开发时可以设置为off，一边lua文件及时生效，如果是线上为了性能建议关闭。

  修改nginx.conf为

  ```nginx
  http {
      ...
      # 直接作用于http
      lua_code_cache off;
      server{
          
      }
  }
  ```

- 标准日志输出

  ```nginx
  ngx.log(log_level,...)
  
  日志级别：
  	ngx.STDERR  -- 标准输出
  	ngx.ALERT
  	ngx.ERR
  	
  -- nginx.conf文件中
  location /log {
      content_by_lua_block {
          local num = 55
          local str = "string"
          local obj
          ngx.log(ngx.ERR, "num: ",num)
          ngx.log(ngx.INFO,"string: ",str)
          Print([[i am print]])
          ngx.log(ngx.ERR,"object: ",obj)
      }
  }
  
  # 会根据配置的
  #error_log  logs/error.log;
  #error_log  logs/error.log  notice;
  #error_log  logs/error.log  info;
  #印在不同的文件下
  ```



#### 重定向

ngx.redirect 就是内部uri跳转

```nginx
#niginx.conf中

location /foo {
    content_by_lua_block {
        ngx.say([[I am foo]])
    }
}

location = /bar {
    rewrite_by_lua_block {
        return ngx.redirect("/foo")
    }
}
```



#### Nginx的上下文管理

​	ngx.ctx（全局共享变量，不同的处理阶段，不同的工作进程，共享数据） 在openresty的体系中，可以通过共享内存的方式完成不同的工作进程的数据共享，可以通过lua模块方式完成单个进程内不同请求的数据共享，如何完成单个请求内不同阶段的数据共享？

ngx.ctx 表就是为了解决这类问题设计的，参考下面例子：

```nginx
# nginx.conf中

location /test {
    rewrite_by_lua_block {
        ngx.ctx.foo = 76
    }
    access_by_lua_block {
        ngx.ctx.foo = ngx.ctx.foo +3
    }
    content_by_lua_block {
        ngx.say(ngx.ctx.foo)
    }
}
```

首先ngx.ctx是一个表，所以我们可以对他进行增删改查，它用来存储基于请求的lua环境数据，其生存周期与当前请求相同。它有一个重要的特性： 单个请求内的rewrite（重写），access（访问）和content（内容）等各个阶段保持一致。

注意：每个请求，都有一个自己的ngx.ctx表，例如

```nginx
location /sub {
    content_by_block {
        ngx.say("sub pre:", ngx.ctx.blah)
        ngx.ctx.blah = 32
        ngx.say("sub pre:", ngx.ctx.blah)
    }
}

location /main {
    content_by_block {
        ngx.ctx.blah = 79
        ngx.say("main pre: ",ngx.ctx.blah)
        -- 内部请求一个子请求
        local res = ngx.location.capture("/sub")
        ngx.print(res.body)
        ngx.say("main post:", ngx.ctx.blah)
    }
}

# 访问/main: main pre: 79 sub pre:nil sub pre:32 main post:79
```



#### 更多的api说明

官方网址：https://www.nginx.com/resources/wiki/modules/lua/#nginx-api-for-lua

```nginx

print()               #与ngx.print()方法有区别，print()相当于ngx.log()
ngx.ctx               #这是一个lua的table，用于保存ngx上下文的变量，在整个请求的生命周期内都有效，详细参考官方
ngx.location.capture()    #发出一个子请求，详细用法参考官方文档。
ngx.location.capture_multi()  #发出多个子请求，详细用户参考官方文档。
ngx.status                #读或者写当前请求的相应状态，必须在输出相应头之前被调用。
ngx.header.HEADER     #访问或设置http header头信息，详细参考官方文档
ngx.req.set_uri()    #设置当前请求的URI，详细参考官方文档
ngx.set_uri_args(args)   #根据args参数重新定义当前请求的URI参数。
ngx.req.get_uri_args()   #返回一个LUA TABLE,包括所有当前请求的URL参数
ngx.req.get_post_args()  #返回一个LUA TABLE,包括所有当前请求的POST参数
ngx.req.get_headers()    #返回一个包含当前请求头信息的lua table.
ngx.req.set_header()     #设置当前请求头header某字段值。当前请求的子请求不会受到影响。
ngx.req.read_body()      #在不阻塞nginx其他事件的情况下同步读取客户端的body信息[详细]
ngx.req.discard_body()   #明确丢弃客户端请求的body.
ngx.req.get_body_data()   #以字符串的形式获得客户端的请求body内容
ngx.req.get_body_file()  #当发送文件请求的时候，获得文件的名字
ngx.req.set_body_data()  #设置客户端请求的BODY
ngx.req.clear_header()   #请求某个请求头
ngx.exec(uri, args)      #执行内部跳转，根据uri和请求参数
ngx.redirect(uri, args)  #执行301或者302的重定向。
ngx.send_headers()       #发送指定的响应头
ngx.headers_sent         #判断头部是否发送给客户端ngx.headers_sent=true
ngx.print(str)           #发送给客户端的响应页面
ngx.say()                #作用类似ngx.print,不过say方法输出后会换行
ngx.log(log.level, ...)   #写入nginx日志
ngx.flush()            #将缓冲区内容输出到页面(刷新响应)
ngx.exit(http-status)   #结束请求并输出状态码
ngx.eof()               #明确指定关闭结束输出流
ngx.escape_uri()        #将URI编码(本函数对逗号，不编码，而php的uriencode会编码)
ngx.unescape_uri()     #将uri解码
ngx.encode_args(table)   #将table解析成url参数
ngx.decode_args(uri)     #将参数字符串编码为一个table
ngx.encode_base64(str)    #BASE64编码
ngx.decode_base64(str)    #BASE64解码
ngx.crc32_short(str)      #字符串的crs32_short哈希
ngx.crc32_long(str)       #字符串的crs32_long哈希
ngx.md5(str)              #返回16进制MD5
ngx.md5_bin(str)          #返回2进制MD5
ngx.today()               #返回当前日期yyyy-mm-dd
ngx.time()                #返回当前时间戳
ngx.now()                 #返回当前时间
ngx.update_time()         #刷新后返回
ngx.localtime()           #返回yyyy-mm-dd hh:ii:ss
ngx.utctime()             #返回yyyy-mm-dd hh:ii:ss格式的utc时间
ngx.cookie_time(src)      #返回可用于http header使用的时间
ngx.parse_http_time(str)  #解析HTTP头的时间
ngx.is_subrequest         #是否子请求(值为true or false)
ngx.re.match(subject, regex, options, ctx)   #ngx正则表达式匹配，详细参考官网
ngx.re.gmatch(subject, regex, opt)      #全局正则匹配
ngx.re.sub(sub, reg, opt)     #匹配和替换(未知)
ngx.re.gsub()                 #未知
ngx.shared.DICT               #ngx.shared.DICT是一个table里面存储了所有的全局内存共享变量
    ngx.shared.DICT.get
    ngx.shared.DICT.get_stale
    ngx.shared.DICT.set
    ngx.shared.DICT.safe_set
    ngx.shared.DICT.add
    ngx.shared.DICT.safe_add
    ngx.shared.DICT.replace
    ngx.shared.DICT.delete
    ngx.shared.DICT.incr
    ngx.shared.DICT.flush_all
    ngx.shared.DICT.flush_expired
    ngx.shared.DICT.get_keys
```





#### openresty中使用json模块

​	web开发中，经常使用的数据结构是json,openresty中封装了json模块。

- 如何引入cjson模块，需要使用require

  ```lua
  local json = require("cjson")
  
  -- json.encode() 将表格数据编码为json字符串：
  jsonString = json.encode(表格数据)
  
  -----------------------------------------------
  -- 在test.lua中
  local json = require("cjson")
  	-- 同时存在哈希的table 和 数组的table，数组型的键值会转换，且地址会转换为数组，因为全使数组型
  local t = {1,3,name="张山", age="19",address={"地址1","地址2"},sex="女"}
  ngx.say(json.encode(t),"<br/>")
  
  local str = json.encode({a=1,[5]=3})
  ngx.say(json.encode(str),"<br/>")
  
  [[
      "1":1,"2":3,"address":["地址1","地址2"],"age":"19","name":"张山","sex":"女"}
      "{\"a\":1,\"5\":3}"
  ]]
  ```

- table所有键为数组型键值对时，会当作数组看待，空位将转化为null, 满足数组的索引

  ```lua
  local str = json.encode({[3]=1,[5]=2,[6]="3",[7]=4})
  ngx.say(str,"<br/>") --[nll,null,1,null,2,"3",4]
  ```

- json.decode()  将json字符串解码为表格对象

  ```lua
  local json = require("cjson")
  local str = [[ 
  	{
     "a":"v","b":2,"c":{"c1":1,"c2":2},"d":[10,11],"1":100
  }
  ]]
  local t = json.decode(str)
  ngx.say("--> ",type(t))  -- table
  
  local str = [[ {"a":1,"b":null }]]
  local t = json.decode(str) 
  ngx.say(t.a,"<br/>")  --1
  ngx.say(t.b == nil,"<br/>")  -- false
  ngx.say(t.b == cjson.null,"<br/>")  --true
  -- 注意：null会被转换为json.null
  ```



#### Nginx的异常处理

pcall ：当我们使用decode解码一个不规范的json字符串时，会抛500错误，但是我们不想看到500错误，这是我们就需要使用pcall命令；使用pcall来包装需要执行的代码。

pcall接受一个函数和要传递给后者的参数，并执行，执行结果：有错误、无错误：返回值 true或者false

pcall以一种“保护模式”来调整第一个参数，因此pcall可以捕获函数执行中的任意错误

- 案例：重新包装json decode解码

  ```lua
  local json = require("cjson")
  
  local function _json_decode(str)
  	return json.decode(str)
  end
  
  function json_decode(str)
  	local ok, t = pcall(_json_decode, str) -- 第一个是函数，第二个是传给函数的参数
  	if not ok then
         ngx.say(t)  -- 错误信息
  		return nil
      end
      return t
  end
  
  local str = [[ {"key:"value"}]]  -- 错误json字符串
  ngx.say(json_decode(str))
  ```

- 方案二： cjson模块自带一个安全接口cjson.safe(), 该接口兼容cjson，不会抛异常，而是返回nil

  ```lua
  local json = require("cjson.safe")
  local str = [[ {"key:","value"}]]
  local t = json.decode(str)
  if t then
  	ngx.say("-->",type(t))
  else
  	ngx.say("t is nil ")
  end
  ```

- 空table返回object还是array

  ```lua
  local json = require("cjson")
  ngx.say("value -->",json.encode({}))  --输出{}
  
  -- 但是空数组{} 应该输出[]，要达到这个目的要把目标的 encode_empty_table_as_object 设置为 false
  local json = require("cjson")
  json.encode_empty_table_as_object(false)
  ngx.say("value --> ",json.encode({}))  --value--> []
  ```

  