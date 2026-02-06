---
title: Android-隐私权-本地网络定义
date: 2026-2-6 15:24:01 +0800
categories:
  - Android
  - Privacy
  - Local_network_protections
tags:
  - Android
  - Privacy
  - Local_network_protections
description: 本地网络
math: true
---
本项目中的本地网络是指使用支持广播的网络接口（例如 Wi-Fi 或以太网）的 IP 网络，但不包括移动网络 (WWAN) 或 VPN 连接。

以下网络被视为本地网络：

|IPv4|IPv6|
|---|---|
|- 169.254.0.0/16 //Link Local  <br>- 100.64.0.0/10 // CGNAT  <br>- 10.0.0.0/8 // RFC1918  <br>- 172.16.0.0/12 // RFC1918  <br>- 192.168.0.0/16 // RFC1918|- 链路本地  <br>- 直接连接的路由  <br>- 线程等桩网络  <br>- 多个子网|

此外，多播地址 (224.0.0.0/4、ff00::/8) 和 IPv4 广播地址 (255.255.255.255) 都被归类为本地网络地址。