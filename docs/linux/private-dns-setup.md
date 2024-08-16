---
title: Private DNS setup
layout: default
parent: Linux
has_children: false
tags: linux dns bind9
---


## 安装

通过包管理安装  

```shell
sudo apt-get install bind9
```

## 配置

- 域名配置`/etc/bind/named.conf.local`

```conf
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";
// 这里指定需要配置的域名
zone "test.seasw.com" {
        type master;
        file "/var/cache/bind/test.seasw.com.zone";
};
zone "0.168.192.in-addr.arpa" {
   type master;
   file "/var/cache/bind/rev.0.168.192.in-addr.arpa";
};
```

- 正向域名解析文件`/var/cache/bind/test.seasw.com.zone`

```conf
$TTL 300
test.seasw.com. IN SOA localhost. root.localhost. (
 1 ; serial
 1H ; refresh
 5M ; retry
 1W ; expire
 1M ) ; minimum
test.seasw.com. IN NS localhost.
serverb.test.seasw.com. IN A 192.168.0.11
servera.test.seasw.com. IN A 192.168.0.10

```

- 逆向域名解析`/var/cache/bind/rev.0.168.192.in-addr.arpa`

```conf
$TTL 300
0.168.192.in-addr.arpa. IN SOA localhost. root.localhost. (
 1 ; serial
 1H ; refresh
 5M ; retry
 1W ; expire
 1M ) ; minimum

0.168.192.in-addr.arpa. IN  NS localhost.
11 IN PTR serverb.test.seasw.com.
10 IN PTR servera.test.seasw.com.
```

## 配置说明

- SOA 开始一个区，基本格式：  
  zone      IN      SOA   Hostname  Contact (  
                          SerialNumber  
                          Refresh  
                          Retry  
                          Expire  
                          Minimum )  
  Hostname：存放本 Zone 的域名服务器的主机名  
  - Contact：管理域的管理员的邮件地址  
  - SerialNumber：本区配置数据的序列号，用于从服务器判断何时获取最新的区数据  
  - Refresh：辅助域名服务器多长时间更新数据库  
  - Retry：若辅助域名服务器更新数据失败，多长时间再试  
  - Expire：若辅助域名服务器无法从主服务器上更新数据，原有的数据何时失效  
  - Minimum：设置被缓存的否定回答的存活时间  

- RR（资源记录）包含4个元素（Name， Value， Type,  TTL）  
  - Type A: 则记录的是一条ipv4的域名到地址的映射,比如：baidu.com 123.123.123.123 A  
  - Type NS:  name则是一个域，value 则是这个域名所对应的服务器的地址，比如：google.com dns.google.com NS ； 在配合一条Type A 则能达到 dns.google.com 8.8.8.8 A 
  - Type CNAME:  一个域名的别名  
  - Type MX:邮件服务地址的别名 比如： foo.com mail.foo.com MX  

## dns 报文格式

![dns](/assets/images/linux/dns.png)  

## 动态域名

- 配置key test.seasw.com-key 这个key可以通过tsig-keygen test.seasw.com-key 生成  

```conf
// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";
key "test.seasw.com-key" {
        algorithm hmac-sha256;
        secret "8LI/dBaqmwg/HxuxBJkrkqRQrFAVLBRZKa3a7zHqq3U=";
};
zone "test.seasw.com" {
        type master;
        file "/var/cache/bind/test.seasw.com.zone";
        allow-update { key test.seasw.com-key;};
};
zone "0.168.192.in-addr.arpa" {
   type master;
   file "/var/cache/bind/rev.0.168.192.in-addr.arpa";
};
```

## 域名更新和删除

```shell
# 使用nsupdate 命令动态的更新或者删除域名
nsupdate -k test.seasw.com.key
> server 127.0.0.1
> update delete servera.test.seasw.com A
> update add servera.test.seasw.com 80000 IN A 192.168.0.2
> send
> quit
```
