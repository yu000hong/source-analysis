# 一些问题

### 为啥“持久化的消息不主动推送消息到客户端”？

客户端消息发到IM服务器之后，IM服务器会给接收客户端发送MSG_SYNC_NOTIFY，然后让客户端再主动去拉，为啥不直接返回MSG_IM呢？

这样做的目的是严格控制消息顺序的，客户端接受到的消息顺序和服务端存储的顺序会一致。只有不持久化的消息才会即时发送到接收客户端。

### 目前是不是只有customer_client才支持消息的非持久化？

我看代码中，只有CustomClient在处理消息时才会去判断MESSAGE_FLAG_UNPERSISTENT标志，如果该标志已经设置，那么就不会调用IMS的存储服务。

只有customer_client才支持消息的非持久化！

### IM终端服务器可以不用重启直接扩缩容，但是IM与IMS和IMR的连接都是靠哈希取模来选择对应的IMS或IMR。采取什么样的方案可以不用停服扩缩容？

IMR扩缩容的话，IM必须修改对应的路由服务器地址配置，然后重启所有IM服务器。

IMS扩缩容的话，IM必须修改对应的存储服务器地址配置，然后重启所有IM服务器；同时，还必须迁移消息数据文件和消息索引文件！

### 如果某IM终端服务器挂掉之后，重新启动后会影响后续的服务么？需要人工介入么？


### 如果某IMR路由服务器挂掉之后，重新启动后会影响后续的服务么？需要人工介入么？


### 如果某IMS消息服务器挂掉之后，重新启动后会影响后续的服务么？需要人工介入么？

### 超级群和普通群有什么区别？存储路由需要进行何种特殊处理？

### 为何系统中使用了多种RPC机制？

目前看到的代码中包含两种RPC：

- custom：自定义协议
- gorpc：https://github.com/valyala/gorpc 

终端 <-> IM：custom

IM <-> IMR：custom

IM <-> IMS：gorpc

**回答**

> 因为终端和IM之间、IM与IMR之间需要维持长连接来保持状态。
> IM终端服务器维护了连接到此服务器的各个终端的路由信息，当终端下线时，会删除路由信息中的相关条目。
> IMR路由服务器维护了连接到此服务器的各个IM服务器的路由信息，当IM连接中断时，会删除路由信息中的相关条目。
> 
> 但是，IMS也会与IM建立长连接的呀（im.go L516），为啥它不用自定义协议，而选择了gorpc呢？难道gorpc只能进行RPC调用，而接收不到连接建立事件或者连接断开事件吗？如果gorpc无法处理连接事件的话，肯定就无法维护相关路由信息了，肯定无法满足终端与IM、IM与IMR之间的RPC需求了；如果gorpc能处理此类连接事件的话，那么为什么还要自定义协议呢？

### 如果终端接收到了MSG_SYNC_GROUP_BEGIN消息，但是未接收到MSG_SYNC_GROUP_END消息，终端如何处理？同样地，如果终端收到了MSG_SYNC_BEGIN消息，但是未接收到MSG_SYNC_END消息，终端如何处理？

