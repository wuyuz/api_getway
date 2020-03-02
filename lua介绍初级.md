## lua介绍初级



#### lua介绍

​	1993年巴西里约热内卢天主教大学诞生了一门编程语言，他们取名lua---在葡萄牙语中代表美丽的月亮，简洁优雅，lua从一开始就是作为一门方便嵌入（其他应用程序）并可扩展的轻量级脚本语言来设计，因此他一直遵循简单、小巧、可移植原则，能以C程序库的形式嵌入到宿主程序中。LuaJIT2和标准Lua 5.1 解释器采用著名的MIT许可协议。



##### Lua和LuaJIT的区别

​	lua非常高效，它运行得比其他脚本（Perl、Python、Ruby）都快，这点在第三方的独立测试中得以证实，尽管如此，仍然会有人不满足，他们总觉得“还不够快”。 LuaJIT就是一个为了榨出一些速度的尝试，它利用即时编译（Just-in Time）技术把Lua代码编译成本机机器码反交给cpu直接执行。



#### Lua编写HelloWord脚本

​	为了方便以后查找，我们在/usr/local/下创建lua目录，用以存放lua脚本，创建一个helloword.lua脚本，再使用notepad进行编辑

```shell
[root@MyHost local]# mkdir lua
[root@MyHost local]# cd lua/
[root@MyHost lua]# touch helloworld.lua
[root@MyHost lua]# ls
helloworld.lua
```

- 编写helloworld代码

  ```lua
  print("hello word")
  
  
  function sayHello()
  	print("function Hello")
  end
  
  sayHello();
  ```

- 编译运行

  ```shell
  [root@MyHost profile.d]# cd /usr/local/openresty/
  [root@MyHost openresty]# ls
  bin  luajit  lualib  nginx
  [root@MyHost openresty]# cd luajit/
  [root@MyHost luajit]# ls
  bin  include  lib  share
  [root@MyHost luajit]# cd bin/
  [root@MyHost bin]# ls
  luajit  luajit-2.1.0-beta1
  [root@MyHost bin]# ./luajit /usr/local/lua/helloworld.lua   # 编译运行
  hello word
  function Hello
  ```



#### Lua基本语法

- 注释

  ```lua
  单行注释： 两个减号  --注释内容
  多行注释： 
  --[[
      多行注释
  ]]
  ```

- 基本类型（lua中有8个基本类型分别为）

  ```lua
  nil (空) -- python的None
  boolean (布尔)
  number (数字) -- python的int，双精度的浮点数
  string (字符串)
  table (表) -- 类似于python的map
  function (函数)
  userdata (自定义类型)
  thread (线程/协程)
  
  -- 使用type函数测试给定变量的值类型
  ```

- 变量

  命名规则：大小写区分、其他的python命名规则类似

  - Lua 标识符用于定义变量，函数获取其他用户定义的项，标识符以一个字母或'_'开头后。

  - **注意**：一般约定，以下划线开头连接的一串大写字母的名字（如：_VERSION)  被保留用于lua的全局变量

  - 不能使用关键字

    ```
    and 、break、 do、else、elseif、end、false、true、for、function、if
    in、local、nil、 not、 or、return、 then、 repeat、 until、while
    ```

  全局变量：在默认情况下，变量总是认为是全局的，除非，你在前面加了"local"，这一点要特别注意，因为可能想在函数中使用局部变量，却忘了加local。

  全局变量不需要声明，给一个变量复制后即创建了一个全局变量，访问一个没有初始化的全局变量也不会报错，只不过得到：nil。

  ```lua
  b = 1  -- 可加分号，也可不加
  print("变量B：",b)
  print("变量B的类型：",type(b))   --nubmber, 和Python一样自动类型赋值
  print(a,c) -- nil nil
  
  --如果你想删除一个全局变量，只需要将变量赋值为nil。
  b = nil
  print(b) -- nil, 当且仅当一个变量不等于nil时，这个变量即存在
  
  function syaVar()
  	local d = 2;
  	e = 4;
  	print("局部变量：",d)
  end
  
  syaVar();
  print(d,e); -- nil 4
  ```

  局部变量：在变量名前加local

- nil类型

  ```lua
  print(type(a))  --nil
  ```

  对于全局变量和table，nil还有一个删除的作用，给全局变量或者table表里的变量赋值一个nil值，等同于把他们删除

  ```lua
  tab = {key1 = "val1",key2="val2"}
  for k,v in pairs(tab) do
  	print(k .. "--" .. v)  -- .. 表示相加的意思，python中字符串相加使用+，这里使用..
  end
  print('------')
   
  tab.key1 = nil   -- 访问table的值
  for k,v in pairs(tab) do
  	print(k,"---",v)
  end
  
  -- 判断nil类型 作比较时应该加上双引号
  type(x)
  print(type(x)==nil)   -- false
  print(type(x)=="nil")  -- true
  ```

- 布尔类型，可选值 true/false

  lua中nil和false为假，其他的所有值都为真，比如：0、空字符

  ```lua
  if a then
  	print(a)
  else
  	print("not a")
  end
  
  if b then
  	print(b)
  else
  	print("not b")
  end
  
  ```

- number类型，用于表示实数，和C/C++里面的都变了类型很类似。可以使用数学函数 math.floor(向下取整)和math.ceil(向上取整)进行取整操作

  ```lua
  -- number类型
  local count = 10
  local order = 3.99
  local score = 98.01
  
  print(math.floor(order))  -- 向下取整 3
  print(math.ceil(score))  -- 向上取整 99
  ```

- string类型

  lua中有三种方式表达字符串：

  ```lua
  1、使用一对匹配的单引号，如：'hello'
  2、使用一对匹配的双引号，如："hello"
  3、字符串可以用一种长括号（[[]])括起来的方式定义，整个词法分析的时候不会受到分行限制，类似于多行字符串，如写诗之类的；注意：lua中的字符串在创建时会插入lua虚拟机中，相对而言是独立的，所以不能修改，也不能通过下标访问。
  
  -- string类型
  local str1 = "hello world"
  local str2 = 'hello lua'
  local str3 = [[
  	窗前明月光，
  	疑似地上霜。
  ]] 
  
  local change = "wang \'Li\'"    -- 支持转义字符
  local change = "wang\n \'Li\'"    -- 换行
  print("a" .. "b")   -- 字符串的拼接，使用..
  
  print(str1)
  print(str2)
  print(str3)
  print(change)
  
  print(tonumber("10") == 10)  -- 字符串转number
  print(tostring(10) == "10")  -- number转字符串
  
  print(#"This is string")  -- 使用#来计算字符串长度 14
  
  s3 = '1234455sd'
  local s4 = string.sub(s,2,5)  -- 字符串的截断，但是必须赋值给新变量，s并没有变化
  print(s3,s4)  -- 1234455sd	2344  截取的是开区间，不骨头不顾尾
  ```

  注意：string类型只要不允许修改，只能重新赋值。

  ```lua
  s = '12313'
  s1 = s
  s = 'dadfafd'
  
  print(s)  -- dadfafd
  print(s1)  -- 12313  s被重新赋值了，但是s1指向没有变，和python一样
  ```



#### function 函数

有名函数：

```lua
optional_function_scope function function_name(argument1, argument2, argument3...)
	function_body
	return result_params_comma_separated
end

--解析：
	optional_function_scope：该参数是可选的指定函数是全局函数还是局部函数，未设置该参数时默认为全局函数，如果你需要设置函数为局部函数，使用local。
	function： 函数定义关键字
	function_name: 指定函数名称
	argument1，argument2...: 函数参数
	function_body: 函数体
	result_params_comma_separaed: 函数返回值，lua语言函数可以返回多个值，用,隔开。
    end： 函数结束关键字


--函数返回两个值的最大值
local function max(v1,v2)
	if (v1>v2) then 
		return v1;
	else
		return v2;
	end
	
end
-- 调用函数
print("两个比较最大值",max(2,5));
```

​	

匿名函数：本质是将函数赋值给变量

```lua
optional_function_scope function_name = function (argument1, argument2, argument3...)
	function_body
	return result_params_comma_separated
end

function foo() end 等价于 foo = function()  end
```

全局函数和局部函数的区别：

- 使用function声明的函数为全局函数，在引用时可以不会因为声明顺序而找不到

- 使用local function的函数为局部函数，在引用时必须声明在声明函数的后面

  ```lua
  local function test1()
  	print("hello test1")
  end
  
  function test()
  	test2()
  	test1()
  end
  
  function test2()
  	print("hello test2")
  end
  
  test() -- 会抛错
  ```

  

函数参数：

- 将函数作为参数传递给函数

  ```lua
  local myprint = function(param)
  	print("这是打印函数 -   ##",param,"##")
  end
  
  local function add(num1,num2,functionPrint)
  	result = num1 + num2
  	functionPrint(result)
  end
  
  add(1,4,myprint)
  ```

- 传递参数，lua参数可变

  ```lua
  local function foo(a,b,c,d)
  	print(a,b,e,r)  
  end
  
  foo(1,3) -- 打印1，3，nil，不会报错
  ```

  - 若参数个数大于形参个数，从左向右，多余的实参被忽略

  - 若参数个数小于形参个数，从左向右，没有被初始化的形参被初始化为nil

  - lua还支持变长参数（不定长参数），用...表示。此时访问参数要用你... 如

    ```lua
    -- 算平均值
    function average(...)
    	result = 0
    	print("...类型",type(...),...) -- ...类型	number	1	2	4	5	6	3
    	local arg = {...}  -- arg 就是table类型,将每个number做成一个table
    	for i, v in ipairs(arg) do
    		result = result + v
    	end
    	print("总共传入" .. #arg .."个数")
    	return result/#arg
    end
    
    print("平均值：",average(1,2,4,5,6,3))
    ```

- 返回值

  lua函数允许返回多个值，返回多个值时用逗号隔开

  ```
  local function init()
  	return 1,"lua"
  end
  
  local x, y = init();
  print(x,y)
  ```

  函数返回值的规则：

  - 若返回值个数大于接受变量个数，多余的返回值将会被忽略
  - 若小于接受的个数，默认赋值为nil

  

