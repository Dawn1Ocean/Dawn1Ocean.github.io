---
title: 硬路由（4）—— Tailscale & 远程访问
date: 2024-08-09 22:21:04
categories:
- 硬路由
- OpenWrt
tags:
- LuCi
- Tailscale
---

{% post_link 'NAS-Router-0' '回到目录' %}
<br/>

# Tailscale 与 VPN

如果想要远程访问路由器局域网内的设备，有两个办法：

1. 路由器暴露在公网，并作端口映射 / 转发使得远程设备能够访问到局域网内的设备
2. 组建虚拟局域网，使得想要访问的公网设备也在这个局域网中

> LeSnow：虚拟局域网 其实也算暴露在公网

> LeSnow：异地组网的必要性也体现在对 NAS 的访问上。最方便的协议是 SMB，SMB （139 / 445端口）一般是不能跑在公网上的，所以需要组网。一些专用 NAS 系统也是默认不运行在公网环境下的，它们可能连防火墙配置都没有（比如 TrueNAS）

方案 1 对路由器安全配置的要求较高，针对日常使用的情况，我们采用方案 2。

Tailscale 是一个基于 WireGuard 的 VPN 配置工具，通过去中心化的方式实现各个节点之间点对点的连接。配置简单友好，跨平台支持。

![](image_N6QaMRZ5dn.png)

# 安装与配置 Tailscale

本部分文章主要来自于[这篇博客](https://www.cnblogs.com/lumia1998/p/18241680 "这篇博客")。

## 下载软件

进入`系统 - TTYD 终端`，将 Tailscale 软件包下载到指定目录。进入 Tailscale 的[Github Releases](https://github.com/adyanth/openwrt-tailscale-enabler/releases "Github Releases")地址，找到最新的软件包，使用 wget 下载到本地。

```bash
wget https://github.com/adyanth/openwrt-tailscale-enabler/releases/download/v1.60.0-e428948-autoupdate/openwrt-tailscale-enabler-v1.60.0-e428948-autoupdate.tgz # 版本自行调整
```

## 解压软件

```bash
tar x -zvC / -f openwrt-tailscale-enabler-v1.60.0-e428948-autoupdate.tgz
```

## 安装依赖包

```bash
opkg update
opkg install libustream-openssl ca-bundle kmod-tun
```

## 设置开机启动，验证开机启动

```bash
/etc/init.d/tailscale enable
ls /etc/rc.d/S*tailscale*
```

## 启动 Tailscale

```bash
/etc/init.d/tailscale start
```

## 获取登录链接并配置路由

```bash
tailscale up
```

![](image_WGytgUptja.png)

复制显示的地址，并在浏览器中打开，使用谷歌 / 微软 / GitHub 等帐号登录 Tailscale 的管理主页进行验证。

## 开启子网路由

子网路由功能使得启动 Tailscale 服务的路由器能够将流量中继至物理子网设备。换言之，只需要路由器配置 Tailscale，就可以使得路由器组建的局域网中的设备均可被远程访问。

输入以下命令，打开本地子路由。子网地址是 OpenWrt 的 Lan 网络。请自行填写子网地址。

```bash
tailscale up --accept-routes --accept-dns=false --advertise-routes=192.168.31.0/24
```

在 Tailscale 的管理页面上，单击设备列表右侧的更多图标，禁用密钥过期，并打开子网路由。

![](image_1Gxvc0B53P.png)

![](image_kosUa1zK-o.png)

现在在 OpenWrt 上已经可以 ping 通其他 Tailscale 节点了，但其他节点还无法连接 OpenWrt 节点，还需要在 OpenWrt 上添加 Tailscale 接口。

## 添加接口

在`网络 - 接口`处新建一个接口，协议选静态地址，设备选`tailscale0`，地址为 Taliscale 管理页面上分配的地址，掩码`255.0.0.0`。防火墙区域选 lan 区域。

![](image_WxmKBO8jVc.png)

![](image_Jp1n2jCHEL.png)

## 添加防火墙规则

在`网络 - 防火墙 - 自定义规则`处添加以下规则：

![](image_a716NxxW20.png)

```text
iptables -I FORWARD -i tailscale0 -j ACCEPT
iptables -I FORWARD -o tailscale0 -j ACCEPT
iptables -t nat -I POSTROUTING -o tailscale0 -j MASQUERADE
```

现在各个Tailscale节点之间已经可以正常互访了。

## 局域网中其他设备的配置

踩坑记录：不要在局域网中其他设备启动 Tailscale 服务，否则可能会使得该设备无法访问路由器。

原因：访问路由器，在路由表中属于访问设备所在的局域网。启动 Tailscale 服务后，对于此局域网，设备会存在两个接口：一个是由 Tailscale 分配的地址，另一个是实际上的局域网地址。如果由 Tailscale 分配地址的跃点数比实际上局域网地址的跃点数小，设备就不会使用物理接口来发送到下一跳设备，而是使用 Tailscale 分配的地址，因此数据包无法送达。

![](image_RngM9JHYDq.png)

# 远程访问局域网

对于如 Windows 这样可安装 Tailscale 客户端的系统环境，在应用中确认已连接 Tailscale，就可实现远程访问局域网。

对于 Linux 等以软件包形式安装 Tailscale 客户端的系统环境，我们需要多运行一条命令来接受子网路由：

```bash
sudo tailscale up --accept-routes
```

