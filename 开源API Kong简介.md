## 开源API Kong简介



#### 什么是API-GETWAY

​	API网关概念并非一开始就有，而是在西戎架构演变过程中，应对面临的一些问题而演化出来的，那API网关要解决什么问题？让我们来看看web应用的演变过程：

![1583637545070](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1583637545070.png)

​	随着电商的场景及系统越来越多，服务越来愈多，通过对业务的切分，逐步聚合成了业务中台的能力，那么由于各个流量及接口管理过于分散不便于整体管理，导致前后端之前的处理变得异常困难，就这样，一个前后端之间的接口管理应运而生。

![1583637890303](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1583637890303.png)



#### 应用场景：

​	业务发展过快，对接系统过多，手工难以比较好集成和管理



#### 主角出场：

​	为了解决这些问题， 业界实现了作用与前端与业务系统之间的中间层网关，即综合前置系统，由其适配各类前端和业务，处理各种协议接入、路由与保温转换、同步异步调用等；

​	这个综合前置系统，就是我们说的API网关。

![1583638293158](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1583638293158.png)

#### API网关定义：

​	网关的角色是作为一个API架构，用来保护、增强和控制对于API服务的访问

![1583638392916](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1583638392916.png)



#### Kong简介、原理和用途

​	Kong是Mashape公司开源的一款API网关产品

![1583638743918](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1583638743918.png)



Kong是客户端和服务间转发API通信的API网关，通过插件扩展功能

Kong有两个主要组件：

- Kong  Server： 基于nginx的服务器，用来接收API请求
- Apache Cassandra:   用来存储操作数据

​      你可以通过增加更多 Kong Server 机器对 Kong 服务进行水平扩展，通过前置的负载均衡器向这些机器分发请求。根据文档描述，两个Cassandra节点就足以支撑绝大多数情况，但如果网络非常拥挤，可以考虑适当增加更多节点。

​       对于开源社区来说，Kong 中最诱人的一个特性是可以通过插件扩展已有功能，这些插件在 API 请求响应循环的生命周期中被执行。插件使用 Lua 编写，而且Kong还有如下几个基础功能：HTTP 基本认证、密钥认证、CORS（ Cross-origin Resource Sharing，跨域资源共享）、TCP、UDP、文件日志、API 请求限流、请求转发以及 nginx 监控。



​	Kong是一个在Nginx运行的Lua应用程序，由lua-nginx-module实现。Kong和OpenResty一起打包发行，其中已经包含了lua-nginx-module。OpenResty不是Nginx的分支，而是一组扩展其功能的模块。基于这一点我们也可以自己通过：nginx+lua来定义实现自己的API网关



#### 网关性能比拼

微服务架构中加入API Gateway（API网关）作用：一般也会把路由，安全，限流，缓存，日志，监控，重试，熔断等都放到 API 网关来做，然后服务层就完全脱离这些东西，纯粹的做业务，也能够很好的保证业务代码的干净，不用关心安全，压力等方面的问题。

​	压力测试方案我们采用Spring Cloud Gateway Benchmark作为基础方案，该方案开源源码查看如下地址：https://github.com/exceedzhang/spring-cloud-gateway-bench，方案wrt和Jmeter测试4种部署在同一服务器上的4种不同网关，如下图所示：

​	根据官方测试结论增加geteway后性能会大幅下降，Spring Cloud Gateway性能最优，表现最好。

![1583753990244](D:\知识点复习\api网关\assets\1583753990244.png)

​	经过测试之后发现，官方结论基本正确。Spring Cloud Gateway性能最优、稳定性最好，但和Zuul性能差距相差并不如官方描述如此巨大，在这个项目基础上我们又测试Kong网关，还是访问no proxy（8000） web server，Kong网关监听80端口再转发到8000端口。测试结论：**使用Kong网关几乎没有任何性能消耗，性能到1600次/秒，增加安全鉴权后访问也没有明显性能下降**。Kong提供路由、日志、均衡负载、性能监控、安全机制已符合API Gateway功能要求，同时性能比Spring Cloud Gateway至少提供数倍。

------



#### Kong的安装

这里主要是在centos中部署

- 环境准备

  ```
  sudo yum install -y pcre pcre-devel zlib zlib-devel openssl openssl-devel
  ```

  配置讲解：

  ```
  pcre 安装
  	pcre(Perl Compatible Regular Expressions) 是一个 Perl 库，包括 perl 兼容的正则表达式，nginx 的 http 库使用 pcre 解析正则表达式。
  
  zlib 安装
  	zlib 库提供多种压缩和加压缩的方式。
  
  
  openssl 安装
  openssl 是一个请打的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议。
  ```

  

- postgresql 安装

  PostgreSQL是完全由社区驱动的开源项目，由全世界超过1000名贡献者所维护。它提供了单个完整功能的版本。可靠性是PostgreSQL的最高优先级。Kong 默认使用 postgresql 作为数据库

  ```shell
  # 添加 rpm
  yum install -y https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-centos96-9.6-3.noarch.rpm
  
  #安装 postgresql 9.5
  sudo  yum install -y postgresql95-server postgresql95-contrib
  
  #初始化数据库
  sudo /usr/pgsql-9.5/bin/postgresql95-setup initdb
  ```

  相应的命令

  ```go
  // 设置成 centos7 开机启动服务
  sudo systemctl enable postgresql-9.5.service
  // 启动 postgresql 服务
  sudo systemctl start postgresql-9.5.service
  // 查看 postgresql 状态
  sudo systemctl status postgresql-9.5.service
  ```

  ![1583642624213](D:\知识点复习\api网关\assets\1583642624213.png)

  配置Postgresql：执行完初始化任务之后，postgresql 会自动创建和生成两个用户和一个数据库

  ```shell
  linux 系统用户 postgres：     管理数据库的系统用户；
  postgresql 用户 postgres：   数据库超级管理员；
  数据库 postgres：             用户 postgres 的默认数据库。
  
  密码由于是默认生成的，需要在系统中修改一下。123
  sudo passwd postgres
  ```

  为了安全以及满足 Kong 初始化的需求，需要在建立一个 postgre 用户 kong 和对应的 linux 用户 kong，并新建数据库 kong。

  ```shell
  // 新建 linux kong 用户 
  sudo adduser kong
  
  // 使用管理员账号登录 psql 创建用户和数据库
  // 切换 postgres 用户
  // 切换 postgres 用户后，提示符变成 `-bash-4.2$` 
  su postgres
  
  // 进入 psql 控制台
  psql
  
  // 此时会进入到控制台（系统提示符变为'postgres=#'）
  // 先为管理员用户postgres修改密码
  \password postgres
  
  // 建立新的数据库用户（和之前建立的系统用户要重名）
  create user kong with password '123456';
  
  // 为新用户建立数据库
  create database kong owner kong;
  
  // 把新建的数据库权限赋予 kong
  grant all privileges on database kong to kong;
  
  // 退出控制台
  \q
  ```

  详细操作如下：

  ![ 3333333333333333333333333333333333333333333333333333331583643099697](D:\知识点复习\api网关\assets\1583643099697.png)

  登陆命令

  ```shell
  psql -U kong -d kong -h 127.0.0.1 -p 5432
  
  [root@MyHost ~]# psql -U kong -d kong -h 127.0.0.1 -p 123
  psql: could not connect to server: Connection refused
  	Is the server running on host "127.0.0.1" and accepting
  	TCP/IP connections on port 123?
  在 work 或者 root 账户下登录 postgresql 数据库会提示权限问题。
  ```

  认证权限配置文件为 `/var/lib/pgsql/9.5/data/pg_hba.conf`
  常见的四种身份验证为：

  ```
  trust： 凡是连接到服务器的，都是可信任的。只需要提供psql用户名，可以没有对应的操作系统同名用户；
  password 和 md5： 对于外部访问，需要提供 psql 用户名和密码。对于本地连接，提供 psql 用户名密码之外，还需要有操作系统访问权。（用操作系统同名用户验证）password 和 md5 的区别就是外部访问时传输的密码是否用 md5 加密；
  ident： 对于外部访问，从 ident 服务器获得客户端操作系统用户名，然后把操作系统作为数据库用户名进行登录对于本地连接，实际上使用了peer；
  peer：通过客户端操作系统内核来获取当前系统登录的用户名，并作为psql用户名进行登录。
  ```

  psql 用户必须有同名的操作系统用户名。并且必须以与 psql 同名用户登录 linux 才可以登录 psql 。想用其他用户（例如 root ）登录 psql，修改本地认证方式为 trust 或者 password 即可。

  ```shell
  vim /var/lib/pgsql/9.5/data/pg_hba.conf
  
  #修改后，把这个配置文件中的认证 METHOD的ident修改为trust，可以实现用账户和密码来访问数据库，
  # "local" is for Unix domain socket connections only
  local   all             all                                     peer
  # IPv4 local connections:
  host    all             all             127.0.0.1/32            trust
  host    all             all             0.0.0.0/0               trust
  # IPv6 local connections:
  host    all             all             ::1/128                 trust
  ```

  pgsql 默认只能通过本地访问，需要开启远程访问。
  修改配置文件 `var/lib/pgsql/9.5/data/postgresql.conf` ，将 `listen_address` 设置为 '*'。

  ```shell
  vim var/lib/pgsql/9.5/data/postgresql.conf
  
  # - Connection Settings -
  listen_addresses = '*'          # what IP address(es) to listen on;
  ```

  最后重启postgresql

  ```shell
  sudo systemctl restart postgresql-9.5.service
  
  # root尝试登陆：psql -U kong -d kong -h 127.0.0.1 -p 5432
  [root@MyHost kong]# psql -U kong -d kong -h 127.0.0.1 -p 5432
  psql (9.5.21)
  Type "help" for help.
  
  kong=> \q
  ```



- 安装kong

  ```shell
  yum install -y https://bintray.com/kong/kong-community-edition-rpm/download_file?file_path=centos/7/kong-community-edition-1.0.2.el7.noarch.rpm 
  ```

  下面修改 kong 的配置文件，默认配置文件位于 `/etc/kong/kong.conf.default`

  ```
  sudo cp /etc/kong/kong.conf.default /etc/kong/kong.conf
  ```

  将之前安装配置好的 postgresql 信息填入 kong 配置文件中：

  ```shell
  sudo vi /etc/kong/kong.conf
  
  
  #------------------------------------------------------------------------------
  # DATASTORE
  #------------------------------------------------------------------------------
  
  # Kong will store all of its data (such as APIs, consumers, and plugins) in
  # either Cassandra or PostgreSQL.
  #
  # All Kong nodes belonging to the same cluster must connect themselves to the
  # same database.
  
  database = postgres             # Determines which of PostgreSQL or Cassandra
                                   # this node will use as its datastore.
                                   # Accepted values are `postgres` and
                                   # `cassandra`.
  
  pg_host = 127.0.0.1             # The PostgreSQL host to connect to.
  pg_port = 5432                  # The port to connect to.
  pg_timeout = 5000               # Defines the timeout (in ms), for connecting,
                                   # reading and writing.
  
  pg_user = kong                  # The username to authenticate if required.
  pg_password =123                   # The password to authenticate if required.
  pg_database = kong              # The database name to connect to.
  
  pg_ssl = off                    # Toggles client-server TLS connect
  ```

- 启动kong

  默认情况下，kong 并没有添加到 $PATH 环境变量中，所以直接 kong start 并不能生效，利用 `whereis kong` 查看 kong 的命令路径：

  ```shell
  #这里bootstrap请根据kong版本选择是up还是bootstrap，初始化数据库
  [root@MyHost kong]# kong migrations bootstrap
  [root@MyHost kong]# /usr/local/bin/kong start
  Kong started
  
  #查看是否成功
  [root@MyHost kong]# curl -I -m 10 -o /dev/null -s -w '%{http_code}\n' http://localhost:8001/
  200
  
  
  ------------------------相关命令
  # kong stop     停止
  # kong reload   重启
  ```

  Kong默认监听下面端口：

  ```shell
  8000，监听来自客户端的HTTP流量，转发到你的upstream服务上。此端口是KONG用来监听来自客户端传入的HTTP请求，并将此请求转发到上有服务器；（kong根据配置的规则转发到真实的后台服务地址。
  
  8443，监听HTTPS的流量，功能跟8000一样。可以通过配置文件禁止。此端口是KONG用来监听来自客户端传入的HTTPS请求的。它跟8000端口的功能类似，转发HTTPS请求的。可以通过修改配置文件来禁止它；
  
  8001，Kong的HTTP监听的api管理接口。Admin API，通过此端口，管理者可以对KONG的监听服务进行配置，插件设置、API的增删改查、以及负载均衡等一系列的配置都是通过8001端口进行管理；
  
  8444，Kong的HTTPS监听的API管理接口。通过此端口，管理者可以对HTTPS请求进行监控；
  
  注意：浏览器访问 IP:8000，如果出现{"message":"no API found with those values"}；如果有防火墙的话，最好先关掉防火墙。
  [root@MyHost kong]# iptables -F
  ```

- 相关补充：

  - 执行完kong migrations boostrap后，进入数据库中查看生成的表

    ```shell
    [root@MyHost kong]# psql -U kong -d kong -h 127.0.0.1 -p 5432
    psql (9.5.21)
    Type "help" for help.
    
    kong=> \dt
                       List of relations
     Schema |             Name              | Type  | Owner 
    --------+-------------------------------+-------+-------
     public | acls                          | table | kong
     public | apis                          | table | kong
     public | basicauth_credentials         | table | kong
     public | certificates                  | table | kong
     public | cluster_ca                    | table | kong
     public | cluster_events                | table | kong
     public | consumers                     | table | kong
     public | hmacauth_credentials          | table | kong
     public | jwt_secrets                   | table | kong
     public | keyauth_credentials           | table | kong
     public | locks                         | table | kong
     public | oauth2_authorization_codes    | table | kong
     public | oauth2_credentials            | table | kong
     public | oauth2_tokens                 | table | kong
     public | plugins                       | table | kong
     public | ratelimiting_metrics          | table | kong
     public | response_ratelimiting_metrics | table | kong
     public | routes                        | table | kong
     public | schema_meta                   | table | kong
     public | services                      | table | kong
     public | snis                          | table | kong
     public | targets                       | table | kong
     public | ttls                          | table | kong
     public | upstreams                     | table | kong
    (24 rows)
    ```



#### 安装Kong-DashBoard图形管理

首先安装npm指令

```shell
# 下载node源码
wget https://nodejs.org/dist/v10.13.0/node-v10.13.0-linux-x64.tar.xz

# 解压
xz -d node-v10.13.0-linux-x64.tar.xz
tar -xf node-v10.13.0-linux-x64.tar

#创建软连接
ln -s ~/node-v10.13.0-linux-x64/bin/node /usr/bin/node
ln -s ~/node-v10.13.0-linux-x64/bin/npm /usr/bin/npm

# 验证
[root@MyHost ~]# npm -v
6.4.1

# 配置淘宝镜像，不然我们安装模块时很慢：

[root@MyHost ~]#npm --registry https://registry.npm.taobao.org install kong-dashboard #临时设置
[root@MyHost ~]#npm config set registry https://registry.npm.taobao.org # 持久设置
```

启动kong-dashboard

```shell
1. 首先要找到安装的kong-dashboard位置，一般在node文件下的bin下： 
[root@MyHost ~]#/root/node-v10.13.0-linux-x64/bin/kong-dashboard start  --kong-url http://localhost:8001

2. 浏览器访问：http://47.95.217.144:8080/#!/
```

![1583653653782](D:\知识点复习\api网关\assets\1583653653782.png)

- 主页面参数讲解

  ```
  routes：配置转发到的域名和地址
  services：配置被转发的域名和地址
  consumers：kong的用户管理，可以创建用户
  plugins：kong的插件，可以安装等
  cwetificates：域名的证书，https肯定有证书吧，配置在这
  upstreams：在routes外可以再配置一层，这个有待深入研究
  ```

  

- 注意，如果发生下面错误，查看是否端口被占用或kong是否启动

  ```shell
  [root@MyHost ~]# /root/node-v10.13.0-linux-x64/bin/kong-dashboard start  --kong-url http://localhost:8001
  Connecting to Kong on http://localhost:8001 ...
  Could not reach Kong on http://localhost:8001
  Error details:
  { Error: connect ECONNREFUSED 127.0.0.1:8001
      at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1113:14)
    errno: 'ECONNREFUSED',
    code: 'ECONNREFUSED',
    syscall: 'connect',
    address: '127.0.0.1',
    port: 8001 }
  ```

- Kong DashBoard相关命令

  ```shell
  # 用自定义端口启动 Kong Dashboard 
  kong-dashboard start \ --kong-url http://localhost:8001 \ --port [port]
  
  # 使用权限认证启动 Kong Dashboard
  kong-dashboard start \ --kong-url http://kong:8001 \ --basic-auth user1=password1 user2=password2
  
  # Kong Dashboard 帮助文档
  kong-dashboard start --help
  ```



#### 使用Kong-Dashboard配置

- services相关的配置

  ![1583669199211](D:\知识点复习\api网关\assets\1583669199211.png)

- 配置好service后，会生成它相应的routes，主要作用是将被反向代理的url/ip；假如我们在我们

  c:/windows/system32/drivers/etc/hosts文件中添加了一个DNS解析

  ```
  27.95.217.144 example.com
  ```

  ![1583669536860](D:\知识点复习\api网关\assets\1583669536860.png)

- 试试在浏览器中输入：<http://example.com:8000/api1/v1/> 就可以访问后端nginx的服务

  ```nginx
  location /api1 {
      default_type application/json;
      add_header Content-Type 'text/html; charset=utf-8';
      return 200 '{"success":false,"messge":"api1"}';
  }
  
  location /api1/v1 {
      default_type application/json;
      add_header Content-Type 'text/html; charset=utf-8';
      return 200 '{"success":false,"messge":"api1/v1"}';
  }
  
  location /api2 {
      default_type application/json;
      add_header Content-Type 'text/html; charset=utf-8';
      return 200 '{"success":false,"messge":"api2"}';
  }
  ```

  

#### 通过命令创建服务和API

kong的新版本把api去掉了，换成了services和routes，我这里用的是新版本，所以是services。

- 创建service

  ```shell
  curl -i -X POST \   # 用post的方式提交信息
    --url http://localhost:8001/services/ \  # 连接admin API接口
    --data 'name=test2' \   # 创建服务名
    --data 'url=http://47.95.217.144:8888/api2'   # 反向解析到我们的服务
  ```

- 为service配置routes

  ```shell
  curl -i -X POST \
    --url http://localhost:8001/services/test2/routes \  # 为test2服务添加routes
    --data 'hosts[]=example.com'  # 要被反向解析的域名
    
  #浏览器访问：http://example.com:8000/
  ```

  

