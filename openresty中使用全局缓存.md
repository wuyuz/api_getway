## openresty中使用全局缓存

#### Nginx全局内存---本地缓存

使用过如Java的朋友可能知道如Ehcache等这种进程内本地缓存。
Nginx是一个Master进程多个Worker进程的工作方式，因此我们可能需要在多个Worker进程中共享数据。



#### 使用ngx.shared.DICT来实现全局内存共享。

- 首先在nginx.conf的http部分分配内存大小

  语法：lua_shared_dict <name> <size>

  该命令主要是定义一块名为name的共享内存空间，内存大小为size。
  通过该命令定义的共享内存对象对于Nginx中所有worker进程都是可见的

  注意：当Nginx通过reload命令重启时，共享内存字典项会从新获取它的内容 （即共享内存保留）
        当Nginx退出时，字典项的值将会丢失。（即共享内存丢失）

  ```nginx
  http {
          
      lua_shared_dict dogs 10m; # 可以定义多个
      ... ...
  }
  ```

- 通过ngx.shared.DICT接口获取共享内存字典项对象

  语法：dict = ngx.shared.DICT

  ```nginx
   dict = ngx.shared[name_var]
   
  #其中，DICT和name_var表示的名称是一致的，比如上面例子中，dogs = ngx.shared.dogs 就是dict = ngx.shared.DICT的表达形式；也可以通过下面的方式达到同样的目的：dogs = ngx.shared["dogs"]
  ```

  

#### 对象操作方法

-  获取 ngx.shared.DICT.get

  ```lua
  语法：value, flags = ngx.shared.DICT:get(key)
  
   获取共享内存上key对应的值。如果key不存在，或者key已经过期，将会返回nil；
   如果出现错误，那么将会返回nil以及错误信息。
      local dogs = ngx.shared.dogs
      local value, flags = dogs:get("Marry")  ---（冒号点号）等价于 dogs.get(dogs, "Marry")
  ```

  返回列表中的flags，是在ngx.shared.DICT.set方法中设置的值，默认值为0.
  如果设置的flags为0，那么在这里flags的值将不会被返回。

-  获取包含过期的key ngx.shared.DICT.get_stale

  ```lua
   语法：value, flags, stale = ngx.shared.DICT:get_stale(key)
  
   与get方法类似，区别在于该方法对于过期的key也会返回，
   第三个返回参数表明返回的key的值是否已经过期，true表示过期，false表示没有过期。
  ```

-  设置 ngx.shared.DICT.set

  ```lua
  语法：success, err, forcible = ngx.shared.DICT:set(key, value, exptime?, flags?)
  
    "无条件"地往共享内存上插入key-value对，这里讲的"无条件"指的是不管待插入的共享内存上是否已经存在相同的key。
    三个返回值的含义：
    success：成功插入为true，插入失败为false
    err：操作失败时的错误信息，可能类似"no memory"
    forcible：true表明通过强制删除（LRU算法）共享内存上其他字典项来实现插入，
                false表明没有删除共享内存上的字典项来实现插入。
  
    第三个参数exptime表明key的有效期时间，单位是秒（s），默认值为0，表明永远不会过期。
    第四个参数flags是一个用户标志值，会在调用get方法时同时获取得到。
  
    local dogs = ngx.shared.dogs
    local succ, err, forcible = dogs:set("Marry", "it is a nice cat!")
  ```

-  安全设置 ngx.shared.DICT.safe_set

  ```
  语法：ok, err = ngx.shared.DICT:safe_set(key, value, exptime?, flags?)
  
   与set方法类似，区别在于不会在共享内存用完的情况下，通过强制删除（LRU算法）的方法实现插入。
   如果内存不足，会直接返回nil和err信息"no memory"
  
  
  注意：set和safe_set共同点是：如果待插入的key已经存在，那么key对应的原来的值会被新的value覆盖！
  ```

-  增加 ngx.shared.DICT.add

  ```
   语法：success, err, forcible = ngx.shared.DICT:add(key, value, exptime?, flags?)
  
   与set方法类似，与set方法区别在于不会插入重复的键（可以简单认为add方法是set方法的一个子方法），
   如果待插入的key已经存在，将会返回nil和和err="exists"
  ```

- 安全增加 ngx.shared.DICT.safe_add

  ```
  语法：ok, err = ngx.shared.DICT:safe_add(key, value, exptime?, flags?)
  
   与safe_set方法类似，区别在于不会插入重复的键（可以简单认为safe_add方法是safe_set方法的一个子方法），如果待插入的key已经存在，将会返回nil和err="exists"
  ```

-  替换 ngx.shared.DICT.replace

  ```
   语法：success, err, forcible = ngx.shared.DICT:replace(key, value, exptime?, flags?)
  
   与set方法类似，区别在于只对已经存在的key进行操作（可以简单认为replace方法是set方法的一个子方法），
   如果待插入的key在字典上不存在，将会返回nil和错误信息"not found"
  ```

-  删除 ngx.shared.DICT.delete

  ```
  语法：ngx.shared.DICT:delete(key)
  
  无条件删除指定的key-value对，其等价于：ngx.shared.DICT:set(key, nil)
  ```

- 自增 ngx.shared.DICT.incr

  ```
  语法：newval, err = ngx.shared.DICT:incr(key, value)
  
   对key对应的值进行增量操作，增量值是value，其中value的值可以是一个正数，0，也可以是一个负数。
   value必须是一个Lua类型中的number类型，否则将会返回nil和"not a number"；
   key必须是一个已经存在于共享内存中的key，否则将会返回nil和"not found".
  ```

- 清除 ngx.shared.DICT.flush_all

  ```
   语法：ngx.shared.DICT:flush_all()
  
   清除字典上的所有字段，但不会真正释放掉字段所占用的内存，而仅仅是将每个字段标志为过期。
  ```

- 清除过期内存 ngx.shared.DICT.flush_expired

  ```
   语法：flushed = ngx.shared.DICT:flush_expired(max_count?)
  
   清除字典上过期的字段，max_count表明上限值，如果为0或者没有给出，表明需要清除所有过期的字段，
   返回值flushed是实际删除掉的过期字段的数目。
  
   注意：与flush_all方法的区别在于，该方法将会释放掉过期字段所占用的内存。
  ```

- 获取keys  ngx.shared.DICT.get_keys

  ```
  语法：keys = ngx.shared.DICT:get_keys(max_count?)
  
  从字典上获取字段列表，个数为max_count，如果为0或没有给出，表明不限定个数。默认值是1024个
  
  注意：强烈建议在调用该方法时，指定一个max_count参数，因为在keys数量很大的情况下，
  如果不指定max_count的值，可能会导致字典被锁定，从而阻塞试图访问字典的worker进程。
  ```



##### 案例演示

- 在nginx.conf中的http模块下设置

  ```
  http部分配置共享内存
  lua_shared_dict shared_data 10m;
  ```

- 编辑test.lua

  ```lua
  --1、获取全局共享内存变量
  local shared_data = ngx.shared.shared_data
  
  --2、获取字典值
  local i = shared_data:get("i")
  if not i then
      i = 1
      --3、赋值
      shared_data:set("i", i)
      ngx.say("set i ", i, "<br/>")
  end
  --递增
  i = shared_data:incr("i", 1)
  ngx.say("i=", i, "<br/>")  
  ```

  

