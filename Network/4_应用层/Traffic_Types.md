---
title: Traffic Types
created: 2025-03-10
tags:
---
> [!cite]- References  
> [Introduction to Multicast](https://networklessons.com/multicast/introduction-to-multicast)  
> [【Network】单播（Unicast）、多播（Multicast）与广播（Broadcast） | 西维蜀黍 的博客](https://swsmile.github.io/2017/06/04/%E3%80%90Network%E3%80%91%E5%8D%95%E6%92%AD%EF%BC%88Unicast%EF%BC%89%E3%80%81%E5%A4%9A%E6%92%AD%EF%BC%88Multicast%EF%BC%89%E4%B8%8E%E5%B9%BF%E6%92%AD%EF%BC%88Broadcast%EF%BC%89/)  

> [!question] which layer should this article on?  
## **单播**(unicast)  

单播是以目的地址为单一目标的一种传输方式  

寻址时，发送方与接收方 1:1 关联  

在单播下，网络中的每一个节点都有**唯一**的 IP 地址  
对于 IPv4，0.0.0.0 到 223.255.255.255 属于单播地址  
 #TBD more details required  
## **选播/任播**(anycast)  

寻址时，发送方与接收方 1:1(1 of N) 关联  
Anycast 网络中的节点在公开网络上宣告相同的 IP 地址，发送方请求通过 BGP 协议路由到任一最近的节点  

在 IPv4 中，任播与单播共用地址池  
在 IPv6 中，任播以特殊格式区分  
 #TBD more details required  
## **多播/组播**(multicast)  

多播是以一组目的地址为目标的传输方式  
多播的一组目的地址称之为**多播组**(multicast group)，同一组内的目的地址不一定位于同一网络内  

多播仅支持 [UDP](Network/3_传输层/UDP.md)  

寻址中，发送方与接收方之间 1:N 或 N:N 关联  
在一个单一的传输中，数据报被同时发送给多个收件人  

对于 IPv4，224.0.0.0 到 239.255.255.255 为多播地址  
且多播地址仅能用于目的地址，不能用于源地址  
## **广播**(broadcast)  

广播是以某一网络中所有设备为目标的传输方式  

IPv6 不支持  
广播特性由多播实现  
 #TBD more details required  
### 广播风暴  
