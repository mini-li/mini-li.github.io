---
title: "dhcp"
layout: default
parent: Network
has_children: false
---

## 基本信息

- dhcp使用udp协议，服务端端口号68，客户端端口号67

## 交互流程

- 客户端执行DHCP-DISCOVER后，如果没有DHCP服务器响应客户端的请求，客户端会随机使用169.254.0.0/16网段中的一个IP地址配置到本机地址。
- 采用先到先得，如果有多个dhcp服务器的话采用第一个收到的

- 广播 UDP 目标端口号为67    源IP 地址0.0.0.0    目的IP:255.255.255.255

- 在client端可能会接受到多个offer包，通常clientdaunt只会接受收到的第一个DHCP offer报文，然后client端就会以广播的方式发送一个DHCP request报文请求分配IP地址。

- server端在收到DHCP request报文之后，会判断”option”字段的serverIP地址是否是自己的IP地址，如果符合分配IP地址的条件，就会给client发送一个DHCP ACK包，如果不满足就发挥发送一个DHCP NAK 包。
- server在收到REQUEST报文后，根据REQUEST报文中携带的client MAC来查找有没有相应的租约记录，如果有则发送ACK报文作为回应，通知client可以使用分配的IP地址。

![dhcp](/assets/images/network/dhcp-流程.png)

## 配置操作

- dhcp全局配置的时候全局的gateway-list 对应的地址应该和选择的接口的ip地址一样。

![dhcp-config](/assets/images/network/dhcp-config.png)

- 接口配置模式

![dhcp-interface-config](/assets/images/network/dhcp-interface-config.png)
