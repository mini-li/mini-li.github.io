---
title: Iptables
layout: default
parent: Linux
has_children: false
---

## iptables 五链

- iptables 可以检测、修改、转发、重定向和丢弃 IPv4 数据包。


{: .highlight }
**数据流量在iptables的5链中的流转流程**
![五链四表](/assets/images/linux/iptable-5-chains.png)

1. 当主机/网络服务器网卡收到一个数据包之后进入内核空间的网络协议栈进行层层解封装
2. 刚刚进入网络层的数据包通过 PRE_ROUTING 关卡时，要进行一次路由选择，当目标地址为本机地址时，数据进入 INPUT，非本地的目标地址进入 FORWARD（需要本机内核支持 IP_FORWARD），所以目标地址转换通常在这个关卡进行
3. INPUT 经过路由之后送往本地的数据包经过此关卡，所以过滤 INPUT 包在此点关卡进行
4. FORWARD 经过路由选择之后要转发的数据包经过此关卡，所以网络防火墙通常在此关卡配置
5. OUTPUT 由本地用户空间应用进程产生的数据包过此关卡，所以 OUTPUT 包过滤在此关卡进行
6. POST_ROUTING 刚刚通过 FORWARD 和 OUTPUT 关卡的数据包要通过一次路由选择由哪个接口送往网络中，经过路由之后的数据包要通过 POST_ROUTING 此关卡，源地址转换通常在此点进行

## iptables 四表

{: .note }
并不是所有的“链”都具有所有类型的“规则”，也就是说，某个特定表中的“规则”注定不能应用到某些“链”中，比如，用作地址转换功能的 nat 表里面的“规则”据不能存在于 FORWARD “链”中,
如果在多张表中有同一个链则执行的先后顺序为`raw -> mangle -> nat -> filter`


![iptables 4table](/assets/images/linux/iptables-4-table.png)

1. filter 表：负责过滤功能；与之对应的内核模块是 iptables_filter
2. nat(Network Address Translation) 表：网络地址转换功能，典型的比如 SNAT、DNAT，与之对应的内核模块是 iptables_nat
3. mangle 表：解包报文、修改并封包，与之对应的内核模块是 iptables_mangle
4. raw 表：关闭 nat 表上启用的连接追踪机制；与之对应的内核模块是 iptables_raw


## 命令规则

iptables 区分ipv4和ipv6,v4主要用命令`iptables`,v6用命令`ip6tables`

iptables 参数：  

```shell
 -p：指定要匹配的数据包协议类型
 -s, --source [!] address[/mask] ：把指定的一个／一组地址作为源地址，按此规则进行过滤。当后面没有 mask 时，address 是一个地址，比如：192.168.1.1；当 mask 指定时，可以表示一组范围内的地址，比如：192.168.1.0/255.255.255.0
 -d, --destination [!] address[/mask] ：地址格式同上，但这里是指定地址为目的地址，按此进行过滤
 -i, --in-interface [!] <网络接口name> ：指定数据包的来自来自网络接口，比如最常见的 eth0 。注意：它只对 INPUT，FORWARD，PREROUTING 这三个链起作用。如果没有指定此选项， 说明可以来自任何一个网络接口。同前面类似，"!" 表示取反。
 -o, --out-interface [!] <网络接口name> ：指定数据包出去的网络接口。只对 OUTPUT，FORWARD，POSTROUTING 三个链起作用

### 查看管理命令

 -L, --list [chain] 列出链 chain 上面的所有规则，如果没有指定链，列出表上所有链的所有规则。

### 规则管理命令

 -A, --append chain rule-specification 在指定链 chain 的末尾插入指定的规则，也就是说，这条规则会被放到最后，最后才会被执行。规则是由后面的匹配来指定
 -I, --insert chain [rulenum] rule-specification 在链 chain 中的指定位置插入一条或多条规则。如果指定的规则号是1，则在链的头部插入。这也是默认的情况，如果没有指定规则号
 -D, --delete chain rule-specification -D, --delete chain rulenum 在指定的链 chain 中删除一个或多个指定规则
 -R num Replays替换/修改第几条规则

### 链管理命令（这都是立即生效的）

 -P, --policy chain target ：为指定的链 chain 设置策略 target。注意，只有内置的链才允许有策略，用户自定义的是不允许的
 -F, --flush [chain] 清空指定链 chain 上面的所有规则。如果没有指定链，清空该表上所有链的所有规则
 -N, --new-chain chain 用指定的名字创建一个新的链
 -X, --delete-chain [chain] ：删除指定的链，这个链必须没有被其它任何规则引用，而且这条上必须没有任何规则。如果没有指定链名，则会删除该表中所有非内置的链
 -E, --rename-chain old-chain new-chain ：用指定的新名字去重命名指定的链。这并不会对链内部照成任何影响
 -Z, --zero [chain] ：把指定链，或者表中的所有链上的所有计数器清零
 -j, --jump target <指定目标> ：即满足某条件时该执行什么样的动作。target 可以是内置的目标，比如 ACCEPT，也可以是用户自定义的链
 -h：显示帮助信息
```

***
命令demo

```shell
# 查看对应表的所有链
iptables -t filter -L
# 查看指定的链 加上-v 可以获取详细信息
iptables -vL FORWARD
# 加上-n显示ip 不然iptables默认会显示域名 比如anywhere
iptables -vnL FORWARD
# 加上–-line-number 可以显示对应的规则编号可以用去操作删除或者是替换
iptables -vnL FORWARD –-line-number

#-------------------------------------------------------------------------------
# 个字段解释：
# - pkts：对应规则匹配到的报文的个数
# - bytes：对应匹配到的报文包的大小总和
# - target：规则对应的target，往往表示规则对应的动作，即规则匹配成功后需要采取的措施
# - prot：表示规则对应的协议，是否只针对某些协议应用此规则
# - opt：表示规则对应的选项
# - in：表示数据包由哪个接口(网卡)流入，我们可以设置通过哪块网卡流入的报文需要匹配当前规则
# - out：表示数据包由哪个接口(网卡)流出，我们可以设置通过哪块网卡流出的报文需要匹配当前规则
# - source：表示规则对应的源头地址，可以是一个IP，也可以是一个网段
# - destination：表示规则对应的目标地址。可以是一个IP，也可以是一个网段
```

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
