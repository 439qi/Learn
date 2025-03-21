---
title: Multicast Domain Name System
aliases:
  - mDNS
created: 2025-03-10
---
> [!cite]- References  
> - [RFC 6762: Multicast DNS](https://www.rfc-editor.org/rfc/rfc6762.html)  
> - [mDNS in the Enterprise | Microsoft Community Hub](https://techcommunity.microsoft.com/blog/networkingblog/mdns-in-the-enterprise/3275777)  
> - [mDNS: How does Multicast DNS work? - IONOS](https://www.ionos.com/digitalguide/server/know-how/multicast-dns/)  
> - [networking - How are names resolved on modern local networks? - Super User](https://superuser.com/questions/1359738/how-are-names-resolved-on-modern-local-networks)  
> - [RFC6762: mDNS — 新溪-gordon V2025.02 文档](https://knowledge.zhaoweiguo.com/build/html/x-learning/rfcs/dns/rfc6762)  
## mDNS 协议  

mDNS 协议用于==局域网==内，在无需 DNS 服务器的情况下使局域网内的主机实现相互发现和通信  
## 原理  

mDNS 协议实现基于 UDP 多播，使用保留的多播地址 `224.0.0.251`/`FF02::FB`，端口 5353  

当一个开启了 mDNS 服务的主机加入局域网时，会向局域网内的主机多播 IGMP报文，表明域名和 IP 地址，其余开启了 mDNS 的主机会响应域名和 IP 地址  
mDNS 的主机域名以 `.local` 结尾，以区别于普通 DNS 域名    

当执行 mDNS 查询时，向局域网内的主机多播 mDNS 查询报文，持有指定域名的主机同样以多播响应自身的域名和 IP，所有开启 mDNS 服务的主机在接收到响应后更新自身的 mDNS 缓存  

## 问题  