---
title: 硬路由（1）—— 概述 & OpenWrt 安装
date: 2024-08-09 21:23:03
categories:
- 硬路由
- OpenWrt
tags:
- 概述
- AX3000T
---

{% post_link 'NAS-Router-0' '回到目录' %}
<br/>

# 概述

以下我们涉及的均为主路由的配置部分。除主路由功能外，还有两种常见的功能：

- 旁路由（旁路网关）：本质上是一个通过 LAN 口与主路由连接的一个客户端设备，管理挂靠在主路由网络下的一个旁系网络，分担一部分路由器的功能。
- 单臂路由：在路由器的一个接口上通过配置子接口（逻辑接口）的方式，实现原来相互隔离的不同 VLAN 之间的互联互通。

与软路由相比，硬路由有着为路由组网功能特化的硬件。特别是在满足日常需求的条件下，硬路由的性价比要远高于软路由。（除非你手头上刚好有一台闲置的机器，且有双网口）

> LeSnow：当然，虽然 OpenWrt 只需要最低 16MB RAM即可运行，但过低的配置则会大大限制 OpenWrt 的功力，尤其是想实现科学上网，那么性能（RAM 和 算力）是务必需要考虑的。常见的家用路由为了节约成本，硬件性能往往是刚刚够用的。此外，OpenWrt 作为开源系统，对于很多硬件平台的优化是不如原厂系统的。（所以也会看到一些打包了闭源驱动的 OpenWrt 魔改版本）因此，就算一台家用路由器可以刷入 OpenWrt，一些服务（如科学上网）也是无法跑起来的，带宽也可能跑不满。

笔者于 2024 年 6 月在 pdd 平台购入了一台小米 AX3000T 路由器，花费 140.1 元。它采用了 **Filogic 820** 平台，虽然不支持 Wi-Fi 7，但对于笔者学校宿舍的有线百兆环境来说完全足够。

> LeSnow：AX3000T 在刷机上的方便之处：可以直接 SSH，不用 TTL

> LeSnow：另一款推荐的硬路由为 RAX3000M，刷机教程也很多，详见[感谢 Wi-Fi7，让移动定制路由器也可以很香！错过90多元的RAX3000M 普通版，135元入坑RAX3000M算力版！](https://post.smzdm.com/p/apmozp4w/ "感谢 Wi-Fi7，让移动定制路由器也可以很香！错过90多元的RAX3000M 普通版，135元入坑RAX3000M算力版！")

对于支持刷机的硬路由设备，我们可以刷入如 OpenWrt 一类的嵌入式设备操作设备，实现路由器端广告屏蔽、科学上网等高级功能。在此之前，我们需要了解一些相关的知识。

## Uboot

先让我们来看看更常见的 PC 机的启动过程：

- BIOS / UEFI 程序部署在主板特定芯片上，操作系统部署在硬盘上
- 上电后，PC 先执行 BIOS 程序，由 BIOS 程序负责初始化内存、硬盘，将 OS 镜像读取到内存中，最后跳转到内存中执行 OS 直至启动

相对应地，嵌入式系统的启动过程：

- Uboot 程序部署在能作为启动设备的 Flash 上，操作系统部署在 Flash 上
- 上电后，嵌入式系统先执行 Uboot 程序，由 Uboot 程序负责初始化内存、Flash，将 OS 镜像读取到内存中，最后跳转到内存中执行 OS 直至启动

因此，可以将 Uboot 程序类比成 PC 中的 BIOS，引导系统启动。对于这一类程序，更通用的描述是 BootLoader，Uboot 只是其中一种。

不死 Uboot（Breed）是路由器刷机常见的 Uboot 之一，由国内个人[hackpascal](https://github.com/hackpascal "hackpascal") 开发。

为什么会被称为“不死鸟”？有些官方升级固件自带 Bootloader，如果从官方固件升级，会导致现有 Bootloader 被覆盖。而当 Breed 更新固件时，它会自动删除固件附带的引导加载程序，因此可以防止 Breed 被覆盖。

## OpenWrt

OpenWrt 项目是针对嵌入式设备的 Linux 操作系统，常用在路由器上。OpenWrt 高度模块化、自动化，不仅占用空间小，而且具有强大的网络组件。

OpenWrt 项目始于 2004 年 1 月，其第一个版本采用了 LinkSys 的源码。在 LinkSys 的代码收费后，改为 Linux 内核集成，并将 OpenWrt 完全模块化，不断推出补丁和驱动。

从 OpenWrt 官方衍生出 3 个项目，加入了一些官方列表没有，针对国内具体情况定制的插件，且源码和开发规范与官方同步，保证了软件的纯净和稳定，目前网络上的很多固件都是针对这3个固件进行编译。

- [Lienol](https://github.com/Lienol/openwrt "Lienol")：紧跟官方版本进行更新，其编译出来的固件体积比其它版本小，集成的软件少，源码改动少，使用起来稳定性更强。
- [Lean](https://github.com/coolsnowwolf/lede "Lean")：由恩山论坛的 Lean 分享了全部开源代码，由于加入了很多中国特色的功能，并告诉所有人怎么通过编译去得到自己想要的OpenWRT，定制灵活性很强。
- [ImmortalWrt](https://github.com/immortalwrt/immortalwrt "ImmortalWrt")：OpenWrt 的一个分支，移植了更多的软件包，支持了更多的设备，更好的性能，并为中国大陆用户进行了特殊的优化。

前两者需要自行编译，ImmortalWrt 可直接选择固件下载。

余下的篇幅就以这台 AX3000T 为例，记录一下笔者刷入 OpenWrt 系统以及配置系统等经验。

# AX3000T刷入OpenWrt

**刷机有风险！操作须谨慎！**

若操作不当，可能会导致路由器“变砖”：由于各种原因不再正常工作。如果遇到 BootLoader 损坏，或是某些路由器不支持 SSH 刷机的情况下，只能采用 TTL 串口刷机 / 救砖。

TTL 刷机的方法较为复杂，主要是需要拆开路由器，通过焊接的方式引出 RX（接收）、TX（发送）以及 GND（接地）线，再与电脑上的软件进行通信，刷入 Uboot。

![某款路由器的 RX、TX、GND 位置示意图](image_BZ-l7jczTx.png "某款路由器的 RX、TX、GND 位置示意图")

以下部分主要来自于[这篇教程](https://www.bilibili.com/opus/902093937054842887 "这篇教程")。

针对可能会遇到的问题，笔者会以“踩坑记录”的形式列出需要注意的地方。

需要的文件：

[https://pan.baidu.com/s/1zDg5E5fqRAL7BX30UsVi0g?pwd=2333](https://pan.baidu.com/s/1zDg5E5fqRAL7BX30UsVi0g?pwd=2333 "https://pan.baidu.com/s/1zDg5E5fqRAL7BX30UsVi0g?pwd=2333")

提取码：2333

解压密码：Ankochovy

**连接路由器的电脑需使用 Windows 系统。**

**建议在刷机时关闭电脑自带防火墙等杀毒软件。**

## 降级固件到 1.0.47

如果固件已经是 1.0.47 可跳过此步骤

1. 用电脑的有线网口连接路由器的中间两个网口的任意一个
2. 打开降级工具，按照工具中的提示进行系统降级

   ![降级工具](image_2L6nP6QpET.png "降级工具")
   - 用卡针顶住路由器 Reset 按钮，再插上电源。持续按住 Reset 按键 8s 左右，看到指示灯闪烁。
   - 上传文件夹中的 1.0.47 固件。上传成功后等待 3\~5 分钟，看到路由器蓝灯闪烁。此时可以手动断电重启路由，降级成功。

## 解锁 SSH

1. 降级完成后进入路由器后台 192.168.31.1 完成基本设定
   - 如果 IP 不是这个，在 cmd 中输入`ping miwifi.com`
2. 从路由器后台的 URL 中获取 stok（红框中从`stok=`到`/web/`之间的部分）

   ![图中选中的部分](image_EjLOUuVCFN.png "图中选中的部分")
3. 进入 cmd
4. 依次输入以下命令（*把"****替换这里****"换成你自己的stok，输入每条命令输入回车后弹出{code：0} 则代表成功，我也不知道失败会是什么样子*）
   ```纯文本
   curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=替换这里/api/misystem/arn_switch -d "open=1&model=1&level=%0Anvram%20set%20ssh_en%3D1%0A"


   curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=替换这里/api/misystem/arn_switch -d "open=1&model=1&level=%0Anvram%20commit%0A"


   curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=替换这里/api/misystem/arn_switch -d "open=1&model=1&level=%0Ased%20-i%20's%2Fchannel%3D.*%2Fchannel%3D%22debug%22%2Fg'%20%2Fetc%2Finit.d%2Fdropbear%0A"


   curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=替换这里/api/misystem/arn_switch -d "open=1&model=1&level=%0A%2Fetc%2Finit.d%2Fdropbear%20start%0A"
   ```

## 连接 SSH

1. 使用工具计算出 SN 密码，先在路由器后台页面复制右下角的 SN

   ![复制 SN 处](image_74s4mSKI50.png "复制 SN 处")
2. 通过 [https://miwifi.dev/ssh](https://miwifi.dev/ssh "https://miwifi.dev/ssh") 网址输入 SN 计算出密码（网盘中也有【计算 SSH 密码工具】）
3. 使用 Termius 或者 MobaXterm（附件中有）等 SSH 工具连接路由器 SSH，地址为`192.168.31.1`，账号为`root`，密码就是第二步中计算得出的密码
4. 看到 ARE U OK 则连接成功

## 备份（很重要，一定要备份!!!）

SSH 连接成功后输入`cat /proc/mtd`

![终端显示结果](image_FftbBF0513.png "终端显示结果")

备份所有分区。

- 踩坑记录：不要一次性备份所有文件。路由器的存储空间不够。可以先备份两三个文件，再从`/tmp`目录拷出，最后删除备份文件，继续备份。有的文件比较大，备份时间比较长，请耐心等待。

```bash
dd if=/dev/mtd1 of=/tmp/BL2.bin
dd if=/dev/mtd2 of=/tmp/Nvram.bin
dd if=/dev/mtd3 of=/tmp/Bdata.bin
dd if=/dev/mtd4 of=/tmp/Factory.bin
dd if=/dev/mtd5 of=/tmp/FIP.bin
dd if=/dev/mtd6 of=/tmp/crash.bin
dd if=/dev/mtd7 of=/tmp/crash_log.bin
dd if=/dev/mtd8 of=/tmp/ubi.bin
dd if=/dev/mtd9 of=/tmp/ubi1.bin
dd if=/dev/mtd10 of=/tmp/overlay.bin
dd if=/dev/mtd11 of=/tmp/data.bin
dd if=/dev/mtd12 of=/tmp/KF.bin
```

![备份成功记录](image_LDjYONB-on.png "备份成功记录")

出现 in 与 out 提示则说明备份成功。将备份文件从`/tmp`目录拷贝出来后删除。

## 刷入 Uboot

在 SSH 客户端中打开`/tmp`文件夹，将 uboot 文件拖入至该文件夹中。

\*文件名：`mt7981_ax3000t-fip-fixed-parts-multi-layout.bin`

在 SSH 命令行中输入：

```bash
mtd write mt7981_ax3000t-fip-fixed-parts-multi-layout.bin FIP
```

开始刷入 Uboot（刷完后不会有提示，保险起见，开始命令后推荐等待 10 分钟或更久后拔电源）

## 刷入 OpenWrt

1. 按住路由器的 Reset 键再插电，Reset 键一直按住等待 10 秒左右，等待指示灯变蓝
2. 手动设置电脑的 IP 地址设置为`192.168.1.2`，网关设定为`255.255.255.0`，在浏览器里输入`192.168.1.1`进入 uboot 页面
3. 在 uboot 界面里选择 qwrt，对于 AX3000T，这里不要选择附件中的 OpenWrt 固件安装
   - 踩坑记录：附件中的 OpenWrt 固件（BleachWrt）过于臃肿，预装了 200+ 软件包，在配置过程中极容易使得 AX3000T 爆内存，导致运行不稳定，不建议使用。
   - 笔者使用的固件：[2024-7-5 | XiaoWanSM-固件仓库 (wwang.pw)](https://pan.wwang.pw/🔥OpenWrt_LEAN/小米AX3000T/2024-7-5 "2024-7-5 | XiaoWanSM-固件仓库 (wwang.pw)")，使用了近一个月测试下来较为稳定。\*固件文件名：`openwrt-mediatek-filogic-xiaomi_mi-router-ax3000t-squashfs-sysupgrade.bin`
   - 如果想安装 ImmortalWrt 固件，需要刷入其他 uboot。对于支持同一个 uboot 的不同固件，如果想更改安装，可以在 LuCi 的`系统 - 备份/升级 - 刷写新的固件`处上传其他固件。对于不支持同一个 uboot 的不同固件间的刷写方法暂时未知。
     ![LuCi-刷写新的固件](image_vxwp5OJnUH.png "LuCi-刷写新的固件")
4. 完成固件刷写大约花费 5 分钟，刷完后路由器会自动重启，然后就可以进入路由器后台进行各种设置配置了（刷完后记得将电脑 IP 设置回自动获取）
5. 路由器后台地址为`192.168.1.1`，登录用户名为`root`，默认密码为`password`

<br/>
{% post_link 'Router-2' '下一章节：系统调优 & 局域网组网' %}

