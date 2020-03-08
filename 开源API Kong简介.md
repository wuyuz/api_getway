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

