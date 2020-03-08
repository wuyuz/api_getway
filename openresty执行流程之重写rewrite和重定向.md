## openresty执行流程之重写rewrite和重定向

#### 重写rewrite阶段

1）重定向
2）内部，伪静态

先介绍一下if，rewrite指令

- **if指令**
  语法：if (condition){...}
  默认值：无
  作用域：server,location
  对给定的条件condition进行判断。如果为真，大括号内的指令将被执行。上面的if和(之间需要留空格，否则会报错。

  - 条件可以为一个变量, 如果一个变量名进行条件判断，空字符串'' 或 字符串为'0',都表示为假 false

    ```nginx
    location /api {    
        set $a '11111';
        if ($a){
            return 200 "11111";
        }
        return 200 "222222"
    }
    # 如果没有匹配到上面的就返回 200 2222222222
    ```

  - 条件为表达式

    ```nginx
    正则表达式匹配：
        = ,!= 比较的一个变量和字符串
        ~：与指定正则表达式模式匹配时返回“真”，判断匹配与否时区分字符大小写；
        ~*：与指定正则表达式模式匹配时返回“真”，判断匹配与否时不区分字符大小写；
        !~：与指定正则表达式模式不匹配时返回“真”，判断匹配与否时区分字符大小写；
        !~*：与指定正则表达式模式不匹配时返回“真”，判断匹配与否时不区分字符大小写；
    
    location /api {
         # 判断 使用正则且不区分大小写
         if ($request_uri ~* "/api/[0-9]+") {
            return 200 "api";
         }
    }
    ```

  - 文件及目录匹配判断：

    ```nginx
    -f, !-f：判断指定的路径是否为存在且为文件；
    -d, !-d：判断指定的路径是否为存在且为目录；
    -e, !-e：判断指定的路径是否存在，文件或目录均可；
    -x, !-x：判断指定路径的文件是否存在且可执行；
    
    location /api {
         if (-f "/usr/local/lua/test.lua") {
            return 200 "test存在";
         }
    }
    ```

    **注意**:

    1)nginx if 没有对应的else
    2)if 表达式中是不能用  &&  ||

    3)nginx的配置中不支持if条件的逻辑与&& 逻辑或|| 运算等逻辑运算,而且不支持if的嵌套语法，否则会报错

    ```nginx
    location /api {
        if ( $arg_a != '1' && $arg_a != '2' ) {
          rewrite ^/(.*)$ https://www.baidu.com permanent;
        }
    }
    
    #会报错nginx: [emerg] invalid condition,修改为：
    location /api {
        set $flag 0;
        if ($arg_a != '1') {
            set $flag "${flag}1";   ---> 01
        }
    
        if ($arg_a != '2'){
            set $flag "${flag}1";   ---> 011
        }
    
        if ($flag = "011"){
            rewrite ^/(.*)$ https://www.baidu.com permanent;
        }
    }
    
    ```



#### Rewrite规则

- http status code 301 与 302区别

  301 redirect: 301 代表永久性转移(Permanently Moved)
  302 redirect: 302 代表暂时性转移(Temporarily Moved)

  详细来说，301和302状态码都表示重定向，就是说浏览器在拿到服务器返回的这个状态码后会自动跳转到一个新的URL地址，这个地址可以从响应的Location首部中获取,用户看到的效果就是他输入的地址A瞬间变成了另一个地址B
  这是它们的共同点。他们的不同在于。301表示旧地址A的资源已经被永久地移除了（这个资源不可访问了），搜索引擎在抓取新内容的同时也将旧的网址交换为重定向之后的网址；302表示旧地址A的资源还在（仍然可以访问），这个重定向只是临时地从旧地址A跳转到地址B，搜索引擎会抓取新的内容而保存旧的网址。

- **rewrite指令**

  语法 rewrite regex replacement [flag];

  regex：perl兼容正则表达式语句进行规则匹配
  replacement：将正则匹配的内容替换成replacement
  flag标记：rewrite支持的flag标记

  rewrite功能就是，使用nginx提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向。

  rewrite只能放在server{},location{},if{}中，并且只能对域名后边的除去传递的参数外的字符串起作用，
  例如http://www.a.com/api/index.jsp?id=1&u=str 只对api/index.jsp重写。

  正则表达式元字符:
  .   :匹配除换行符以外的任意字符
  ?  :重复0次或1次

  \+ :重复1次或更多次

  \* :重复0次或更多次
  \d    :匹配数字
  ^     :匹配字符串的开始字符
  $     :匹配字符串的结束字符
  {n}   :重复n次
  {n,}  :重复n次或更多次
  [c]   :匹配单个字符c
  [a-z] :匹配a-z小写字母的任意一个

  分组：在rewrite中，如果使用小括号()，那么在小括号之间匹配的内容，可以在后面通过$1来引用，$2表示的是前面第二个()里的内容

- flag标志位：

  - last : 相当于Apache的[L]标记，表示完成rewrite
  - break : 停止执行当前虚拟主机的后续rewrite指令集
  - redirect : 返回302临时重定向，地址栏会显示跳转后的地址
  - permanent : 返回301永久重定向，地址栏会显示跳转后的地址

- 简单案例：在nginx.conf中

  ```nginx
  location /foo {
    rewrite ^ /bar redirect;
  }
  
  location /bar {
    echo "bar";
  }
  ```

  简单案例2：

  ```nginx
  location /api {    
      rewrite ^/(.*) https://www.baidu.com/s?wd=$1 permanent;
  }
  
  #请求：http://192.168.31.150/api   ---> $1 = api
  #请求：http://192.168.31.150/api/pro   ---> $1 = api/pro
  ```

  last 和 break 区别：

  last： 停止当前这个请求，并根据rewrite匹配的规则重新发起一个请求。新请求又从第一阶段开始执行…
  break：相对last，break并不会重新发起一个请求，只是跳过当前的rewrite阶段，并执行本请求后续的执行阶段…

  break与last都停止处理后续rewrite指令集，不同之处在与last会重新发起新的请求，
  而break不会。当请求break时，如匹配内容存在的话，可以直接请求成功

  简单案例：

  ```nginx
  location /break/ {  
     rewrite ^/break/(.*) /test/$1 break;  #----break；/test/$1 会在根目录下去查找/test/$1文件，即去 /data/www/html/test/目录下找 $1 文件
  }  
  
  location /last/ {  
     rewrite ^/last/(.*) /test/$1 last;    #----last；/test/$1 会重新走一遍location匹配流程
  }
  
  location /test/ {  
     echo "test page";
  }  
  
  # 上述的区别在于：break的情况下会直接去/test/文件下查找，而last会再走一遍所有location，从而匹配到/test/ 输出 test page
  ```

  

#### 伪静态

```nginx
location = /pro {
    echo "pro_$arg_pid";
}

location /{
    rewrite ^/product/(\d+).html$ /pro?pid=$1 last;
}
```

应用场景：
	a)可以调整用户浏览的URL，看起来更规范，合乎开发及产品人员的需求。
	b)为了让搜索引擎搜录网站内容及用户体验更好，企业会将动态URL地址伪装成静态地址提供服务。
	c)网址换新域名后，让旧的访问跳转到新的域名上。例如，访问京东的360buy.com会跳转到jd.com
	d)根据特殊变量、目录、客户端的信息进行URL调整等

可应用在server,location,if标签

执行顺序是：
	a）执行server块的rewrite指令
	b）执行location匹配
	c）执行选定的location中的rewrite指令
如果其中某步URI被重写，则重新循环执行a-c，直到找到真实存在的文件；循环超过10次，
则返回500 Internal Server Error错误。

- rewrite_by_lua

  语法：rewrite_by_lua  <lua-script-str>
  语境：http、server、location、location if
  阶段：rewrite tail
  作为rewrite阶段的处理，为每个请求执行指定的lua代码。注意这个处理是在标准HttpRewriteModule之后进行的：执行内部URL重写或者外部重定向，典型的如伪静态化的URL重写。其默认执行在rewrite处理阶段的最后
  - ngx.redirect ---重定向
    语法：ngx.redirect(uri, status?)

    该方法会给客户端返回一个301/302重定向，具体是301还是302取决于设定的status值，
    如果不指定status值，默认是返回302

    ```nginx
    #rewrite ^ /bar redirect; 等价于 ngx.redirect("https://www.baidu.com", 302)
    #rewrite ^ /bar permanent; 等价于 ngx.redirect("https://www.baidu.com", 301)
    
    --------------nginx.conf配置文件
    location /lua_rewrite_1 {
      rewrite_by_lua_block {
        if ngx.req.get_uri_args()["jump"] == "1" then
          return ngx.redirect("https://www.baidu.com", 302)
        end
      }
      echo "no rewrite";
    }
    
    #当我们请求http://10.11.0.215/lua_rewrite_1时发现没有跳转，
    #而请求http://10.11.0.215/lua_rewrite_1?jump=1时发现跳转到百度首页了。此处需要 301/302跳转根据自己需求定义。
    ```

    再举个例子

    ```nginx
    location /foo {
      set $a 12;
      set $b '';
      rewrite_by_lua 'ngx.var.b = tonumber(ngx.var.a) + 1'; #lua代码块
      if ($b = '13') {
        rewrite ^ /bar redirect;
        break;  # 终结当前location不往下走
      }
      echo "res = $b";
    }
    
    location /bar {
      echo "bar";
    }
    
    ---------------------------------------
    #因为if会在rewrite_by_lua之前运行，所以判断将不成立。正确的写法应该是这样：
    location /foo {
        set $a 12;
        set $b '';
        rewrite_by_lua_block {
            ngx.var.b = tonumber(ngx.var.a) + 1
            if tonumber(ngx.var.b) == 13 then
                return ngx.redirect("/bar");
            end
        }
        echo "res = $b";
    }
    ```

    

- ngx.req.set_uri ---内部重写
  语法： ngx.req.set_uri(uri, jump?)

  通过参数uri重写当前请求的uri；参数jump，表明是否进行locations的重新匹配。
  当jump为true时，调用ngx.req.set_uri后，nginx将会根据修改后的uri，重新匹配新的locations；
  如果jump为false，将不会进行locations的重新匹配，而仅仅是修改了当前请求的URI而已。jump的默认值为false。

  ```nginx
  #rewrite ^ /lua_rewrite_3;             等价于  ngx.req.set_uri("/lua_rewrite_3", false);
  #rewrite ^ /lua_rewrite_3 break;       等价于  ngx.req.set_uri("/lua_rewrite_3", false);
  #rewrite ^ /lua_rewrite_4 last;        等价于  ngx.req.set_uri("/lua_rewrite_4", true);
  
  ---------------nginx.conf配置文件
  location /foo {
      rewrite_by_lua_block {
           ngx.req.set_uri_args({a = 1, b = 2});
           ngx.req.set_uri("/bar/index.html", true);
      }
  }
  
  location /bar {
    echo "bar uri : $uri,a : $arg_a,b : $arg_b";
  }
  
  #ngx.req.set_uri_args：重写请求参数；
  #注意 ngx.req.set_uri(uri, true) 时，ngx.req.set_uri_args的顺序
  ```

  





