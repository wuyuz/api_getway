## Kong 代理详解



Kong侦听四个端口的请求，默认情况是：

8000：此端口是Kong用来监听来自客户端的HTTP请求的，并将此请求转发到您的上游服务。这也是本教程中最主要用到的端口。

8443：此端口是Kong监听HTTP的请求的端口。该端口具有与8000端口类似的行为，但是它只监听HTTPS的请求，并不会产生转发行为。可以通过配置文件来禁用此端口。

8001：用于管理员对KONG进行配置的端口。

8444：用于管理员监听HTTPS请求的端口。

在本文中，我们将介绍Kong的路由功能，并详细说明8000端口上的客户端请求如何根据请求头、URI或HTTP被代理到配置中的上游服务。

先注册个API，然后跟着COPY几个命令玩玩：

### 入门示例

先做一个最简单的转发。当访问8000端口时，自动转发到<http://api01.bitspaceman.com:8000/news/qihoo>。

1、先创建两个Service：

```kotlin
curl -i -X POST \
  --url http://localhost:8001/services/ \
  --data 'name=example-service' \
  --data 'url=http://api01.bitspaceman.com:8000/news/qihoo'

curl -i -X POST \
  --url http://localhost:8001/services/ \
  --data 'name=163-service' \
  --data 'url=http://3g.163.com/touch/jsonp/sy/recommend'
```

2、然后，添加两个Route：

```kotlin
curl -i -X POST \
  --url http://localhost:8001/services/example-service/routes \
  --data 'hosts[]=news.com'

curl -i -X POST \
  --url http://localhost:8001/services/163-service/routes \
  --data 'paths[]=/news'  \
  --data 'hosts[]=news.com'
```

3、最后，访问一下：



![img](https:////upload-images.jianshu.io/upload_images/13236281-00c0d1a055665320.png?imageMogr2/auto-orient/strip|imageView2/2/w/1014/format/webp)

根据hosts=news.com做转发



![img](https:////upload-images.jianshu.io/upload_images/13236281-be59ed36d43dae27.png?imageMogr2/auto-orient/strip|imageView2/2/w/924/format/webp)

根据hosts=news.com和paths=/news做转发

原理，等同于Nginx的location。后面介绍下详细的用法。



### hosts属性

可以设置多个host，像下面这样：

```ruby
$ curl -i -X POST http://localhost:8001/routes/ \
    -H 'Content-Type: application/json' \
    -d '{"hosts":["example.com", "foo-service.com"]}'

# 或者

$ curl -i -X POST http://localhost:8001/routes/ \
    -d 'hosts[]=example.com' \
    -d 'hosts[]=foo-service.com'
```

也可以使用通配符：

```json
{
    "hosts": ["*.example.com", "service.com"]
}
```



### paths属性

可以设置多个`path`:

```json
{
    "paths": ["/service", "/hello/world"]
}
```

还可以使用正则表达式：

```json
{
    "paths": ["/users/\d+/profile", "/following"]
}
```

给正则设置优先级：

```json
[
    {
        "paths": ["/status/\d+"],
        "regex_priority": 0
    },
    {
        "paths": ["/version/\d+/status/\d+"],
        "regex_priority": 6
    },
    {
        "paths": ["/version"],
        "regex_priority": 3
    },
]
```

优先级别如下：
 1、/version
 2、/version/\d+/status/\d+
 3、/status/\d+

**如何捕获正则分组？**

如下面一个path：

```ruby
/version/(?<version>\d+)/users/(?<user>\S+)
```

支持这样一个请求：

```undefined
/version/1/users/john
```

还可以被插件使用：

```bash
local router_matches = ngx.ctx.router_matches
-- router_matches.uri_captures is:
-- { "1", "john", version = "1", user = "john" }
```

**Path添加字符的方式 ?**

```kotlin
$ curl -i -X POST http://localhost:8001/routes \
    --data-urlencode 'uris[]=/status/\d+'
```



### preserve_host属性

当使用代理的时候，Kong的默认（false）是将上游请求的Host头设置为API的upstream_url属性的主机名。

```json
{
    "name": "my-api",
    "upstream_url": "http://my-api.com",
    "hosts": ["service.com"],
}
```

客户端请kong的请求头：

```undefined
GET / HTTP/1.1
Host: service.com
```

设置为false，kong将从upstream_url中提取主机名作为HOST的值去请求上游服务。

```undefined
GET / HTTP/1.1
Host: my-api.com
```

设置为true，客户端请求的HOST通过kong透传到上游服务，而不是从upstream_url提取。

```undefined
GET / HTTP/1.1
Host: service.com
```



### strip_uri属性

指定uri前缀去匹配一个API，但是不包含在上游的请求中。这个参数接收一个boolean的值。

| uris     | strip_uri | 客户端请求         | 上游请求           |
| :------- | :-------- | :----------------- | :----------------- |
| /mockbin | false     | /some_path         | not proxied        |
| /mockbin | false     | /mockbin           | /mockbin           |
| /mockbin | false     | /mockbin/some_path | /mockbin/some_path |
| /mockbin | true      | /some_path         | not proxied        |
| /mockbin | true      | /mockbin           | /                  |
| /mockbin | true      | /mockbin/some_path | /some_path         |



### method属性

就是GET、POST、PUT、DELETE等等，不多说了。

作者：DreamsonMa