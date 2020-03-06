## Lua 进阶知识



#### string类型

- string.upper(s) : 接受一个字符串s, 返回一个把所有小写字母变成大写字母的字符串

  ```lua
  print(string.upper("Hello Lua")) -- HELLO LUA
  ```

- string.lower(s) : 接受一个字符串s， 返回一个把所有大写字母变成小写字母的字符串

  ```lua
  print(string.lower("Hello Lua"))  -- hello lua
  ```

- string.len(s) ：接受一个字符串，返回他的长度；使用此函数是不推荐的，推荐使用#运算符来获取lua长度

  ```lua
  print(string.len("Hello Lua")) -- 9
  print(#("Hello lua"))  -- 9
  ```

- string.find(s, p [, init [,  plain]]) : 查找子字符串，在s字符串中的第一次匹配p字符串。若匹配成功，则返回p字符串中出现的开始位置和结束位置；若匹配失败，则返回nil。第三个参数，init默认为1，并且可以为负整数，当init为负数时，表示从后往前数的字符个数；再从此索引处开始向后匹配字符串p。第四参数默认为false，当其为true时，关闭模式匹配；只会把p看成一个字符串对待。

  ```lua
  local find = string.find
  print(find("abc cba","ab"))  -- 1  2
  print(find("abc cba dab","ab"))  -- 1  2 这是第一个出现的ab的位置
  print(find("abc cba","ab",2))  -- nil
  print(find("abc cba","ba",-1)) -- nil
  print(find("abc cba","ba",-3)) -- 6 7 从倒数第三个开始
  print(find("abc cba","b[ac]",-6)) -- 2 3 默认第四个参数为false，支持正则等功能
  print(find("abc cba","b[ac]",-5)) -- 6 7 
  print(find("abc cba","b[ac]",-5,true)) -- nil 关闭所有功能
  ```

  find的模式匹配

  ```lua
  local s = "am+df"
  print(string.find(s,"m+",1,false))  -- 2  2 这里算是开启了正则，所以只有2 2
  print(string.find(s,"m+",1,true))  -- 2 3
  -- 其中字符 + 在lua正则表达式中的意思是匹配在它之前的那个字符一次或多次，也就是说m+在正则表达式里会匹配m，mm...
  ```

- string.format(formatstring, ...) : 格式化输出

  按照格式化参数 formatstring，返回后面... 内容的格式化版本。编写格式化字符串的规则与标准c语言的printf函数的规则基本相同：它由常规文本和指示组成，这些指示控制了每个参数应放格式化结果的什么位置，及如何放入他们。

  ```lua
  -- 一个指示由字符%加上一个字母组成，这些字母指定了如何格式化参数，例如 d用于十进制数、x用于十六进制数、o用于八进制、f用于浮点数、s用于字符串等。在字符%和字母之间可以再指定一些其他选项，用于控制格式细节
  
  print(string.format("%.4f",3.1415926))  -- 保留4位小数
  print(string.format("%d %x %o",31, 31, 31)) -- 十进制31转换不同进制
  
  
  d = 29;m=7;y=2015
  print(string.format("%s %02d/%02d/%d","today is:",d,m,y)) -- today is: 29/07/2015,第一个%s对应后面的“this is：”，这里的%02d表示保留两位数
  ```

- 整型数字和字符互换, string.byte(s,[,i,[,j]]) : 返回字符 s[i], s[i+i], s[i+2]，..., s[j], i默认是1，即第一个字节，j的默认值位i

  lua字符串总是由字节构成，下标是从1开始的，这不同于c和perl

  ```lua
  print(string.byte("abc",1,3))  -- 97	98	99
  print(string.byte("abc",3))  --99 缺少第三个参数，第三个参数默认和第二个相同
  print(string.byte("abc")) -- 97  缺少第二第三个参数，默认两个参数为1
  
  -- 由于string.byte 只返回整数，并不像string.sub函数那样尝试创建新的lua字符串，因此使string.byte更为高效
  ```

- string.char(...) : 接受0个或多个整数（0~255），返回这些整数对应的ascii码字符组成的字符串，当参数为空时，默认为0

  ```lua
  print(string.char(96,97,99))
  print(string.char())
  ```

- string.match(s, p [,init]) : 匹配字符串，在字符串s中匹配字符串p，若成功则返回目标字符串与模式匹配的子串；否则返回nil；第三个参数init默认是1，并且可以为负整数，当init为负数时，表示从后往前的字符个数，再开始匹配p

  ```lua
  print(string.match("hello lua","lua")) -- lua
  print(string.match("lua lua","lua",2))  -- 匹配到后面那个lua
  print(string.match("today is: 29/07/2015","%d+/%d+/%d+"))  --29/07/2015
  ```

- string.gmatch(s, p)  : 匹配多个字符串，返回一个迭代器函数，通过这个迭代器函数可以遍历到在字符串s中出现模式串p的所有地方

  ```lua
  s = "hello world fr4osm lua"
  for w in string.gmatch(s, "%a+") do  -- 匹配最长连续且只含字母的字符串
  	print(w)
  end
  
  [[ -- 运行结果
  hello
  world
  fr
  osm
  lua
  ]]
  
  -- 例2
  t = {}
  s = "from=world, to=lua"
  for k, v in string.gmatch(s,"(%a+)=(%a+)") do
      t[k] = v
  end
  
  for k, v in pairs(t) do
      print(k, v)
  end
  
  [[
  to	lua
  from	world
  ]]
  ```

- string.rep(s, n) : 字符串的重复，返回n次 拷贝s拼接的字符串

  ```lua
  print(string.rep("abc",3))  -- abcabcabc
  ```

- string.sub(s, i [, j]) : 截取子字符串，返回字符串s中，索引i到索引j之间的字符串，当j缺省时，默认为-1，也就是字符串s的最后位置。i可以为负数，当索引i在字符串s的位置索引j的后面时，将返回一个空字符串

  ```lua
  print(string.sub("hello lua", 4, 7))   -- lo l
  print(string.sub("hello lua", 2))   --ello lua
  print(string.sub("hello lua", 2, 1))  -- 
  print(string.sub("hello lua", -3, -1))  --lua
  ```

- string.gsub(s, p, r [,n])  : 替换字符串, 将目标字符串s中的所有子串p替换成字符串r。可选参数n，表示限制替换次数。返回值有两个，第一个是被替换后的字符串，第二个是替换的次数

  ```lua
  print(string.gsub("lua lua lua", "lua", "hello"))  --hello hello hello	3
  print(string.gsub("lua lua lua","lua","hello", 2))  -- 替换两次，hello hello lua	2
  
  -- 可使用正则
  print(string.gsub("lua lux lua","lu[x]","hello", 2)) --lua hello lua	1
  ```

- string.reverse(s) : 反转字符串

  ```lua
  print(string.reverse("hello lua"))  --aul olleh
  ```

  

#### table 类型

lua中的table内部实际采用了哈希表和数组分别保存键值普通值；下标从1开始。

不推荐混合使用这两种方式。

```lua
local color = {first="red","blue",third="green","yellow"}
print(color["first"])  -- red
print(color[1]) -- blue
print(color["third"]) -- green
print(color[2])  -- yellow
```

- table.getn 获取长度；相当于去长度写作一元操作 #； 字符串的长度是它的字节数（就是以一个字符一个字符计算的字符串长度）。对于常规的数组，里面从1到n放着一些非空的值的时候，它的长度就精确的为n，最后一个值的下标

  ```lua
  local tbTest = {1,a=2,3}
  print("Test"..table.getn(tbTest))  --2, 为什么不是3，因为getn只能执行数值型table即数组的长度
  
  local tbTest1 = {b = 1, a=2,3}
  print("Test"..table.getn(tbTest))  --1
  
  -- 注意 数组型的表示方法如下：{[1] =1, [2]=3},[1]表示数值型下标。
  local tbTest1 = {[3]=6,b = 1, a=2,3}  -- 这里输出的是1，是因为[3] 和[1]=3不连续，所以只算一个
  print("Test"..table.getn(tbTest))  --1
  
  local tbTest1 = {[2]=6,b = 1, a=2,[3]=6}   -- 显然连续数组型下标
  print("Test"..table.getn(tbTest))  --2 
  ```

  获取table的真实长度

  ```lua
  function table_length(t)
  	local length = 0
  	for k, v in pairs(t) do
  		length = length +1
  	end
  	return length
  end
  ```

  特殊说明：如果数组有一个nil，，那么#t可能是指向任意一个nil的值，会发生错误；**如果一个元素要删除，直接remove，不要用nil去代替**

- table.concat(table, [, sep [, i [,j]]]) 对于元素是string或者number类型的表 table。拼接数组，要求不是map型table

  ```lua
  local a = {1,3,5,"hello"}
  
  print(table.concat(a)) -- 135hello
  print(table.concat(a,"|")) --1|3|5|hello
  print(table.concat(a," ",4, 2)) -- 
  print(table.concat(a," ",2, 4)) -- 3 5 hello
  ```

- table.insert(table, [pos , ] value) : 在数组型表table的pos索引位置插入value,其他元素后移到空的位置。pos的默认值是表的长度加1，即默认是插入最后一个值

  ```lua
  local a= {1，8}
  table.insert(a,1,3) -- 在表索引为1处插入3,也就是将索引为1 的值后移一位
  print(a[1],a[2],a[3]) -- 3 1 8
  ```

- table.remove(table [, pos]) : 在表table中删除索引位pos（pos只能是number）的元素，并返回这个被删除的元素，它后面的元素索引都减一，pos默认是表的长度，也就是说默认删除最后一个。

  ```
  local a= {1,2,4,6}
  print(table.remove(a,1))  --1 删除索引为1的元素
  print(a[1],a[2],a[3])  --2 4 6
  ```

- table.sort(table [, comp]) : 排序（数组型table）

  ```lua
  local a = {1,7,3,4,23}
  table.sort(a)
  print(a[1],a[2],a[3],a[4],a[5]) -- 1 3 4 7 23
  
  -- 可按照给定的比较函数comp给表table排序，也就是从table[1] 到table[n],这里n表示table的长度。比较函数有两个参数，如果希望第一个参数排在第二个前面，就应该返回true，否则返回false，如果comp函数没有给出，默认从小到大
  
  local function compare(x,y) -- 从大到小排序
      return x>y
   end
  
  table.sort(a,compare)
  ```

- table.maxn(table) : 返回数组型表table中最大索引编号，如果此表没有正的索引编号，返回0

  ```lua
  local a ={}
  a[-1] = 10
  print(table.maxn(a)  --0
  
  a[5] = 10
  print(table.maxn(a))  --5
  ```

  LuaJIT2.1新增加table.new 和table.clear函数非常有用，前者用来预分配table空间，后者用来高校释放table空间。

- 判断table是否为空，有时候不小心引用了一个没有赋值的变量，这是它的值默认是nil，如果对一个nil进行索引的话会出异常。

  ```lua
  local person = {name="Bob",sex="M"}
  person = nil
  -- print(person.name) --map类型的访问可以 person.name/person["name"], 这里会报错
  
  if person ~= nil then
      print(person.name)
      print(person["name"])
  else
      print("person为空")
  end
  
  -- 其实table为nil有两种情况：
  	1、table为nil
  	2、table = {}，没有值，但是有内存地址，长度为0
  ```

- next(table [, index]) : 允许程序遍历表中的每个字段，返回下一索引和该索引的值

  参数： table为要遍历的表，index要返回的索引的迁移索引的号

  ```lua
  --要判断table是否为空
  function isTableEmpty(t)
  	return t == nil or next(t) == nil
  end
  ```



#### 变量

全局变量： 这个变量在没有被同名局部变量覆盖时候，所有代码都可见

局部变量： 该变量只在申明的代码块中可见，并且可以覆盖同名全局变量

注意： 全局变量其实本质也是一个table，它把我们创建的全局变量存放在一个table中，而这个table的名字为：**_G**

```lua
-- 定义一个全局变量
gName = " 我是全局变量"
-- 用三种方式输出
print(gName)
print(_G["gName"])
print(_G.gName)
```

虚变量： 当一个方法返回多个值时，有些返回值用不到，要是声明很多变量一一接收，显然不太合适，此时用"_"来接受

```lua
local _,finish = string.find("hello","he")  -- 我们只关心结束值

-- 在遍历时也可以使用
local t = {1,4,5}
for _, v in ipairs(t) do
    print(v)
end
```



#### Lua的时间操作

在lua中，函数time、date和difftime提供所有的日期和时间功能，在openresty的世界里，不推荐使用这里的标准时间函数，推荐使用ngx_lua模块提供的带缓存的时间接口，如：ngx.today,  ngx.time, ngx.utctime, ngx.localtime, ngx.now, ngx.http_time, 以及ngx.cookie_time等。

- os.time([table]) ：它会返回当前时间和日期的时间戳，如赋值table，表示此table指定日期的时间戳

  ```
  字段名称        取值范围
  year          四位数字
  month         1-12
  day           1-31
  hour          0-23
  min           0-59
  sec           0-59
  isdst         boolean(夏令时)
  ```

  对于time函数，如果参数为table，那么table中必须含有year,month,day字段

  ```lua
  print(os.time())  -- 1583242527
  a = {year=2020, month=1,day=30, hour=0,min=0,sec=0}
  print(os.time(a))  -- 1580313600
  ```

- os.difftime(t2,t1) ： 返回t1和t2的时间差

  ```lua
  local day1 = {year=2018, month=1, day= 30}
  local t1 = os.time(day1)
  
  local day2 = {year=2020, month=1, day= 31}
  local t2 = os.time(day2)
  print(os.difftime(t2, t1))  -- 63158400
  ```

- os.date(format, [, time]) : 把一个表示日期和时间的数值，转换成更高级的表现形式

  ```lua
  格式字符               含义
  %a                   星期中天数的简写
  %H                   24小时制的小时数
  ...
  
  print(os.date("today is %A,in %B"))
  print(os.date("now is %x %X))
  print(os.date("%Y-%m-%d %H:%M:%S"))
  print(os.date("%Y-%m-%d %H:%M:%S",1580313600))  -- 指定时间格式化输出，时间戳
          
  [[
  today is Tuesday,in March
  now is 03/03/20 21:44:55
  2020-03-03 21:44:55
  2020-01-30 00:00:00        
          ]]
          
  print(os.date("%Y-%m-%d %H:%M:%S"，os.time()))
          
  -- 使用*t，返回table格式
  t = os.date("*t",os.time())
  for i, v in pairs(t) do
      print(i, v)
  end
  [[
  sec	37
  min	49
  day	3
  isdst	false
  wday	3
  yday	63
  year	2020
  month	3
  hour	21
          ]]
  ```



#### Lua中的模块

​	从lua5.1开始，Lua加入了标准的模块管理机制，Lua的模块是由变量、函数等已知元素组成的table，因此创建一个模块很简单，就是创建一个table，然后把需要导出的变量、函数放入其中，最后返回这个table就行了

- 模块定义： 模块的文件名 和 模块定义引用名称要一致

  ```lua
  -- 文件名 modle.lua
  -- 定义一个名为model的模块
  model = {}
  
  -- 定义一个常量
  model.constant = "常量"
  
  -- 定义一个函数
  function model.func1()
  	io.write("这是一个公有函数！\n")
  end
  
  local function func2()
  	print("这是一个私有函数！")
  end
  
  function model.func3()
  	func2()
  end
  
  return model
  ```

- 引用模块

  ```lua
  print(package.path)
  require("model")  -- 需要将模块中要导出的table名和文件名一致
  print(model.constant)
  model.func3()
  model.func1()
  
  -- 如果报错，未找到相应的文件名，是requir的查找路径问题，我们可以通过下面代码查看路径
  print(package.path)
  ```

- require 函数

  lua提供了一个require的函数，用来加载模块，要加载一个模块，只需要简单地调用就可以了，例如

  ```lua
  require("模块名") 或者 require "模块名"
  
  -- 执行require后会返回一个由模块常量或函数组成的table，并且会定义一个包含该table的全局变量，和导入的模块名同名
  
  -- 如果定义的model，为local修饰为局部变量，可如下，同时模块中local model = {}
  local m = require("model")
  ```

- require 加载机制： 我们使用require命令时，系统需要知道引入哪个路径下的model.lua文件。require 用于搜索lua文件的路径是存放在全局变量 packeage.path 中，当lua启动后，会以环境变量LUA_PATH的值来初始化这个环境变量。如果没有找到该环境变量，则使用一个编译时定义的默认路径来初始化。

  lua文件路径存放在全局变量package.path中，首先去当前目录中找，但是我们运行路径下如果没有则报错。

  我们可以在自己环境变量中添加存放model.lua的路径

  ```shell
  vi /etc/profile.d/operesty.sh
  
  export LUA_PATH="/usr/local/lua/?.lua;;" # 这里的两个;; 表示新加的路径后面加上原有的默认路径，否则会覆盖
  source /etc/profile
  
  # 如果上面的方式没有效果，可以执行export LUA_PATH="/usr/local/lua/?.lua;;"这条命令添加
  ```

  

#### Lua元表

  举个例子，在lua table中我们可以访问对应的key来得到value值，但是却无法对两个table进行操作，那如何计算两个table的相加操作a+b？

- 那如何计算两个table的相加操作a+b？我们想a+b的意思是a、b中的数据合在一起。

  ```lua
  local t1 = {1,2,3}
  local t2 = {4,5,6,7}
  
  local t3 = t1 + t2  -- {1,2,3,4,5,6,7}，问有没有这种方案实现
  -- 这种类似的需求，lua提供了元表（Metatable），允许我们改变table的行为，每个行为关联了对应的元方法。
  ```

- lua提供了元表（Metatable），允许我们改变table的行为，每个行为关联了对应的元方法。

  - setmetatable(table, metatable): 对指定table设置元表（metatable)和元方法, 如果元表（metatable）中存在__metatable键值，setmetable会失败。

    ```lua
    mytable= {}   -- 普通表
    mymetatable = {}  -- 元表
    setmetatable(mytable, mymetatable)   -- 把mymetatable 设为mytable的元表
    
    -- 等价于
    mytable = setmetatable({},{})
    ```

  - getmetatable(table): 返回对象的元表（metatable）

    ```lua
    -- 返回对象元表
    getmetatable(mytable)   -- 返回mymetatable
    ```

- 元方法的命名都是以__两个下划线开头

  - __index元方法，**对表读取索引一个元方法**，这是metatable最常见的键，当你通过键来访问table的时候，如果这个键没有值，那么lua就会寻找该table的metatable（假定有metatable） [**简单的说，\_\_index就是给新增的元表添加一个索引访问的功能，t['xx']或t.xx]** ]

    ```lua
    local t = {}   -- 普通表t为空
    local other = {foo = 2}  -- 元表中有foo值
    
    setmetatable = {t,{__index=other}}  -- 把other设为t的元表__index中查找
    print(t.foo)  --2   首先回去t中找foo，如果没有则去__index元表中查找
    print(t.bar)  -- nil
    
    -- foo为什么能输出2，就是因为我们重写le__index索引的重载，lua在执行中如果没有t中没有foo，如果我们把t设置一下foo的值为3，看看结果
    
    local t = {foo=3}
    local other = {foo = 2}  -- 元表中有foo值
    setmetatable = {t,{__index=other}}  -- 把other设为t的元表__index中查找
    print(t.foo)  --3   首先回去t中找foo
    print(t.bar)  -- nil
    ```

    如果__index包含一个函数的话，lua就会调用那个函数，table和键会作为参数传递给函数。\_\_index元方法查看表中元素是否存在，如果不存在，返回nil，如果存在则由\_\_index返回

    ```lua
    local t = {key1="value"}
    
    local function metatable_func(mytable, key)  -- 此处的mytable就是调用的table，如：t。
    	if key == "key2" then
    		return "metatablevalue"
    	else
    		return nil
    	end
    end
    
    setmetatable(t,{__index=metatable_func})
    print(t.key1);  --value
    print(t.key2);  -- metatablevalue
    print(t.key3);  --nil
    ```

    总结：

    Lua查找一个表元素时的规则，其实就是3个步骤：

    - 在表中查找，如果找到，返回该元素，找不到则继续步骤2
    - 判断该表是否有元表，如果没有元表，返回nil，有元表则继续
    - 判断元表有没有__index方法，如果\_\_index方法为nil，则返回nil，如果为表，则查找，如果为函数，则执行函数

  - __newindex 元方法

    对表进行更新的，__index则用来对表进行访问的。 当你给的表进行索引进行赋值，但此索引不存在；解释器就会查找\_\_newindex 元方法是否存在，如果存在则调用\_\_newindex这个函数/值进行执行，而不会对原表进行赋值操作

    ```lua
    mymetatable = {}
    
    mytable = setmetatable({key1="value"},{__newindex=mymetatable})
    print("mytable.key1=",mytable.key1)  -- mytable.newkey=	value
    
    -- 对原表没有的索引会创建到元表中
    mytable.newkey = "新值2"
    print("mytable.newkey = ",mytable.newkey)  --mytable.newkey = nil	
    print("mymetatable.newkey=",mymetatable.newkey)  --mymetatable.newkey= 新值2	
    
    -- 对已有的索引，不会调用__newindex，只是对原表的索引进行赋值
    mytable.key1="新值1"
    print("mytable.newkey = ",mytable.key1)  --ytable.newkey = 	新值1
    print("mymetatable.newkey=",mymetatable.key1)  --mymetatable.newkey=	nil
    
    -- 如果设置了元表后，行为就发生了变化，新索引会赋值到元表中；如果没有设置元表，则所有的赋值都是基于原表来的
    ```

    也就是说如果要对表的索引赋值或更改，如果原表存在则在原表上修改，但是没有就会去元表中添加键值对。

    如果我们要对原来的table进行赋值，那我们就可以用rawset：（如果设置了元表，但是我们还是想给原表设置/添加新索引对）

    ```lua
    mytable = setmetatable({key1= "value2"}, {
        __newindex = function(t,k,v) -- 第一个参数为table，第二个参数为key，第三个参数为value
        	print(k,"触发__newindex的key")  -- key2	触发__newindex的key
            rawset(t,k,v);  -- 如果不设置rawset，则会将新值给元表
        end
    })
    
    mytable.key1 = "new value"
    mytable.key2 = 4
    print("mytable.key1= ",mytable.key1)  -- new value
    print("mytable.key2=", mytable.key2)  --4
    
    -- key2 原来是不在mytable表中的，通过元方法__newindex中函数使用了rawset，就可以对table进行赋值
    ```

- 为表添加操作符： "+"

  我们这里定义"+"这元方法，把它定义为两个table相连，如：

  ```lua
  t1 = {1,2,3}
  t2 = {4,6}
  --使得t1 +t2 相加的结果，我们想得到的是{1，2，3，4，6}，那我们如果写元表？
  -- "+"对应的元方法为__add
  
  local function add(mytable, newtable)
  	local num = table.maxn(newtable)
  	for i = 1, num do
  		table.insert(mytable, newtable[i])
  	end
  	return mytable
  end
  
  setmetatable(t1, {__add = add})  -- 这样就实现了两个table相加
  t1 = t1 + t2
  for k, v in ipairs(t1) do
      print("key=",k," value=",v)
  end
  
  ----------------------------------------------------------
  -- add的本质
  local mt = {}
  --定义mt.__add元方法（其实就是元表中一个特殊的索引值）为将两个表的元素合并后返回一个新表
  mt.__add = function(t1,t2)
      local temp = {}
      for _,v in pairs(t1) do
          table.insert(temp,v)
      end
      for _,v in pairs(t2) do
          table.insert(temp,v)
      end
      return temp
  end
  local t1 = {1,2,3}
  local t2 = {2}
  --设置t1的元表为mt
  setmetatable(t1,mt)
  
  local t3 = t1 + t2
  --输出t3
  local st = "{"
  for _,v in pairs(t3) do
      st = st..v..", "
  end
  st = st.."}"
  print(st)
  ```

   以下是我们的操作符对应的关系

  | 函数       | 描述             |
  | ---------- | ---------------- |
  | __add      | 运算符 +         |
  | __sub      | 运算符 -         |
  | __mul      | 运算符 *         |
  | __ div     | 运算符 /         |
  | __mod      | 运算符 %         |
  | __unm      | 运算符 -（取反） |
  | __concat   | 运算符 ..        |
  | __eq       | 运算符 ==        |
  | __lt       | 运算符 <         |
  | __le       | 运算符 <=        |
  | __call     | 当函数调用       |
  | __tostring | 转化为字符串     |
  | __index    | 调用一个索引     |
  | __newindex | 给一个索引赋值   |

- \_\_call :__call可以让table当做一个函数来使用。也就是table() , 这样的就是调用\_\_call() 元方法，参数第一个为调用的table，后面的可以自己传入

  ```lua
  local mt = {}
  --__call的第一参数是表自己
  mt.__call = function(mytable,...)
      --输出所有参数
      for _,v in ipairs{...} do
          print(v)
      end
  end
  
  t = {}
  setmetatable(t,mt)
  --将t当作一个函数调用
  t(1,2,3)  -- 结果： 1  2  3
  
  ------------------------------------------------
  -- 案例2：对两个数组型table，元素求和
  local function call_func(mytable,newtable)
      local sum = 0
      local i
      for i=1, table.maxn(mytable) do
          sum = sum + mytable[i]
      end
      for i = 1, table.maxn(newtable) do
          sum = sum + newtable[i]
      end
      return sum
  end
  
  local t1 = {1,2,4}
  local t2 = {2,4}
  setmetatable(t1, {__call=call_func}) -- 使得t1能被函数调用，call_func有结构多个参数，第一个时本身，后面的是括号里的参数
  
  local sum = t1(t2)
  print(sum)  --13
  
  --------------------------------------------------
  -- 案例3： 参数验证
  local function call_test(mytable, n1, n2)
      print(n1,n2)  --3,5
      return "返回值"
  end
  local t1 = {1,2}
  setmetatable(t1, {__call=call_test})
  local sum = t1(3,5)
  print("打印返回值：",sum)  --打印返回值： 返回值
  ```

- __tostring： 可以修改table转化为字符串的行为

  ```lua
  local mt = {}
  --参数是表自己
  mt.__tostring = function(t)
      local s = "{"
      for i,v in ipairs(t) do
          if i > 1 then
              s = s..", "
          end
          s = s..v
      end
      s = s .."}"
      return s
  end
  
  t = {1,2,3}
  --直接输出t
  print(t)   --table: 0x14e2050
  --将t的元表设为mt
  setmetatable(t,mt)
  --输出t
  print(t)  --{1，2，3}  字符类型
  
  ----------------------------------------
  -- 案例2：修改表的输出行为
  mt = {}
  mt.__tostring = function(mytable)
              local sum = 0
              for k,v in pairs(mytable) do
                  sum = sum +v
              end
              return "all value sum ="..sum
          end
  
  t1={1,2,4}
  setmetatable(t1,mt)
  print(t1)  -- print方法会调用table的tostring的元方法
  ```

- \_\_index ：调用table的一个不存在的索引时，会使用到元表的__index元方法，和前几个元方法不同，__index可以是一个函数也可是一个table。如果原表中没有就去__index中找（函数/表）

  ```lua
  local mt = {}
  --第一个参数是表自己，第二个参数是调用的索引
  mt.__index = function(t,key)
      return "it is "..key
  end
  
  t = {1,2,3}
  --输出未定义的key索引，输出为nil
  print(t.key)   --nil
  setmetatable(t,mt)
  --设置元表后输出未定义的key索引，调用元表的__index函数，返回"it is key"输出
  print(t.key)  -- it is key
  
  ---------------------------------------------------
  local mt = {}
  mt.__index = {key = "it is key"}
  
  t = {1,2,3}
  --输出未定义的key索引，输出为nil
  print(t.key)    --nil
  setmetatable(t,mt)
  --输出表中未定义，但元表的__index中定义的key索引时，输出__index中的key索引值"it is key"
  print(t.key)   --it is key
  --输出表中未定义，但元表的__index中也未定义的值时，输出为nil
  print(t.key2)  -- nil
  ```

- \_\_newindex：当为table中一个不存在的索引赋值时，会去调用元表中的__newindex元方法

  ```lua
  -- 赋值函数时
  local mt = {}
  --第一个参数时表自己，第二个参数是索引，第三个参数是赋的值
  mt.__newindex = function(t,index,value)
      print("index is "..index)
      print("value is "..value)
  end
  
  t = {key = "it is key"}
  setmetatable(t,mt)
  --输出表中已有索引key的值
  print(t.key)  -- it is key
  --为表中不存在的newKey索引赋值，调用了元表的__newIndex元方法，输出了参数信息
  t.newKey = 10
  --表中的newKey索引值还是空，上面看着是一个赋值操作，其实只是调用了__newIndex元方法，并没有对t中的元素进行改动
  print(t.newKey) --nil
  
  
  --赋值给一个table时，为t中不存在的索引赋值会将该索引和值赋到__newindex所指向的表中，不对原来的表进行改变。
  local mt = {}
  --将__newindex元方法设置为一个空表newTable
  local newTable = {}
  mt.__newindex = newTable
  t = {}
  setmetatable(t,mt)
  print(t.newKey,newTable.newKey)
  --对t中不存在的索引进行负值时，由于t的元表中的__newindex元方法指向了一个表，所以并没有对t中的索引进行赋值操作将，而是将__newindex所指向的newTable的newKey索引赋值为了"it is newKey"
  t.newKey = "it is newKey"
  print(t.newKey,newTable.newKey)
  ```



#### 点号与冒号操作符的区别

```lua
local str = "abcde"
print("case 1:",str:sub(1,2))  -- case 1:	ab
print("case 2:",str.sub(str,1,2))  --case 2:	ab

-- 冒号操作会带入一个self参数，用来代表自身，而点号操作，只是内容的展开，在函数定义时，使用冒号将默认接受一个self参数，而使用点号则需要显式传入self参数
obj = {x = 20}
function obj:func1()
    print(self.x)  --20
	return "ok"
end
print(obj:func1())  --ok

--等价于：
obj = {x = 20}
function obj.func1(self)
    print(self.x)  -- 20
end
print(obj.func1(obj)) 
```



#### Lua 面向对象

面向对象编程（Object Oriented Programming ，OOP）是一种非常流行的计算机编程架构



面向对象特征：

- 封装：能够把一个实体的信息、功能、响应都装在一个单独的对象中的特征
- 继承： 继承的方法允许在不改动源程序的基础上扩展功能等
- 多态： 同一操作用于不同对象，可以有不同的解释，产生不同结果
- 抽象：简化复杂的现实问题途径

Lua中面向对象：对象由属性和方法组成，lua中的面向对象是用table来描述对象的属性，function表示方法；lua中的类可以通过table + function模拟出来。

一个简单实例：以下简单的类代表矩形类， 包括两个属性：length 和 width，getArea方法用获取面积大小

新建rect.lua脚本

```lua
local rect = {length=0,width=0}

-- 派生类的方法 new，相当于init
function rect:new(length, width)
    local o = {
        --设定各个项的值
        length = length or 0,
        width = width or 0
    }
    setmetatable(o,{__index = self}) -- 这里将原有的table使用继承
    return o
end

-- 派生类的方法 getArea
function rect:getArea()
    return self.length * self.width
end

return rect
```

在引用文件中这样使用。注意package.path一定要在检索到rect.lua文件

```lua
local rect = require("rect")
local rect1 = rect:new(1,2)  --初始化，相当于init
local rect2 = rect:new(3,4)

print(rect1:getArea())  --2
print(rect2:getArea())  --12
print(rect1.length) --1
```



- 基类案例, 实现继承类

  基类shape.lua

  ```lua
  -- 创建一个shape.lua
  local shape = {name = ""}
  
  -- 创建实体对象方法 new
  function shape:new(name)
  	local o = {
          name = name or "shape"
  	}
  	setmetatable(o, {__index = self})
  	return o
  end
  
  -- 获取周长的方法 getPerimeter
  function shape:getPerimeter()
  	print("getPerimeter in shape")
  	return 0
  end
  
  -- 获取面积的方法 getArea
  function shape:getArea()
  	print("getArea in shape")
  	return 0
  end
  return shape
  ```

  派生类trangle.lua

  ```lua
  local shape = require("shape")
  
  local triangle = {}
  
  --派生类的方法 new
  function triangle:new(name,a1,a2,a3)
  	local obj = shape:new(name)  -- 创建一个shape的对象
  	
  	-- 当方法在子类中查询时,再去父类中找
  	local super_mt = getmetatable(obj)  -- 获取shape的元表
  	setmetatable(self, super_mt)  -- 给当前派生类附加shape元素（元表）
  	
  	-- 将父类的方法和属性给super属性
  	obj.super = setmetatable({},super_mt)  -- 这里还把父类的属性添加到当前类的super键中
  	
  	-- 属性赋值
  	obj.a1 = a1 or 0
  	obj.a2 = a2 or 0
  	obj.a3 = a3 or 0
  	
  	setmetatable(obj,{__index = self})
  	return obj
  end
  
  -- 派生类的方法： getPerimeter
  function triangele:getPerimeter()
  	print("getPerimeter in triangle")
  	return (self.a1 + self.a2 + self.a3)
  end
  
  -- 派生类的方法： getHalfPerimeter
  function triangle:getHalfPerimter()
  	print("getHalfPerimeter in triangle")
  	return (self.a1 + self.a2 + self.a3) /2
  end
  
  return triangle
  ```

  应用以上的派生/基类

  ```lua
  local triangle = require("triangle")
  
  local triangle1 = triangle:new("t1",1,2,4)
  local triangle2 = triangle:new("t2",2,3,4)
  
  print("t1 getPerimeter:" ,triangle1:getPerimeter())
  print("t1 getHalfPerimeter:" ,triangle1:getHalfPerimeter())
  [[ -- 运行结果
      getPerimeter in triangle
      t1 getPerimeter:	7
      getHalfPerimeter in triangle
      t1 getHalfPerimeter:	3.5
      ]]
  
  print("t2 getPerimeter:" ,triangle2:getPerimeter())
  print("t2 getHalfPerimeter:" ,triangle2:getHalfPerimeter())
  
  -- 从shape继承的方法，调用父类方法
  print(triangle2.super:getPerimeter)
  
  [[
      getPerimeter in shape
      0
  ]]
  ```



#### Lua 面向对象进阶

​	在lua原生语法特性中是不具备面向对象设计的特性。因此，要想在lua上像其他高级语言一样使用面向对象的设计方法有以下两种选择：一种是使用原生的元表(metatable)来模拟面向对象设计，另外一种则是使用第三方框架[LuaScriptoCore](https://link.jianshu.com?t=https%3A%2F%2Fvimfung.github.io%2FLuaScriptCore%2F)来实现。下面将逐一讲解这两种方式的实现过程（**以下内容将基于Lua 5.3版本进行阐述**）。

- 关于元表（metatable）

  ​	在lua中每种类型变量都可以有一个元表，而元表实际上是一个`table`，它用于定义原始值在特定操作下的行为。如果想改变一个变量在特定操作下的行为，则可以在它的元表中设置对应元方法（metamethod）。换种说法，元表就是一个变量钩子，用来钩取变量的底层处理方法（即元方法），然后改写这些方法的处理行为。

  ![1583310711348](D:\知识点复习\api网关\assets\1583310711348.png)

  | 元方法     | 说明                                                         |      |
  | ---------- | ------------------------------------------------------------ | ---- |
  | __index    | 当访问变量某个key时，如果没有对应的value，则会访问元表`__index`元方法所指定的对象。如果指定值为`table`类型，则会访问该`table`的key所对应的值；如果指定值为`function`类型，则该方法返回值作为对应key的值 |      |
  | __newindex | 当设置变量的某个key时，如果没有对应的key，则会访问元表`__newindex`元方法来处理键值设置 |      |
  | __add      | 当两个变量进行加法操作时触发，如：`var1 + var2`              |      |
  | __sub      | 当两个变量进行减法操作时触发，如：`var1 - var2`              |      |
  | __mul      | 当两个变量进行乘法操作时触发，如：`var1 * var2`              |      |
  | __div      | 当两个变量进行除法操作时触发，如：`var1 / var2`              |      |
  | __mod      | 当两个变量进行取模操作时触发，如：`var1 % var2`              |      |
  | __unm      | 当变量进行取反操作时触发，如：`~var`                         |      |
  | __pow      | 当变量进行幂操作时触发，如：`var^2`                          |      |
  | __concat   | 当两个变量进行连接时触发，如：`var1 .. var2`                 |      |
  | __eq       | 当两个变量判断是否相等时触发，如：`var1 == var2`             |      |
  | __lt       | 当一个变量判断是否小于另一个变量时触发，如：`var1 < var2`    |      |
  | __le       | 当一个`变量判断是否小于或等于另一个变量时触发，如：`var1 <= var2` |      |
  | __call     | 当变量被用作方法调用时触发，一般来说`function`类型是允许被调用的，对于其他类型默认是不能进行调用的，那么该元方法的作用就是让你的变量能够像`function`一样被调用 |      |
  | __tostring | 当使用`tostring`转换变量为字符串或者调用`print`进行打印时触发，如：`local t = {}; print (t);` |      |
  | __gc       | 当变量被回收时触发。                                         |      |
  | __mode     | 当设置`table`为弱引用`table`时使用。弱引用`table`可以让保存的key或者value为弱引用状态，方便GC标记和回收（如果非弱引用情况下，必须要table被回收时，其内部的key和value才允许GC回收） |      |

  

- 元表设置基本步骤

  ```lua
  1、先创建一个作为元表的table
  2、设置需要实现的元方法。
  3、使用setmetatable方法将元表绑定到变量中。
  
  -- 创建元表并设置元方法
  local mt = {};
  
  mt.__index = function (table, key)
    return "Hello Metatable!";
  end
  
  -- 创建实例并绑定元表
  local t = {};
  setmetatable(t, mt);
  ```

- 类型声明； 在开始构建类型前，我们先为面向对象设想一些基本的规则，这样可以避免后面参与扩展和开发的人因为理解的不一样，导致整个结构的规则混乱和不一致。根据需要我们先设定如下几点：

  ```lua
  1、类型名称首字母必须大写
  2、类型必须为全局的变量
  3、类型必须使用__index元方法指向自身
  4、类型必须使用__gc元方法进行销毁时的操作
  5、类型的属性必须使用点语法进行声明和访问
  6、类型的类方法和实例方法声明和调用必须使用冒号(:)语法声明，目的让方法都带有一个默认的self参数
  7、类型的构造方法命名为create，并且为类方法
  
  -- 声明类型
  Object = {};
  
  -- 设置__index元方法
  Object.__index = Object;
  
  -- 设置__gc元方法
  Object.__gc = function (instance)
    -- 进行对象销毁工作
    print(instance, "destroy");
  end
  
  -- 定义构造函数
  function Object:create() 
    local instance = {};
    setmetatable(instance, self);
    return instance;  
  end
  
  -- 定义实例方法
  function Object:toString()
    print (tostring(self));
  end
  
  local obj = Object:create();
  print(obj:toString());
  
  -- 解析：
  [[
  	一个全局的table变量Object来作为类型,使用了元方法__index进行自身指向，这样做的目的是使实例对象能够访问对象所定义的属性或者方法。因为__index的特点是当变量访问指定不存在的key时，就会去调用其元表的__index方法，由于__index指向就是Object，因此就会判断Object是否存在该key，并进行返回。另外如果指向的对象也设置了元表并且使用了__index，那么会继续寻找其元表的__index指向，直到最终没有设置元表的对象（这个特性在实现继承时特别关键，并且效果很好）;
  	元方法__gc在这里也使用到了，利用其特性可以轻松地知道对象实例销毁时机，可以在方法里面进行一些后续的处理，类似C++中的析构函数。在这里只是简单地打印是哪个对象被销毁。
  	接着我们讲解一下构造方法create的实现，方法中调用了setmetatable方法将self(即类型Object)元表绑定到instance这个变量，配合之前设置__index元方法，实例变量就会拥有与Object相同的一些属性和方法定义了。
  ]]
  ```

  

- 区分构造方法和类方法属性等； 在面向对象中，其实类型和实例应该都会拥有属性，即类属性和实例属性。那么如果直接在`Object`中定义属性是没有办法区分这是类属性还是实例属性的。所以，这里借鉴了javascript中的`prototype`机制。简单来说就是把类型和实例定义分离，让所有的实例属性和实例方法定义到`prototype`中。类型变量中只保留类属性和类方法。接下来开始动手改写`Object`：

  ```lua
  -- 定义类型
  Object = {};
  
  -- 设置__index元方法
  Object.__index = Object
  
  -- 定义类型属性
  Object.objCount = 0;
  
  -- 定义构造函数
  function Object:create() 
  
    Object.objCount = Object.objCount + 1;
    
    local instance = {};
    setmetatable(instance, self.prototype);
    return instance;  
  
  end
  
  -- 定义类方法
  function Object:outputObjectTag(object)
  
    print (object.tag);
  
  end
  
  -- 定义类型的prototype
  Object.prototype = {};
  
  -- 设置prototype的__index元方法
  Object.prototype.__index = Object.prototype;
  
  -- 设置prototype的__gc元方法
  Object.prototype.__gc = function (instance)
      print(instance, "destroy");
  end
  
  -- 定义实例对象属性
  Object.prototype.tag = 999;
  
  -- 定义实例对象方法
  function Object.prototype:toString()
      return tostring(self);
  end
  
  [[
  上面例子主要做出了如下几点改动：
  1、在类型Object中增加一个类属性prototype，并设置其__index元方法指向。
  2、create方法中之前设置元表为Object，现在改为Object.prototype。
  3、__gc元方法从Object转移到Object.prototype中，因为实例构造时绑定元表是prototype，所以销毁监听要相对应转移。
  4、tag属性和toString方法转移到Object.prototype中进行定义
  ]]
  
  local obj  = Object:create();
  print (obj.tag);                  -- 输出999
  Object:outputObjectTag(obj);      -- 输出999
  print (Object.objCount);          -- 输出1
  ```

- 类的继承

  ```lua
  -- 创建类型并绑定父类作为元表
  Person = {};
  Person.__index = Person;
  setmetatable(Person, Object);
  
  -- 创建prototype并绑定父类prototype作为元表
  Person.prototype = {};
  Person.prototype.__index = Person.prototype;
  setmetatable(Person.prototype, Object.prototype);
  
  local person = Person:create();
  print (person.tag)       -- 输出999
  ```

  

