---
title: NAS / 硬路由：从入门到入门
date: 2024-08-09 21:04:06
categories:
- NAS
tags:
- 概述
---

为了获得更良好的体验，在阅读前最好对以下知识有部分了解：

- Linux & 系统终端的使用
- 计算机网络
- Docker
- Nginx

# 引言

众所周知，一台服务器能实现的事情有很多。如果你有以下需求：

- 大量数据存储需求（比如电影、番剧、小姐姐
- 多个设备间的数据同步、储存与访问
- 自己的（家庭）影音系统
- PT（Private Tracker）下载
- ......

那你就可以考虑组装一台 NAS（Network Attached Storage，网络附属存储）了！

为了方便管理网络流量，并且提高本地环境对 NAS 的访问速度，NAS 通常借由路由器与日常使用的网络设备，如 PC、手机、平板等组成本地局域网。也可以进一步使用 Tailscale、WireGuard、ZeroTier 等工具实现异地组网，以便在任何有互联网连接的地方访问局域网中的设备。

接下来的篇幅涉及两大部分：硬路由与 NAS。硬路由是专为路由功能设计的独立硬件设备，也就是我们常说的路由器；软路由是基于软件工具在普通的硬件上来实现传统路由器的功能。在硬路由部分，我们会使用 OpenWrt 系统，这也是在软路由配置中常用的系统，在通用配置部分可以相互借鉴。

NAS 除了可以完成它的名字中包含的网络存储功能，还能作为一台服务器运行各种服务。借助 Docker 与虚拟机技术，NAS 能做到的远不止网络存储这么简单。

如果你和笔者一样喜欢折腾的话，笔者很推荐你基于原生 Debian 系统从零开始搭建起属于自己的一台 NAS。也希望这一系列文章能够给你提供些许经验。让我们开始吧！

{% post_link 'Router-0' '硬路由（0）—— 编译属于自己的 OpenWrt' %}
<br/>
{% post_link 'Router-1' '硬路由（1）—— 概述 & OpenWrt 安装' %}
<br/>
{% post_link 'Router-2' '硬路由（2）—— 系统调优 & 局域网组网' %}
<br/>
{% post_link 'Router-3' '硬路由（3）—— 校园网 & 认证方案' %}
<br/>
{% post_link 'Router-4' '硬路由（4）—— Tailscale & 远程访问（IPv4）' %}
<br/>
{% post_link 'Router-5' '硬路由（5）—— 校园网 & 远程访问（IPv6）' %}
<br/>
{% post_link 'Router-6' '硬路由（6）—— Tricks & 组网图景' %}
<br/>
<br/>
{% post_link 'NAS-1' 'NAS（1）—— 概述 & 硬件' %}
<br/>
{% post_link 'NAS-2' 'NAS（2）—— 基于原生 Debian 搭建系统' %}
<br/>
{% post_link 'NAS-3' 'NAS（3）—— Cockpit & 系统调优' %}
<br/>
{% post_link 'NAS-4' 'NAS（4）—— UPS 不断电系统' %}
<br/>
{% post_link 'NAS-5' 'NAS（5）—— 硬盘 & 软 RAID' %}
<br/>
{% post_link 'NAS-6' 'NAS（6）—— SMB & WebDAV' %}
<br/>
{% post_link 'NAS-7' 'NAS（7）—— Docker 服务' %}
<br/>
{% post_link 'NAS-8' 'NAS（8）—— Nginx & HTTPS' %}
<br/>
{% post_link 'NAS-9' 'NAS（9）—— Homepage 导航页' %}
<br/>
{% post_link 'NAS-10' 'NAS（10）—— PT' %}
<br/>
{% post_link 'NAS-11' 'NAS（11）—— HTPC' %}
<br/>
{% post_link 'NAS-12' 'NAS（12）—— Immich 照片管理' %}
<br/>
{% post_link 'NAS-13' 'NAS（13）—— Navidrome + MusicTag 音乐管理' %}
<br/>