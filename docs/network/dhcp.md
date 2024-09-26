---
title: "dhcp"
layout: default
parent: Network
has_children: false
---

## 基本信息

- dhcp使用udp协议，服务端端口号68，客户端端口号67

- **DHCP Snooping信任功能**，保证dhcp服务器的正确性
    - 信任接口正常接收DHCP服务器响应的DHCP ACK、DHCP NAK和DHCP Offer报文。
    - 非信任接口在接收到DHCP服务器响应的DHCP ACK、DHCP NAK和DHCP Offer报文后，丢弃该报文。

- **DHCP Snooping绑定表** 
    - 连接在二层接入设备上的PC配置为自动获取IP地址，DHCP服务器将含有IP地址信息的DHCP ACK报文通过单播的方式发送给PC，
    - 二层接入设备收到DHCP ACK报文后，会从该报文中提取关键信息（包括PC的MAC地址以及获取到的IP地址、地址租期），并获取与PC连接的使能了DHCP Snooping功能的接口信息（包括接口编号及该接口所属的VLAN），根据这些信息生成DHCP Snooping绑定表
    - DHCP Snooping绑定表根据DHCP租期进行老化或根据用户释放IP地址时发出的DHCP Release报文自动删除对应表项。
    - 由于DHCP Snooping绑定表记录了DHCP客户端IP地址与MAC地址等参数的对应关系，故通过对报文与DHCP Snooping绑定表进行匹配检查，能够有效防范非法用户的攻击
    - 为了保证设备在生成DHCP Snooping绑定表时能够获取到用户MAC等参数，DHCP Snooping功能需应用于二层网络中的接入设备或第一个DHCP Relay上
![dhcp-binding-table](/assets/images/network/dhcp-snooping-binding-table.png)


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
