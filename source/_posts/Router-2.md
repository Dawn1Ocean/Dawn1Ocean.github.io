---
title: 硬路由（2）—— 系统调优 & 局域网组网
date: 2024-08-09 22:03:42
categories:
- 硬路由
- OpenWrt
tags:
- LuCi
---

{% post_link 'NAS-Router-0' '回到目录' %}
<br/>

# 写在前面

AX3000T 的 RAM 为 256MB 左右，运行 OpenClash（请自行查询此软件包的用途）较为吃力。在配置前，请明确自己的需求，并删除对自己无用的软件包。命令如下：

```bash
opkg remove --autoremove <软件包名>
```

我们可以使用 ShellCrash（请自行查询此软件包的用途）脚本来替代 OpenClash，前者内存占用更小。

有些固件会自带 Turbo ACC 网络加速软件包，请保持默认配置，不要勾选`MTK 无线硬件加速`、`BBR 拥塞控制算法`以及`DNS 缓存`选项，否则会导致路由器无法行使正常功能。

**重要提示：** 安装的固件当中预装的软件包一般与内核版本与固件绑定，相互的依赖关系十分复杂，更新软件包前请慎重。**请勿更新 OpenWrt 内核。** 如果你的 OpenWrt 系统因软件包更新与内核等依赖出现了问题，建议将系统恢复出厂设置，这样是最快并且最简洁的解决方式。

如果还能通过 SSH 等方式打开路由器终端，请输入以下命令：

```bash
firstboot
reboot
```

重启后路由器即恢复原样，按照说明重新配置一遍即可。

# 基础配置

为了安全起见，建议在`系统 - 管理权`处重新设置管理员密码。

![](image_LtMFIDYShS.png)

由于`192.168.1.1`地址可能会与运营商设备产生冲突，建议在`网络 - 接口 - LAN`处设置协议为静态协议，IPv4 地址为其他地址。

![](image_5P_A5b9_qq.png)

同时在`系统 - Web 管理`处将 HTTP 监听地址更改为新的 LAN 口地址。

![](image_1IqvUBng0N.png)

OpenWrt 采用的包管理器为 opkg。在执行后续操作时，请先根据 WAN 接口配置以及{% post_link 'Router-3' '下一章节：校园网 & L2TP / 网页认证' %}部分，确保路由器能够访问公网。在`系统 - TTYD 终端`处登录`root`，输入：

```bash
opkg update
```

来进行包的更新。如果无法更新，请尝试更换镜像源。

更换镜像源的方法：在`系统 - TTYD 终端`处登录`root`，输入：

```bash
cd /etc/opkg
vi disfeeds.conf
```

根据软件版本替换镜像源。笔者的配置：

```text
src/gz openwrt_core https://mirrors.cloud.tencent.com/lede/snapshots/targets/mediatek/filogic/packages                                                                    
src/gz openwrt_base https://mirrors.cloud.tencent.com/lede/snapshots/packages/aarch64_cortex-a53/base                                                                     
src/gz openwrt_luci https://mirrors.cloud.tencent.com/lede/snapshots/packages/aarch64_cortex-a53/luci                                                                     
src/gz openwrt_packages https://mirrors.cloud.tencent.com/lede/snapshots/packages/aarch64_cortex-a53/packages                                                             
src/gz openwrt_routing https://mirrors.cloud.tencent.com/lede/snapshots/packages/aarch64_cortex-a53/routing                                                               
src/gz openwrt_telephony https://mirrors.cloud.tencent.com/lede/snapshots/packages/aarch64_cortex-a53/telephony
```

替换完成后，再次输入

```bash
opkg update
```

以更新 opkg 源。

# LAN 接口配置

在`网络 - 接口 - LAN - 高级设置`处确认接口选项是否勾选了所有带"lan"的适配器以及所有处于 Master 模式的无线适配器。

![](image_PSG0oKIsPI.png)

# WAN 接口配置

在`网络 - 接口 - WAN - 基本设置`处

如果没有给路由器预先分配好的静态 IP，一般采用 DHCP 客户端协议。本文不涉及 PPPoE 拨号上网。

![](image_85gXepEvM8.png)

如果处于路由器配置阶段，想使用网线给路由器共享网络，请将电脑的有线网卡与路由器 WAN 口均配置为静态地址。

有线共享网络方法：

1. 首先在路由器的`网络 – 无线`页面保证无线是开启的，如果不是请点击`重启无线`打开。这里可以直接配置自己的无线名字和密码，配好后用手机连接上。
   注意，重启无线后可能需要在路由器中执行该命令，原因请参考{% post_link 'Router-3' '下一章节：校园网 & L2TP / 网页认证' %}部分：`ip rule add from all lookup main pref 0`
2. 在路由器的`网络 – 接口 - WAN`  页面中修改为静态IP，IP地址是 192.168.3.2，子网掩码 255.255.255.0，网关 192.168.3.1，DNS 223.5.5.5，其它留空即可，保存应用。
3. 电脑的有线口连接路由器的 Wan 口，电脑的无线连接其它可以上网的 Wi-Fi（不能是一会要使用的手机的热点，否则还需要一个设备）。
4. 进入`控制面板 - 网络和共享中心 - 更改适配器设置`，在弹出的窗口中右击`WLAN – 属性`，在上面点击`共享`，首先在下面的下拉框中选择你的有线网卡（一般是`以太网`），再勾选`允许其他网络用户通过此计算机的 Internet 连接`来连接。如果弹出任何对话框请确定。

   ![](image_Ueqw5TL7o1.png)
5. 在网络连接的窗口右击`以太网–属性`，往下找到 Internet 协议版本 4，双击，点击“使用下面的 IP 地址”，修改地址为192.168.3.1，子网掩码 255.255.255.0，确定即可。

   ![](image_3_NUJh-mf0.png)

# 无线配置

常用的无线模式有以下两种：

1. 接入点 AP（Access Point）：最常见的模式，能将有线信号转变为无线 WiFi 信号，能够设置 WiFi 的 SSID，认证方式等
2. 客户端（Client）：将路由器当成无线网卡使用，能接收无线信号，通过有线方式传递出去，可以在路由器配置阶段临时使用，较为方便

在`网络 - 无线 - 对应无线网络`处配置。在`无线安全`处设置认证方式。

![](image_RJoHFs5B1F.png)

如果处于路由器配置阶段，想使用已有的无线网络给路由器共享网络，请在`网络 - 无线 - 无线概况`中点击对应频段的扫描按钮，并连接对应无线网络即可。对于路由器连接校内 WiFi，请参照{% post_link 'Router-3' '硬路由（3）—— 校园网 & 认证方案' %}

![](image_-rSiwThd70.png)

![](cad0e47b12bc1e9dbfd8435b8d3aa5db_fTvpoCs0-_.png)

# 静态地址分配

对于长期处于路由器下局域网的设备，如果我们希望该设备在局域网中的 IP 地址不发生改变，而对于其他临时加入局域网的设备采取 DHCP 方式分配 IP 地址，我们可以在路由器端设置静态地址分配，即对于同一个物理设备（MAC 地址相同），我们创建一个无限期（infinite）的 DHCP 租约，即可实现需求。

在`网络 - DHCP/DNS - 静态地址分配`处添加静态租约，填写主机名，MAC 地址，IPv4 地址，租期。

![](image_busw032GsT.png)

如果设备已经被 DHCP 分配地址，可以重新连接路由器以获得永久（或自定义时长）租约。

<br/>
{% post_link 'Router-3' '下一章节：校园网 & 认证方案' %}
