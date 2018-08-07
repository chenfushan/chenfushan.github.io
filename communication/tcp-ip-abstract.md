# TCP/IP概述类内容

## TCP/IP协议族

**四层模型**

链路层 -> 网络层 -> 运输层 -> 应用层

**七层模型**

物理层 -> 数据链路层 -> 网络层 -> 传输层 -> 会话层 -> 表示层 -> 应用层

这里大家都喜欢问七层模型, 其实四层分割也足够了.

> 物理层 + 数据链路层 = 链路层 : 建立、维护、断开物理连接以及进行硬件地址寻址、差错校验等.
> 网络层 : 逻辑地址寻址，实现不同网络之间的路径选择. ICMP IGMP IP（IPV4 IPV6） ARP RARP
> 传输层 : 定义传输数据的协议端口号，以及流控和差错校验. TCP UDP.
> 会话层 + 表示层 + 应用层 = 应用层 : 建立、管理、终止会话. 数据的表示、安全、压缩. 网络服务与最终用户的一个接口. HTTP FTP TFTP SMTP SNMP DNS TELNET HTTPS POP3 DHCP这些都算应用层. 还有JPEG这种数据压缩方式等.

## 互联网地址

互联网地址就是IP地址, 32bit长度. 单纯的0/1表示, 不要使用十进制数字. 一共5类, 依靠前置位1来确定.

A类: `0 | 网络号 7 位 | 主机号 24 位 |`
B类: `1 | 0 | 网络号 14 位 | 主机号 16位 |`
C类: `1 | 1 | 0 | 网络号 21 位 | 主机号 8位 |`
D类: `1 | 1 | 1 | 0 | 多播组号 28 位 |`
E类: `1 | 1 | 1 | 1 | 0 | 27 位备用 |`

| 类型 | 范围 |
|:--:|:---:|
| A | 0.0.0.0 ~ 127.255.255.255 |
| B | 128.0.0.0 ~ 191.255.255.255 |
| C | 192.0.0.0 ~ 223.255.255.255 |
| D | 224.0.0.0 ~ 239.255.255.255 |
| E | 240.0.0.0 ~ 255.255.255.255 |

##域名系统

DNS作为很重要的一环, 为互联网的普及立下了汗马功劳啊. 毕竟我们习惯了记住: `www.baidu.com` 而不是: `180.97.33.107` 这个IP. DNS是一个分布式数据库, 各个地方都有存在, 大约半个小时就可以同步完一次更新吧(模糊的). DNS污染和DNS攻击都很有名.

##数据帧封装

应用程序的数据想要传输, 就要先对数据送入协议栈, 然后逐层的当做比特数据流送入网络. 每层的下沉都要增加首部信息. 

```
| <5>以太网首部(14字节) | <4>IP首部(20字节) | <4>TCP首部(20字节)/UDP首部(8字节) | 应用数据(<1>用户数据 + <2>Appl首部) | <5>以太网尾部(4字节) | 
```

以上是一个数据进入协议栈的封装好的数据帧. 以太网数据帧物理特性要求长度必须在46 ~ 1500字节 也就是上面的去掉以太网首部和以太网尾部的其余内容加起来的长度.

上面内容中的`<1~5>`表示的封装的顺序, 由上到下的封装过程.

##分用

当目的主机收到一个以太网数据帧, 数据流向就是从下到上了. 从以太网驱动程序开始不断向上, 根据首部的内容来分用给各个协议栈处理.

```

-> 以太网驱动程序 -以太网首部-> ARP / IP / RARP选择对应协议处理 -解析IP首部-(接下一行)

-> ICMP / IGMP / TCP / UDP 选择对应处理 -TCP首部->应用程序数据

```

通过之前封装的首部来解析对应需要处理的方案. 根据端口号等选择对应的应用程序处理.

## 客户-服务模型

客户-服务模型, 可以包含我们时长追求的c/s, b/s架构的争论. 这不重要, 都是客户-服务模型的子集. 这种模型有两类:

1. 重复型.
2. 并发型.

**重复型**

- 等待一个客户请求的盗来.
- 处理客户请求.
- 发送响应.
- 返回第一步

**并发型**

- 等待客户请求盗来
- 启动新服务器处理客户请求, 处理结束后终止新服务器.
- 返回第一步

## 端口

TCP/UDP的端口号都是一个16bit的数字, 用于识别应用程序. 知名的端口号( 1 ~ 1023 )由Internet号分配机构(Internet Assigned Numbers Authority)来管理.

客户端不关心自己的端口号, 因为都是临时的. 一般处于 ( 1024 ~ 5000 ). 大于5000的是为其他服务器预留的. 但是现在新的流行的变多, 也很多流行的服务器软件运行在比较大的端口号. 例如: `memcache` 11211端口, `redis` 6379端口.

