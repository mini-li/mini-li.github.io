---
title: Overlay
layout: default
parent: Network
has_children: false
---

常见的overlay方案包括underlay路由、vxlan、ipip等
<!--more-->

## docker容器添加额外的网络

- docker 容器配置一个underlay的物理ip

```shell
# 获取所有的命名空间
lsns
# 获取网络命名空间`lsns --help`查看具体的每一列代表什么
lsns --type net

# 查看docker的网络隔离命名空间
ls -alh /run/docker/netns

docker run -d  --name nginx nginx
# 查看网络命名空间
docker inspect nginx
# 查看当前的ip直接访问测试是否ok 172.17.0.3

# 将某个进程所在的命令空间挂载到指定名字
ip netns attach <name> <pid>
# 查看某个进程所在命名空间
ip netns identify <pid>
# 也可以将具体的命名空间连接到/var/run/netns
# 这里将docker创建的nginx容器具体的网络命名空间aa4fc1e4df42，软连接到ip识别的网络命名空间下
ln -s /run/docker/netns/aa4fc1e4df42 /var/run/netns/busybox     

# 这里获取nginx的pid以后直接使用attach的方式
ps -ef|grep nginx
ip netns attacch nginx 3901662

#  增加一个macvlan设备并配置给nginx隔离空间
ip link add macvlan1 link enp0s31f6 type macvlan mode bridge
ip link set dev macvlan1 netns nginx
ip netns exec nginx ip link
# 配置ip并测试网络是否ok
ip netns exec nginx ip link set macvlan1 up
ip netns exec nginx ip route
# 随便选一个空闲的ip分配给macvlan1
ping 172.16.20.120
ip netns exec nginx ip addr add 172.16.20.120/23 dev macvlan1
ip netns exec nginx ip route
# 测试到网关是否ok 
ip netns exec nginx ping 172.16.21.254
# 测试访问nginx服务是否ok,页面访问或者直接curl
curl 172.16.20.120
```

## 通过underlay路由实现overlay

- 通过给两台主机的隔离网络添加路由实现overlay互通
  - node1 物理网卡ip: 192.168.3.2 mac: 52:54:00:09:ef:b4 
  - node2 物理网卡ip: 192.168.3.3 mac:52:54:00:b9:d4:ba
  - node1 overlay :10.0.2.0/24  
  - node2 ovlary :10.0.3.0/24

