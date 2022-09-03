---
title: "利用内网穿透解决nacos内网问题"
date: 2022-08-30T20:51:52+08:00
description: "利用内网穿透解决nacos与服务在两个内网环境中时，无法互通的问题"
tags: 
  - "nacos"
  - "proxy"
  - "devops"
---

## 背景

### 技术栈

* java
* spring-cloud
* nacos

### 办公地点

* 3个不同的省市自治区

## 问题

由于开发环境部署在A市的机房中，于是其他市的开发就无法接入开发环境进行联调，特别是遇到B市的后端与C市的前端共同开发需求的情况。

核心问题其实就两个：

1. A市外的机器要访问nacos把服务注册到nacos中
2. 开发环境中gateway需要访问到外部的机器

## 解决方案

最后解决的时候发现，上面两个问题其实总结下来，依旧是内网穿透的问题，所以需要一个公网机器中转即可

### 注册nacos

这一部分其实相对简单，因为开发过程中总要看看数据库什么的，所以很早就使用了frp进行内网穿透

frps配制，其实frps本身并没有什么特殊需要配制的东西，主要就是port和token注意一下就好

主要还是frpc部分

```ini
...
#frpc的基础部分，ip啊port啊这些就直接省略了
#增加ssh映射
[nacos]
type = tcp
local_ip = 192.168.0.15	#内网的ip地址，由于内网存在多台机器，我只需要将frpc安装在一台机器上，由此机器进行内网中转即可
local_port = 2222				#内网机器端口
use_encryption = false	
use_compression = false
# remote port listen by frps
remote_port = 150			#公网机器开发的端口
```

这样设置好之后，访问公网的150端口就可以访问到 `192.168.0.15`就可以访问15机器了

```bash
ssh -p150 root@111.111.111.111
```

此处可以使用 `ssh -D` 参数命令打开socks代理端口，由于我没有通过此方案，就没有拿到命令参数了

这里抛砖引玉一下，其实可以使用现有的一些软件，进行规则分流，比如我要访问 `192.168.0.15` 的时候，通过xx线路走

### gateway访问外网服务

这个确实困扰我了好久，可以说好几个月了，主要还是以前我们这里负责的需求都是从mq里获取数据，正好不用通过gateway来转发，所以也一直没怎么上心

最近接的需求各种支付宝/微信的http异步通知，调试起来太痛苦了，前两天在看 [goproxy](https://github.com/snail007/goproxy) 文档的时候，突然意识到一个问题，gateway访问我不也是一个内网穿透么？我是内网，他访问我罢了，于是反向做一个内网穿透做一个就好了

goproxy[详细文档](https://snail007.github.io/goproxy/manual/zh/#/?id=_4%e5%86%85%e7%bd%91%e7%a9%bf%e9%80%8f)

在这里我使用了高级用法一的方案

1. 公网机器执行

```bash
proxy bridge -p ":33080" -C proxy.crt -K proxy.key
```

2. A市内网`192.168.0.15`机器上执行

```bash
proxy server -r ":19551@:9551" -P "111.111.111.111:33080" -C proxy.crt -K proxy.key
```

3. 开发机器上执行

```bash
proxy client -P "111.111.111.111:33080" -C proxy.crt -K proxy.key
```

4. 设置spring-cloud微服务启动参数

```bash
java  ... -Dspring.cloud.nacos.discovery.ip=192.168.0.15 -Dspring.cloud.nacos.discovery.port=19551 -Dspring.cloud.nacos.discovery.metadata.version=test ...
# 此处主要是修改在nacos注册时的ip/port/version
# 修改启动命令而不是配置文件，主要原因还是这个配制过于本地化了，并不通用免得不小心提交了
# idea可以直接设置vm参数来进行
# 这里的version也说明一下，主要用于与前端联调，在gateway收到请求后，会通过version向nacos获取实例，让前端与你联调时，指定verion，这样可以保证请求只会打到你个人的机器上
```

#### 工作原理

1. 在`nacos`上注册服务的ip/port为 `192.168.0.15:19551` 
2. `gateway`需要转发时，问`nacos`拿到的ip/port为`192.168.0.15:19551`
3. 由于已经架设了`proxy`于是向`192.168.0.15:19551`地址发送的请求，都被proxy转发到了`111.111.111.111:33080`地址
4. `111.111.111.111`上的`proxy bridge`会看有哪些的`client`
5. `111.111.111.111`将内网的请求转发到`client`上
6. `client`收到请求后，由于设置了`":19551@:9551"`，于是client把请求转发到本地`9551`接口上

至此功能完成