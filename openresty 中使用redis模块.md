## openresty 中使用redis模块



在高并发的场景下，我们常用到缓存技术，现在我们常用的分布式缓存redis最知名。

操作redis,我们需要引入redis模块，require "resty.redis"; 我们现在做个可以操作redis进行赋值。

#### 连接redis服务器

- redis的简单使用

  ```lua
  -- test.lua文件中
  -- 定义redis关闭连接的方法
  local function close_redis(red)
  	if not red then
  		return
  	end
  	local ok, err = red:close()
  	if not ok then
  		ngx.say("close redis error: ", err)
  	end
  end
  
  local redis = require "resty.redis"  -- 引入redis模块, 可以在lualib中找到
  local red = redis:new()  -- 创建一个对象，注意用冒号调用
  -- 设置超时毫秒
  red:set_timeout(1000)
  -- 连接创建
  local ip="10.171.25.174"
  local port = 6379
  local ok, err = red:connect(ip, port)
  if not ok then
      ngx.say("connect to redis error: ",err)
      return close_redis(red)
  end
  
  [[  -- redis密码认证机制
  ok,err = red:auth("1234")
  if not ok then
  	ngx.say("fail to auth:",err)
  	return close_redis(red)
  end
  ]]
  
  -- 调用api设置key
  ok, err = red:set("msg", "hello world")
  if not ok then
      ngx.say("set msg error:",err)
      return close_redis(red)
  end
  -- 调用api获取key值
  local res,err = red:get("msg")
  if not res then
      ngx.say("get msg error: ",err)
    	return close_redis(red)
  end
  
  -- 判断返回值为null
  if res == ngx.null then
      res = "该键没有值"
  end
  
  ngx.say("msg: ",res)
  close_redis(red)
  ```

  注意：如果连接redis报错

  ```nginx
  方案一：在redis.conf中注释 #bind 127.0.0.1； 同时将protected-mode  no
  
  方案二：启动redis后，redis-cli进入：输入：CONFIG SET protected-mode no
  
  -----------------------------------------------------------------
  连接授权redis,在redis.conf配置文件中配置认证密码
  requirepass 12345
  
  如果添加了密码后的验证：red:auth("123")
  ```

- redis连接池

  redis的连接是tcp连接，建立tcp连接需要三次握手，而释放tcp连接需要四次挥手，而这些往返时延仅需要一次我们应该复用tcp连接，此时就可以考虑使用连接池，即连接池可以复用连接。我们需要close_redis函数改写

  ```lua
  local function close_redis(red)
  	if not red then
  		return
  	end
  	
  	-- 释放连接（连接池实现）
  	local pool_max_idle_time = 10000 -- 每个连接空闲时长毫秒
  	local pool_size = 100  --连接池大小
  	local ok, err = red:set_keepalive(pool_max_idle_time,pool_size)
  	if not ok then
  		ngx.say("set keepalive error: ",err)
  	end
  end
  
  [[
     即设置空闲连接超时时间防止一直占用不释放:设置连接池大小来复用连接,注意：
     	1、连接池是每个worker进程的，而不是每个server
     	2、当连接超过最大连接池大小时，会按照LRU算法回收空闲连接为新的使用
     	3、连接池中的空闲连接出现异常时自动被移除
     	4、连接池时通过ip和port标识的，即相同的ip和port会使用同一个连接池
  ]] 
  ```

  注意： 我们可以通过red:get_reused_times来判断，当前连接是否是从内建连接池中获取的，该方法返回0表示该链接没有被重复使用，如果连接来自连接池，则非零（返回重复使用的次数），放在连接池的连接不需要认证（已经认证过了  ）。

- 优化连接，通过get_reused_times() 函数区分连接池对象

  ```lua
  -- test.lua文件中
  -- 连接池
  local function close_redis(red)
  	if not red then
  		return
  	end
  	
  	-- 释放连接（连接池实现）
  	local pool_max_idle_time = 10000 -- 每个连接空闲时长毫秒
  	local pool_size = 100  --连接池大小
  	local ok, err = red:set_keepalive(pool_max_idle_time,pool_size)
  	if not ok then
  		ngx.say("set keepalive error: ",err)
  	end
  end
  
  local redis = require "resty.redis"  -- 引入redis模块, 可以在lualib中找到
  local red = redis:new()  -- 创建一个对象，注意用冒号调用
  -- 设置超时毫秒
  red:set_timeout(1000)
  -- 连接创建
  local ip="10.171.25.174"
  local port = 6379
  local ok, err = red:connect(ip, port)
  if not ok then
      ngx.say("connect to redis error: ",err)
      return close_redis(red)
  end
  
  -- 优化连接
  local count,err = red:get_reused_times()
  if 0==count then  -- 新建连接，认证
  	ok,err = red:auth("1234")
  	if not ok then
  		ngx.say("fail to auth:",err)
  		return close_redis(red)
  	end
  elseif err then  -- 从连接池来到，不需要认证
  	ngx.say("failed to get reused times: ", err)
  	return
  end
  
  -- 调用api设置key
  ok, err = red:set("msg", "hello world")
  if not ok then
      ngx.say("set msg error:",err)
      return close_redis(red)
  end
  -- 调用api获取key值
  local res,err = red:get("msg")
  if not res then
      ngx.say("get msg error: ",err)
    	return close_redis(red)
  end
  
  -- 判断返回值为null
  if res == ngx.null then
      res = "该键没有值"
  end
  
  ngx.say("msg: ",res)
  close_redis(red)
  ```




#### 项目中使用redis模块

​	在关于web+lua+openresty开发中，项目中会大量操作redis。 在上面的我们写的代码中如果没使用一次redis就重复一段代码，这样会显得很冗余，所以我们要进行二次封装redis模块。

- 在/usr/local/openresty/lualib/中新增一个文件二次封装redis.lua,  新文件redis_iresty.lua

  ```lua
  -- Copyright (C) Yichun Zhang (agentzh), CloudFlare Inc.
  
  local redis_c = require "resty.redis"
  
  
  local ok, new_tab = pcall(require, "table.new")
  if not ok or type(new_tab) ~= "function" then
      new_tab = function (narr, nrec) return {} end
  end
  
  
  local _M = new_tab(0, 155)
  _M._VERSION = '0.20'
  
  
  local commands = {
      "append",            "auth",              "bgrewriteaof",
      "bgsave",            "bitcount",          "bitop",
      "blpop",             "brpop",
      "brpoplpush",        "client",            "config",
      "dbsize",
      "debug",             "decr",              "decrby",
      "del",               "discard",           "dump",
      "echo",
      "eval",              "exec",              "exists",
      "expire",            "expireat",          "flushall",
      "flushdb",           "get",               "getbit",
      "getrange",          "getset",            "hdel",
      "hexists",           "hget",              "hgetall",
      "hincrby",           "hincrbyfloat",      "hkeys",
      "hlen",
      "hmget",             --[[ "hmset", ]]     "hscan",
      "hset",
      "hsetnx",            "hvals",             "incr",
      "incrby",            "incrbyfloat",       "info",
      "keys",
      "lastsave",          "lindex",            "linsert",
      "llen",              "lpop",              "lpush",
      "lpushx",            "lrange",            "lrem",
      "lset",              "ltrim",             "mget",
      "migrate",
      "monitor",           "move",              "mset",
      "msetnx",            "multi",             "object",
      "persist",           "pexpire",           "pexpireat",
      "ping",              "psetex",       --[[ "psubscribe", ]]
      "pttl",
      "publish",      --[[ "punsubscribe", ]]   "pubsub",
      "quit",
      "randomkey",         "rename",            "renamenx",
      "restore",
      "rpop",              "rpoplpush",         "rpush",
      "rpushx",            "sadd",              "save",
      "scan",              "scard",             "script",
      "sdiff",             "sdiffstore",
      "select",            "set",               "setbit",
      "setex",             "setnx",             "setrange",
      "shutdown",          "sinter",            "sinterstore",
      "sismember",         "slaveof",           "slowlog",
      "smembers",          "smove",             "sort",
      "spop",              "srandmember",       "srem",
      "sscan",
      "strlen",       --[[ "subscribe", ]]      "sunion",
      "sunionstore",       "sync",              "time",
      "ttl",
      "type",         --[[ "unsubscribe", ]]    "unwatch",
      "watch",             "zadd",              "zcard",
      "zcount",            "zincrby",           "zinterstore",
      "zrange",            "zrangebyscore",     "zrank",
      "zrem",              "zremrangebyrank",   "zremrangebyscore",
      "zrevrange",         "zrevrangebyscore",  "zrevrank",
      "zscan",
      "zscore",            "zunionstore",       "evalsha"
  }
  
  local mt = { __index = _M }
  
  
  local function is_redis_null(res)
  	if type(res)== "table" then
  		for k, v in pairs(res) do
  			if v ~= ngx.null then
  				return false
  			end
  		end
  		return true
  	elseif res == ngx.null then
  		return true
  	elseif res == nil then	
  		return true
  	end
  	return false
  end
  
  local function _M.close_redis(self,redis)
  	if not redis then
  		return
  	end
  	
  	-- 释放连接（连接池实现）
  	local pool_max_idle_time = 10000 -- 毫秒
  	local pool_size = 100 --连接池大小
  	local ok, err = redis:set_keepalive(pool_max_idle_time,pool_size)
  	if not ok then
  		ngx.say("set keepalive error: ",err)
  	end
  end
  
  function _M.connect_mod(self, redis)
  	redis:set_timeout(self.timeout)
  	local ok, err = redis:connect(self.ip,self.port)
  	if not ok then 
  		ngx.say("connect to redis error: ",err)
  		return self:close_redis(redis)
  	end
  	
  	if self.password then  -- 密码认证
  		local count, err = redis:get_reused_times()
  		if 0== count then
  			ok, err = redis:auth(self.password)
  			if not ok then
  				ngx.say("faild to auth: ",err)
  				return
  			end
  		elseif err then
  			ngx.say("failed to get reused times: ",err)
  			return
  		end
  	end
  	return ok, err
  end
  
  function _M.init_pipeline(self)
  	self._reqs = {}
  end
  
  function _M.commit_pipeline(self)
  	local reqs = self._reqs
  	if nil == reqs or 0== #reqs then
  		return {}, "no pipeline"
  	else
  		self._reqs = nil
  	end
  	
  	local redis, err = redis_c:new()
  	if not redis then
  		return nil, err
  	end
  	
  	local ok, err = self:connect_mod(redis)
  	if not ok then
  		return {}, err
  	end
  	
  	redis:init_pipeline()
  	for _, vals in ipairs(reqs) do
  		local fun = redis[vals[1]]
  		table.remove(vals,1)
  		fun(redis, unpack(vals))
  	end
  	
  	local results, err = redis:commit_pipeline()
  	if not results or err then	
  		return {}, err
  	end
  	
  	if is_redis_null(results) then
  		results = {}
  		ngx.log(ngx.WARN, "is null")
  	end
  	
  	self:close_redis(redis)
  	for i, value in ipairs(results) do
  		if is_redis_null(value) then
  			results[i] = nil
  		end
  	end
  	return results, err
  end
  
  local function do_command(self, cmd,...)
  	if self._reqs then
  		table.insert(self._reqs, {cmd,...})
  		return 
  	end
  	
  	local redis, err = redis_c:new()
  	if not redis then
  		return nil, err
  	end
  	
  	local ok,err = self.connect_mod(redis)
  	if not ok or err then
  		return nil, err
  	end
  	
  	redis:select(self.db_index)
  	
  	local fun = redis[cmd]
  	local result, err = fun(redis,...)
  	if not result or err then
  		return nil, err
  	end
  	
  	if is_redis_null(result) then	
  		result = nil
  	end
  	
  	self:close_redis(redis)
  	return result,err
  end
  
  for i = 1, #commands do
  	local cmd = commands[i]
  	_M[cmd] = function (self, ...)
  		return do_command(self,cmd,...)
  	end
  end
  
  function _M.new(self, opts)
  	opts = opts or {}
  	local timeout = (opt.timeout and opts.timeout * 1000) or 1000
  	local db_index = opts.db_index or 0
  	local ip = opts.ip or '127.0.0.1'
  	local port = opts.port or 6379
  	local password = opts.password or ''
  	local pool_max_idle_time = opts.pool_max_idle_time or 60000
  	local pool_size = opts.pool_size or 100
  	
  	return setmetatable({
  		timeout = timeout,
  		db_index = db_index,
  		ip = ip,
  		port = port,
  		password = password,
  		pool_max_idle_time = pool_max_idle_time
  		pool_size = pool_size
  		_reqs = nil
  	},mt)
  end
  
  return _M
  ```

- 案例：使用二次封装的redis_iresty.lua文件

  ```lua
  -- test1.lua中
  local redis = require "resty.redis_iresty"
  
  local opts = {
      ip = "47.95.217.144",
      port = 6379,
      --password="123",
      db_index = 1
  }
  
  local red = redis:new(opts)
  local ok,err = red:set("dog", "an animal")
  if not ok then
  	ngx.say("failed to set dog:",err)
  	return
  end
  
  ngx.say("set result: ",ok)
  ```

  

#### openresty使用mysql

​	编写个案例操作mysql数据库，编辑test.lua， 学习网址：https://github.com/openresty/lua-resty-mysql

- 定义一个连接mysql的数据库

  ```lua
  -- 定义关闭mysql的lianjei
  
  local function close_db(db)
  	if not db then
  		return
  	end
  	
  	db:close()
  end
  -- 引入mysql模式
  local mysql = require("resty.mysql")
  -- 创建实例
  local db,err = mysql:new()
  if not db then
  	ngx.say("new mysql error: ",err)
  	return 
  end
  
  --设置超时时间（毫秒）
  db:set_timeout(1000)
  
  --连接属性定义
  local props = {
  	host = "127.0.0.1",
  	port = 3306,
  	database = "test",
  	user = 'root',
  	password = '123',
  	charset = 'utf8'
  }
  
  local res,err, errno,sqlstate = db:connect(props)
  
  if not res then
  	ngx.say("connect to mysql error: ", err, "errno:",errno)
  	return close_db(db)
  end
  
  ngx.say("================删除表================","<br/>")
  --删除表
  local drop_table_sql = "drop table if exists user"  -- 删除user表
  res, err, errno, sqlstate = db:query(drop_table_sql)  -- query就是执行语句
   
  if not res then
  	ngx.say("drop to mysql error: ", err, "errno:",errno)
  	return close_db(db)
  end
  
  ngx.say("================新建表================","<br/>")
  -- 创建表
  local create_table_sql = "create table user(id int primary key auto_increment,ch varchar(100))"  -- sql语句
  res, err, errno, sqlstate = db:query(create_table_sql)  -- query就是执行语句
   
  if not res then
  	ngx.say("create to mysql error: ", err, "errno:",errno)
  	return close_db(db)
  end
  
  ngx.say("================插入数据================","<br/>")
  -- 插入数据
  local insert_table_sql = "insert into user(ch) values('hello')"  -- sql语句
  res, err, errno, sqlstate = db:query(insert_table_sql)  -- query就是执行语句
  
  if not res then
  	ngx.say("insert to mysql error: ", err, "errno:",errno)
  	return close_db(db)
  end
  ngx.say("insert row:", res.affected_rows,"id:",res.insert_id,"<br/>")
  
  ngx.say("================更新数据================","<br/>")
  -- 更新数据
  local update_table_sql = "update user set ch='hello2' where id ="..res.insert_id  -- sql语句
  res, err, errno, sqlstate = db:query(update_table_sql)  -- query就是执行语句
   
  if not res then
  	ngx.say("update to mysql error: ", err, "errno:",errno)
  	return close_db(db)
  end
  
  ngx.say("================查询数据================","<br/>")
  -- 查询数据
  local select_table_sql = "select id, ch from user" -- sql语句
  res, err, errno, sqlstate = db:query(select_table_sql)  -- query就是执行语句
   
  if not res then
  	ngx.say("select to mysql error: ", err, "errno:",errno)
  	return close_db(db)
  end
  
  for i, row in ipairs(res) do
  	for name, val in pairs(row) do
  		ngx.say("id: ",name,"  ",val)
  	end
  end
  
  close_db(db)
  ```

  注意：对应新增/修改/删除都会返回如下格式的数据

  ```lua
  {
      insert_id = 0,
      server_status = 2,
      warning_count = 1,
      affected_rows = 1,
      message = nil
  }
  
  对于查询会返回如下格式：
  {
      {id=1, ch="xxx"},
      {id=2, ch="xxx"},
  }
  -- null会返回ngx.null
  ```

- 防止sql注入；没有提供占位符，只能通过转义

  ```lua
  --防止sql注入
  local ch_param = ngx.req.get_uri_args()["ch] or ''
  
  --使用ngx.quote_sql_str防止注入
  local query_sql = "select id, ch from user where ch= " ..ngx.quote_sql_str(ch_param)
  res, err, errno, sqlstate = db:query(query_sql)  -- query就是执行语句
   
  if not res then
  	ngx.say("select to mysql error: ", err, "errno:",errno)
  	return close_db(db)
  end
  
  for i, row in ipairs(res) do
  	for name, val in pairs(row) do
  		ngx.say("id: ",name,"  ",val)
  	end
  end
  ```

  