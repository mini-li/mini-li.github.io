---
title: ipvs
layout: default
parent: Linux
has_children: false
---



## 1. 基本信息


- IPVS 支持比 IPTABLES 更复杂的负载平衡算法（最小负载、最少连接、局部性、加权等）。
- NAT、IP tunneling和Direct Routing。


- kube-proxy中的iptables：

> 为了避免增加内核和用户空间的数据拷贝操作，提高转发效率，Kube-proxy提供了iptables模式。
> 在该模式下，Kube-proxy为service后端的每个Pod创建对应的iptables规则，直接将发向
> Cluster IP的请求重定向到一个Pod IP。
> 
> 该模式下Kube-proxy不承担四层代理的角色，只负责创建iptables规则。该模式的优点是
> 较userspace模式效率更高，但不能提供灵活的LB策略，当后端Pod不可用时也无法进行重试。

- kube-proxy中的ipvs
> kube-proxy监控Pod的变化并创建相应的ipvs rules。ipvs也是在kernel模式下通过
> netfilter实现的，但采用了hash table来存储规则，因此在规则较多的情况下，Ipvs
> 相对iptables转发效率更高。除此以外，ipvs支持更多的LB算法。
> 
> 从k8s的1.8版本开始，kube-proxy引入了IPVS模式，IPVS模式与iptables同样基于
> Netfilter，但是采用的hash表，因此当service数量达到一定规模时，hash查表的
> 速度优势就会显现出来，从而提高service的服务性能。
> 
> 如果要设置kube-proxy为ipvs模式，必须在操作系统中安装IPVS内核模块。


## 2. 三种工作模式
### 2.1 NAT模式

客户端发送的请求到达Director后，Director根据负载均衡算法改写目标地址为后端某个RIP(web服务器池中主机之一)并转发给该后端主机，就像NAT一样。
当后端主机处理完请求后，后端主机将响应数据交给Director，并由director改写源地址为VIP后传输给客户端。
大多数商品化的IP负载均衡硬件都是使用此方法，如Cisco的LocalDirector、F5的Big/IP。

### 2.2 TUN模式

调度器把请求报文通过IP隧道转发至真实服务器，而真实服务器将响应直接返回给客户，所以调度器只处理请求报文。由于一般网络服务响应报文比请求报文大许多，采用VS/TUN技术后，调度器得到极大的解放，集群系统的最大吞吐量可以提高10倍。

### 2.3 DR模式

VS/TUN模式下，调度器对数据包的处理是使用IP隧道技术进行二次封装。VS/DR模式和VS/TUN模式很类似，只不过调度器对数据包的处理是改写数据帧的目标MAC地址，通过链路层来负载均衡。

VS/DR通过改写请求报文的目标MAC地址，将请求发送到真实服务器，而真实服务器将响应直接返回给客户。同VS/TUN技术一样，VS/DR技术可极大地提高集群系统的伸缩性。这种方法没有IP隧道的开销，对集群中的真实服务器也没有必须支持IP隧道协议的要求，但是要求调度器与真实服务器都有一块网卡连在同一物理网段上，以便使用MAC地址通信转发数据包。