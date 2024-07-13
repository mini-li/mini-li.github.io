---
title: "Linux virtual network"
layout: default
parent: Network
has_children: false
---

Linux 下常见的虚拟网络介绍以及Linux的路由和路由表介绍
<!--more-->

# 常见的虚拟网络设备

## veth

veth pair是一对虚拟网络设备，一端发送的数据会由另外一端接受，常用于不同的网络命名空间
虚拟网卡对的流量会从一端传到另一端,[veth说明](https://man7.org/linux/man-pages/man4/veth.4.html).

使用ip命令可以轻松的创建一个虚拟网卡对,比如:
```shell
ip link add veth0 type veth peer name veth00
```

## tap/tun

TAP/TUN设备是一种让用户态程序向内核协议栈注入数据的设备，TAP等同于一个以太网设备，工作在二层；而TUN则是一个虚拟点对点设备，工作在三层。

## macvlan/ipvlan

通过ip或者mac来过滤流量
常用于添加underlay网络

### macvlan四种模式

#### Vepa 模式

- vepa模式下同一个eth下的macvlan间的流量需要走到交换机再下来，需要交换机支持hairpin_mode
![macvlan-vepa](/assets/images/network/macvlan-vepa.png)

- 模式验证

```shell
# 创建net namespace
ip netns add ns1
ip netns add ns2
# 创建macvlan
ip link add macvlan1 link enp0s31f6 type macvlan mode vepa
ip link add macvlan2 link enp0s31f6 type macvlan mode vepa
# 将macvlan放入指定的命名空间
ip link set dev macvlan1 netns ns1
ip link set dev macvlan2 netns ns2
# 启动macvlan1并配置ip
ip netns exec ns1 ip link set dev macvlan1 up
ip netns exec ns1 ip link
ip netns exec ns1 ip addr add 172.16.20.98/23 dev macvlan1
# 测试到网关是否通
ip netns exec ns1 ping 172.16.21.254

# 启动macvlan2并配置ip
ip netns exec ns2 ip link set dev macvlan2 up
ip netns exec ns2 ip link
ip netns exec ns2 ip addr add 172.16.20.99/23 dev macvlan2
# 测试到网关都是通的
ip netns exec ns2 ping 172.16.21.254

# 测试相互之间是不通的
ip netns exec ns2 ping 172.16.20.98

# 发现当前到网关都是通的但是相互之间是不同的说明交换机没有开启hairpin_mode
# 使用bridge模拟交换机模拟开启hairpin_mode和不开的情况
# 开启hairpin_mode方法echo 1 > /sys/class/net/br0/brif/veth0/hairpin_mode
```

#### bridge 模式

- 这个模式和linx的网桥很类似
![macvlan-bridge](/assets/images/network/macvlan-bridge.png)

- 模式验证

```shell
# 这里和上面的操作一模一样试试创建macvlan的时候指定mode bridge
ip link add macvlan1 link enp0s31f6 type macvlan mode bridge
ip link add macvlan2 link enp0s31f6 type macvlan mode bridge

# 测试在这种模式下相互直接是可以之际通信的
# 可以直接在enp0s31f6上抓包看到有相互的流量
```

#### private 模式

- macvlan之间是强隔离的，相互之间都不能通信

![macvlan-private](/assets/images/network/macvlan-private.png)

#### Passthrough 模式

- 将物理网卡直接分配给虚拟机，虚拟机可以直接使用物理卡

![macvlan-passthrough](/assets/images/network/macvlan-passthrough.png)

### ipvlan

- ipvla创建的设备mac都是相同的
- ipvlan和macvlan在很多情况下是一样的，但是有时更好用ipvlan比如：物理口的交换机绑定了指定mac的情况下

![ipvlan](/assets/images/network/ipvlan.png)

#### ipvlan 二层模式测试

- 二层模式本质上和macvlan没有区别

```shell
# 创建网络命令空间
ip netns add ns1
ip netns add ns2
# 创建ipvlan设备
ip link add name ipvl1 link enp0s31f6 type ipvlan mode l2
ip link add name ipvl2 link enp0s31f6 type ipvlan mode l2
# 将设备放入命名空间
ip link set dev ipvl1 netns ns1
ip link set dev ipvl2 netns ns2
# 给设备配置ip
ip netns  exec ns1 ip link
ip netns  exec ns1 ip link set dev ipvl1 up
ip netns  exec ns2 ip link
ip netns  exec ns2 ip link set dev ipvl2 up
ip netns  exec ns1 ip addr add 172.16.20.98/23 dev ipvl1
ip netns  exec ns2 ip addr add 172.16.20.99/23 dev ipvl2

# 测试网络
ip netns  exec ns2 ping 172.16.21.254
ip netns  exec ns1 ping 172.16.21.254

```

#### ipvlan 三层模式测试

- 三层模式和二层模式几乎相同，改变一下mod就行区别只是在三层模式下物理网卡表现为路由器二层模式下表现为交换机

**Note:** 在L3模式中抓到ARP，二层广播和组播都不处理，并且这里是不可以直接同underlay网络的，需要在路由器上配置路由才行.
{: .notice--warning}

- 直接通过不同的三层测试:

```shell
# 通过创建三层网络查看mac地址
# 可以看到这里创建的设备的mac和物理卡的是一样的
ip link add name ipvl1 link enp1s0 type ipvlan mode l3
ip link add name ipvl2 link enp1s0 type ipvlan mode l3
# 放入不同的空间
ip netns add ns1
ip netns add ns2
ip link set dev ipvl1 netns ns1
ip link set dev ipvl2 netns ns2
ip netns exec ns1 ip link set dev ipvl1 up
ip netns exec ns2 ip link set dev ipvl2 up
# 配置网络
ip netns exec ns1 ip addr add 192.168.11.2/24 dev  ipvl1
ip netns exec ns1 ip addr add 192.168.22.2/24 dev  ipvl2
# 配置路由
ip netns exec ns1 ip route add default dev ipvl1
ip netns exec ns2 ip route add default dev ipvl2
# 测试网络
ip netns exec ns2 ping 192.168.11.2
```

## bridge

虚拟网桥，实现二层转发
可以通过brctl 命令来创建和管理，可以通过ip命令

### vlan 实验验证二层

- 常用命令：
- 查看网桥link信息： `bridge link show`
- 查看vlan信息:`bridge vlan show`
- 查看对应网桥的mac地址转发表:`bridge fdb show br br0`
![bridge-vlan](/assets/images/network/bridge-vlan.png)

**ProTip:** \
*pvid*：端口的默认vlan，所有从该端口输入的没有携带vlan的报文，会被打上该vlan标签，该选项只对输入报文有效。\
*untagged*：端口的untag vlan，输出报文携带该vlan时，会被剥离。
{: .notice--info}

- 实验操作：

```shell
# 启动内核vlan
sudo modprobe 8021q 
sudo ip link add br0 type bridge vlan_filtering 1 
# 创建虚拟网卡对
ip link add veth1 type veth peer name veth11
ip link add veth2 type veth peer name veth22
ip link add veth3 type veth peer name veth33
# 将虚拟网卡挂到网桥br0
ip link set veth1 master br0 
ip link set veth2 master br0 
ip link set veth3 master br0 
# 配置vlan
bridge vlan add dev veth1 vid 11 pvid untagged 
bridge vlan add dev veth3 vid 12 pvid untagged 
bridge vlan add dev veth2 vid 11 
bridge vlan add dev veth2 vid 12 
# 查看vlan配置
bridge vlan show

# veth1             1 Egress Untagged
#                  11 PVID Egress Untagged
#veth2             1 PVID Egress Untagged
#                  11
#                  12
#veth3             1 Egress Untagged
#                  12 PVID Egress Untagged
#

# 启动设备
ip link set up dev br0 
ip link set up dev veth1 
ip link set up dev veth2 
ip link set up dev veth3
# 配置ip
ip netns exec ns1 ip addr add 10.0.0.11/24 dev veth11
ip netns exec ns2 ip addr add 10.0.0.22/24 dev veth22
ip netns exec ns3 ip addr add 10.0.0.33/24 dev veth33

# 测试netns2到netns1和netns3,这个时候都是不通的，因为
# 这个时候veth2 进入交换机的包默认tag是1，到veth1以后辉Untag 1 发给veth11，然后veth11回包发个veth1 默认tag是11
# 这个时候 veth2 是没法接受tag11的，因为还没有创建tag 11 子接口，也没有untag
ip netns exec ns2 ping 10.0.0.11
ip netns exec ns2 ping 10.0.0.33

# 创建vlan子接口
ip netns exec ns2 ip link add link veth22 name veth22.11 type vlan id 11
ip netns exec ns2 ip link add link veth22 name veth22.12 type vlan id 12
ip netns exec ns2 ip link set dev veth22.11 up
ip netns exec ns2 ip link set dev veth22.12 up
ip netns exec ns2 ip addr
ip netns exec ns2 ip addr add 10.0.0.211/24 dev veth22.11
ip netns exec ns2 ip addr add 10.0.0.212/24 dev veth22.22
# 这个时候再来测试就是ok的
ip netns exec ns2 ping 10.0.0.11 -I veth22.11
ip netns exec ns2 ping 10.0.0.12 -I veth22.11
ip netns  exec ns3 ip addr
ip netns exec ns2 ping 10.0.0.33 -I veth22.11
ip netns exec ns2 ping 10.0.0.33 -I veth22.22
```

## vrf

VRF全称是虚拟路由转发（Virtual Routing Forwarding）
虚拟隔离路由，实现三层路由

### 两个vrf联通实验

1. 使用veth联通两个vrf
    创建两个vrf，vrf0 和vrf1 `ip link add vrf0 type vrf table 10`
2. 给两个vrf配置网络设备`ip tuntap add dev tap0 mode tap`
3. 给tap配置ip，保证不是在同一个网段172.18.1.2/24和172.18.2.2/24
4. 创建veth: `ip link add veth0 type veth peer name veth00`
5. 给两个vrf配置对端的路由
6. 测试连通性

```shell
# 创建vrf0
ip link add vrf0 type vrf table 10
ip link set vrf0 up
ip tuntap add dev tap0 mode tap
ip link set tap0 master vrf0
ip addr add 172.18.1.2/24 dev tap0
ip link set tap0 up
ip addr show vrf vrf0

# 创建vrf1，与vrf0 镜像的配置
ip link add vrf1 type vrf table 11
ip link set vrf1 up
ip tuntap add dev tap1 mode tap
ip link set tap1 master vrf1
ip addr add 172.18.2.2/24 dev tap1
ip link set tap1 up
ip addr show vrf vrf1
ip route show vrf vrf1

# 创建veth来连通两个vrf
ip link add veth0 type veth peer name veth00
ip link show type veth
ip link set veth0 master vrf0 
ip link set veth00 master vrf1
ip link set veth0 up
ip link set veth00 up
# 配置对端路由
ip route add vrf vrf0 172.18.2.0/24 dev veth0
ip route add vrf vrf1 172.18.1.0/24 dev veth00
ip route show vrf vrf0
ip route show vrf vrf1
# 测试连通性
ping 172.18.2.2 -I vrf0
```

## 路由

## 路由规则

1. 在发送ip包之前会把发送到目的ip与本地路由表进行比较来判断从哪个网卡出去
2. 查看本地的路由规则`ip rule`
3. 添加路由表`ip rule add table 110` 查看指定路由表 `ip route list table 100`
4. 向路由表添加路由 `ip route add 172.16.20.0/23 via 172.16.10.60 table 100`获取目标地址的路由`ip route get 172.16.20.254`

**ProTip:** 这里第一列数字代表优先级，数字越小优先级越高
最后的一列代表需要查找的路由表可以是数字或者名字，可以通过ip route show table local查看具体的路由信息from all 匹配所有的入口流量
{: .notice--info}

>0:        from all lookup local \
>32766:        from all lookup main \
>32767:        from all lookup default

## 查看路由表

- 查看网络路由`ip route show table main` 或者`netstat -rn`或者`route -n`

> Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface \
> 0.0.0.0         192.168.2.1     0.0.0.0         UG        0 0          0 enp0s9 \
> 172.17.0.0      0.0.0.0         255.255.0.0     U         0 0          0 docker0

## 字段解释

- Destination ：要去的目标网络
- Gateway： 网关
- Genmask ： 掩码
- Flags：路由标记，
  - U： 表示运行
  - G：表示网关，
  - H：表示主机，
  - R 恢复动态路由产生的表项
  - D: 该路由是由重定向报文创建的
  - M:  该路由被ICMP重定向报文修改过
  - A：表示该路由是任播地址，多个目标具有相同的IP地址。
  - ！表示路由已经禁止
- scope link/global/host:  
- link 表示二层
- global是全局生效
- host表示是本地路由
- src x.x.x.x: 源ip

## iptables

在另一文档中有做详细的[iptables介绍](/pages/linux/iptables/)
