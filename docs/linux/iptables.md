---
title: Iptables
layout: default
parent: Linux
has_children: false
---

# iptables

iptables 可以检测、修改、转发、重定向和丢弃 IPv4 数据包。
iptables 区分ipv4和ipv6,v4主要用命令`iptables`,v6用命令`ip6tables`

## 参数说明

- -P	设置默认策略:`iptables -P INPUT DROP`
- -F	清空规则链
- -L	查看规则链
- -A	在规则链的末尾加入新规则
- -I	num 在规则链的头部加入新规则
- -D	num 删除某一条规则
- -s	匹配来源地址 IP/MASK，加叹号"!"表示除这个 IP 外。
- -d	匹配目标地址
- -i	网卡名称 匹配从这块网卡流入的数据
- -o	网卡名称 匹配从这块网卡流出的数据
- -p	匹配协议,如 tcp,udp,icmp
- --dport num	匹配目标端口号
- --sport num	匹配来源端口号



```shell
# 查看对应表的所有链
iptables -t filter -L
# 查看指定的链 加上-v 可以获取详细信息
iptables -vL FORWARD
# 加上-n显示ip 不然iptables默认会显示域名 比如anywhere
iptables -vnL FORWARD
# 加上–-line-number 可以显示对应的规则编号可以用去操作删除或者是替换
iptables -vnL FORWARD –-line-number
```

## 各个字段的说明

- pkts：对应规则匹配到的报文的个数
- bytes：对应匹配到的报文包的大小总和
- target：规则对应的target，往往表示规则对应的动作，即规则匹配成功后需要采取的措施
- prot：表示规则对应的协议，是否只针对某些协议应用此规则
- opt：表示规则对应的选项
- in：表示数据包由哪个接口(网卡)流入，我们可以设置通过哪块网卡流入的报文需要匹配当前规则
- out：表示数据包由哪个接口(网卡)流出，我们可以设置通过哪块网卡流出的报文需要匹配当前规则
- source：表示规则对应的源头地址，可以是一个IP，也可以是一个网段
- destination：表示规则对应的目标地址。可以是一个IP，也可以是一个网段


## policy

![iptables-policy](/assets/images/linux/iptables.jpg)

policy ACCEPT ，0 packets，0bytes 三部分

- policy表示当前链的默认策略
    - policy ACCEPT表示上图中FORWARD的链的默认动作为ACCEPT，说白了就是**黑名单**机制，默认所有人都能通过，只有指定的人不能通过.
    - 如果需要配置**白名单**模式的话修改默认策略为drop则执行命令：`iptables -P FORWARD DROP`
- packets 表示当前链（上例为FORWARD链）默认策略匹配到的包的数量，0 packets表示默认策略匹配到0个包
- bytes 表示当前链默认策略匹配到的所有包的字节总和

## iptables常用场景

让内让的ip走snat出去

### ip转发nat或者MASQUERADE

```shell
# 使用snat
iptables -t nat -A POSTROUTING -s 192.168.188.0/24 -j SNAT --to-source 210.14.67.127
# 使用MASQUERADE,这里指定网桥br0的网段是192.168.188.0/24 出去的包自动nat
iptables -t nat -A POSTROUTING -s 192.168.188.0/24 ! -o br0 -j MASQUERADE
```

### 端口映射

将到本机的ip转发到容器或者是别的

```shell
iptables -t nat -A PREROUTING -d 192.168.1.100 -p tcp --dport 2222  -j DNAT --to-dest 172.16.1.11:22
```

## 备份和恢复

备份

```shell
iptables-save > /etc/iptables.rule
```

恢复

```shell
iptables-restore < /etc/iptables.rule
```
