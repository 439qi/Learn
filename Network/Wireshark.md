---
title: Wireshark
created: 2025-03-10
tags:
  - manual
---
> [!cite]- References  
> [wireshark网络抓包工具使用教程 | hengshuai's blog](https://blog.usword.cn/frontend/debug-skill/wireshark.html)  

## 简介  

[Wireshark](https://gitlab.com/wireshark/wireshark/-/tree/master) 是一款开源免费跨平台的网络抓包工具  

## 混杂模式

在「菜单栏」=>「捕获」=>「选项」打开的对话框中，可以选择是否开启混杂模式  

在混杂模式下，Wireshark 不验证 MAC 地址，会捕获所有经由网卡的数据包  
在普通模式下，Wireshark 仅捕获发送给本机的包(包括广播)  


## 过滤器

Wireshark 提供了过滤器用于从大量抓包记录中过滤指定的记录  
过滤器分为两种，捕获过滤器和显示过滤器  

捕获过滤器在捕获时即进行过滤，不满足条件的记录不会进行捕获  

显示过滤器在显示时进行过滤，其不会干预捕获时的行为，仅在显示抓包记录时进行过滤  
