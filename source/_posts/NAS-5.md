---
title: NAS（5）—— 硬盘 & 软 RAID
date: 2024-08-26 02:43:31
categories:
- NAS
- RAID
---

{% post_link 'NAS-Router-0' '回到目录' %}
<br/>

# 硬盘健康监测

## SMART

S.M.A.R.T. 代表的是 Self-Monitoring Analysis and Reporting Technology，意思是自我检测、分析及报告技术。可以用来预测和分析硬盘的潜在问题。

对于机械硬盘（HDD）的寿命信息，我们主要关注这几个条目：

- 重新分配扇区系数（05）
- 当前等待中扇区数（C5）
- **无法修正的扇区总数（C6，重要，不为零即建议备份数据、换盘）**

对于固态硬盘（SSD）的寿命信息，我们主要关注这几个条目：

- 可用备用空间（03）
- 已用寿命百分比（05）
- **媒体与数据完整性错误计数（0E，重要，不为零即建议备份数据、换盘）**

在 Windows 系统，我们可以使用 [CrystalDiskInfo](https://crystalmark.info/en/software/crystaldiskinfo/ "CrystalDiskInfo") 这款软件来查看硬盘的 Smart 信息。

在 Linux 系统，我们可以使用`smartmontools`包来查看硬盘的 Smart 信息。

```bash
sudo apt install smartmontools -y
```

要确保您的驱动器支持 SMART，请键入：

```bash
sudo smartctl -i /dev/sda
```

其中`/dev/sda`一般为机械硬盘 / U 盘，固态硬盘如`/dev/nvme0n1p1`，可以通过`lsblk`命令查看各硬盘。

如果您的驱动器未启用 SMART，您可以通过键入以下内容来启用它：

```bash
sudo smartctl -s on /dev/sda
```

检查硬盘整体健康状态：

```bash
sudo smartctl -H /dev/sda
```

此命令应返回：

```text
=== START OF READ SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED
```

如果它没有返回 PASSED，您应该立即备份所有数据。您的硬盘驱动器可能出现故障。

## Scrutiny

Scrutiny 是用于smartd 监控硬盘 S.M.A.R.T 信息的 WebUI。

![](image_v5JOICGCKw.png)

### 通过Docker部署

可以先阅读{% post_link 'NAS-7' 'Docker 服务' %}部分，将 NAS 上的 Docker 环境配置完成。

使用 Docker Compose：

```yaml
version: "3.5"
services:
    scrutiny:
      image: ghcr.io/analogj/scrutiny:master-omnibus
      container_name: scrutiny
      cap_add:
        - SYS_RAWIO
      environment:
        - PUID=1001 # PUID 与 PGID 请根据实际情况设置
        - PGID=100
        - TZ=Asia/Shanghai
      volumes:
        - /srv/dockerapps/scrutiny/config:/config
        - /run/udev:/run/udev:ro
        - /srv/dockerapps/scrutiny/influxdb:/opt/scrutiny/influxdb
      ports:
        - "8082:8080" # 端口请根据实际情况设置，8080 为 Web 端口
        - "8086:8086" # 8086 为 InfluxDB 数据库端口
      devices:     
        - "/dev/sda" # 通过 lsblk 命令查看硬盘
        - "/dev/sdb"
        - "/dev/sdc"
        - "/dev/nvme0n1"
      restart: unless-stopped
```

服务启动后访问对应端口即可查看硬盘详细信息。记得开放防火墙对应端口。

![](image_38Or3UI6Wm.png)

# RAID

使用 RAID（Redundant Array of Independent Disks，独立磁盘冗余阵列），可以大幅提高硬盘设备的 I/O 读写速度，还有各种数据冗余备份机制供用户选择，能够降低用户数据盘损坏带来的数据丢失的风险。

RAID 技术通过把多个硬盘设备组合成一个容量更大、安全性更好的磁盘阵列，并把数据切割成多个区段后分别存放在各个不同的物理硬盘设备上，然后利用分散读写技术来提升磁盘阵列整体的性能，同时把多个重要数据的副本同步到不同的物理硬盘设备上，从而起到了非常好的数据冗余备份效果。

## 软 RAID 和硬 RAID

软件 RAID 是通过操作系统的软件来实现 RAID 功能。软件 RAID 不需要额外的硬件控制器，而是依赖于操作系统的 RAID 驱动程序，通过 CPU 处理 RAID 相关的计算和数据操作。

硬件 RAID 是通过在独立的硬件控制器上实现 RAID 功能。硬件控制器通常是一块专用的 RAID 卡，通过硬件来管理 RAID 阵列，包括数据读写、磁盘故障检测和数据恢复等。

相比于硬 RAID，软 RAID 有三个明显的缺点：

- 占用内存空间
- 占用 CPU 资源
- 软件的 RAID 程序无法将已经安装了操作系统的磁盘也做成 RAID

最后一点意味着，软 RAID 只有在操作系统成功启动后才能运行，如果运行过程中操作系统崩溃，组成 RAID 的磁盘中的数据将无法读写。

但在实际的使用过程中，硬 RAID 需要配备相应的硬件，配置的时间成本和花费都较高，在日常使用的情况下，我们通常采用软 RAID 方案即可。

## RAID 种类

这一部分主要参考[这篇文章](https://www.cnblogs.com/kuangdaoyizhimei/p/18314827 "这篇文章")。

### RAID 0

RAID 0 技术把多块物理硬盘设备（至少两块）通过硬件或软件的方式串联在一起，组成一个大的卷组，并将数据依次写入各个物理硬盘中。这样一来，在最理想的状态下，硬盘设备的读写性能会提升数倍，但是若任意一块硬盘发生故障，将导致整个系统的数据都受到破坏。通俗来说，RAID 0 技术能够有效地提升硬盘数据的吞吐速度，但是不具备数据误修复能力。

如下图所示，数据被分别写入到不同的硬盘设备中，两块硬盘会分别保存数据资料，最终实现提升读取、写入速度的效果。

![](image_z7GasqjUZW.png)

- 最少需要的硬盘数：2
- 优点：更快的数据访问
- 缺点：没有硬盘冗余，所以数据丢失的风险会增加。未针对不同容量大小的硬盘进行优化。

### RAID 1

RAID1模式的磁盘阵列的模式为“N+1”，无论有多少个磁盘，它只用其中一个来存储数据，其它每个磁盘都是它的镜像，举个例子：有两个磁盘，那磁盘利用率就是 50%，有 N 个磁盘，那磁盘利用率就是 1/N。

![](image_VtXfS8z50u.png)

- 最少需要的硬盘数：2
- 优点：N 个硬盘冗余
- 缺点：存储池可用空间被限制为一个硬盘的容量。未针对不同容量大小的硬盘进行优化。

### RAID 5

RAID 5 技术是把硬盘设备的数据奇偶校验信息保存到其他硬盘设备中。RAID 5 磁盘阵列中数据的奇偶校验信息并不是单独保存到某一块硬盘设备中，而是存储到除自身以外的其他每一块硬盘设备上。这样的好处是，其中任一设备损坏后都可以依赖其它几块设备重建丢失的这块硬盘数据。

RAID 5 需要使用至少 3 个驱动器，像 RAID 0 一样跨多个驱动器条带化数据，但也有跨驱动器分布的“奇偶校验”。在单个驱动器发生故障的情况下，使用存储在其他驱动器上的奇偶校验信息将数据拼凑在一起。停机时间为零。读取速度非常快，但由于必须计算奇偶校验，写入速度稍慢。它非常适合数据驱动器数量有限的文件和应用程序服务器。

![](image_cJcs4zH-qf.png)

- 需要的硬盘数：3
- 优点：1 个硬盘的容错机制并将存储空间配置优化
- 缺点：未针对不同容量大小的硬盘进行优化
  - 在磁盘故障的情况下，恢复可能需要更长的时间，因为必须从所有可用驱动器计算奇偶校验
  - 无法承受并发驱动器故障

### RAID 6

RAID 6 使用双奇偶校验块来实现比 RAID 5 更好的数据冗余，这增加了阵列中最多两个驱动器故障的容错能力。每个磁盘都有两个奇偶校验块，它们存储在阵列中的不同磁盘上，RAID 6 是用于维护高可用性系统的非常实用的基础架构。

![](image_RCHIYlrrPJ.png)

- 至少需要 4 个驱动器
- 优点：2 个硬盘冗余
- 缺点：由于双重奇偶校验，写入数据需要更长的时间。未针对不同容量大小的硬盘进行优化。

### RAID 10（RAID 1+0）

以上提到的 RAID 均为基本 RAID 级别，而 RAID 10、RAID 50、RAID 60 等属于跨区 RAID 级别。

RAID 10 技术需要至少 4 块硬盘来组建，其中先分别两两制作成 RAID 1 磁盘阵列，以保证数据的安全性；然后再对两个 RAID 1 磁盘阵列实施 RAID 0 技术，进一步提高硬盘设备的读写速度。

这样从理论上来讲，只要坏的不是同一阵列中的所有硬盘，那么最多可以损坏 50%的硬盘设备而不丢失数据。由于 RAID 10 技术继承了 RAID 0 的高读写速度和 RAID 1 的数据安全性，在不考虑成本的情况下 RAID 10 的性能也超过了 RAID 5，因此当前成为广泛使用的一种存储技术。

![](image_BfXDKEHJ4_.png)

## 通过 Cockpit 创建软 RAID

如果想使用 ZFS，可以跳过这里直接查看ZFS部分。

通过 Cockpit 创建软 RAID 的过程非常简单，在`存储`栏找到`创建 MDRAID 设备`：

![](image_rXjQ4U5lca.png)

根据自己的需求选择 RAID 级别及对应磁盘即可。

![](image_sHTpZSTZAA.png)

# ZFS

ZFS 被描述为“终极文件系统”，稳定、快速、安全并面向未来。

ZFS 的特点包括：存储池（被称为 "zpool" 的集成卷管理系统）、写时复制、快照、数据完整性校验和自动修复（擦除）、RAID-Z、最大 16 Exabyte 文件大小，以及最大 256×10¹⁵ Zettabyte 存储，且对文件系统（数据集）或文件的数量没有限制 。

![](image_poj7G05kCN.png)

- 存储池：与大多数文件系统不同，ZFS 结合了文件系统和卷管理器的特性。这意味着，它与其他文件系统不同，ZFS 可以创建跨越一系列硬盘或池的文件系统。不仅如此，你还可以通过添加硬盘来增大池的存储容量。ZFS 可以进行[分区和格式化](https://www.howtogeek.com/175159/an-introduction-to-the-z-file-system-zfs-for-linux/ "分区和格式化")。
- 写时复制：在大多数文件系统上，当数据被重写时，它将永久丢失。而在 ZFS 中，新数据会写到不同的块。写完成之后，更新文件系统元数据信息，使之指向新的数据块。这确保了如果在写新数据的时候系统崩溃（或者发生其它事，比如突然断电），那么原数据将会保存下来。这也意味着，在系统发生崩溃之后，不需要运行 [fsck](https://en.wikipedia.org/wiki/Fsck "fsck") 来检查和修复文件系统。
- 快照：ZFS 使用快照来跟踪文件系统中的更改。[快照](https://www.freebsd.org/doc/handbook/zfs-term.html "快照")包含文件系统的原始版本（文件系统的一个只读版本），实时文件系统则包含了自从快照创建之后的任何更改。没有使用额外的空间。因为新数据将会写到实时文件系统新分配的块上。如果一个文件被删除了，那么它在快照中的索引也会被删除。所以，快照主要是用来跟踪文件的更改，而不是文件的增加和创建。
- 数据完整性校验和自动修复：当向 ZFS 写入新数据时，会创建该数据的校验和。在读取数据的时候，使用校验和进行验证。如果前后校验和不匹配，那么就说明检测到了错误，然后，ZFS 会尝试自动修正错误。
- RAID-Z：ZFS 不需要任何额外软件或硬件就可以处理 RAID。ZFS 有自己的 RAID 实现：RAID-Z 。RAID-Z 是 RAID-5 的一个变种，不过它克服了 RAID-5 的写漏洞：意外重启之后，数据和校验信息会变得不同步。为了使用[基本级别的 RAID-Z](https://wiki.archlinux.org/index.php/ZFS/Virtual_disks#Creating_and_Destroying_Zpools "基本级别的 RAID-Z")（RAID-Z1），你需要至少三块磁盘，其中两块用来存储数据，另外一块用来存储[奇偶校验信息](https://www.pcmag.com/encyclopedia/term/60364/raid-parity "奇偶校验信息")。而 RAID-Z2 需要至少两块磁盘存储数据以及两块磁盘存储校验信息。RAID-Z3 需要至少两块磁盘存储数据以及三块磁盘存储校验信息。另外，只能向 RAID-Z 池中加入偶数倍的磁盘，而不能是奇数倍的。

ZFS 也存在一些问题：

- ZFS 采用[通用开发与散布许可证](https://zh.wikipedia.org/wiki/CDDL "通用开发与散布许可证")（CDDL）授权，其与 GPL 不兼容，因此 ZFS [不可能](https://sfconservancy.org/blog/2016/feb/25/zfs-and-linux/ "不可能")被纳入 Linux 内核中。然而，这并不妨碍第三方开发者开发并发布原生的 Linux 内核模块，比如 [OpenZFS](https://openzfs.org/ "OpenZFS") (以前被称为 ZFS on Linux)。这使得 OpenZFS 项目必须主动跟上 Linux 内核版本。
- ZFS 使用了自己的 ARC 缓存模型。这带来了更高的缓存命中率，但是这也导致了这部分缓存不在内核的 `cached` 内存内。因此，当内存不够时，这部分缓存无法被及时的释放。因此，推荐使用 ZFS 的系统准备充足的内存。

## 通过 Cockpit 创建 ZFS 池

通过 Cockpit 创建 ZFS 池的过程非常简单，首先确保你已安装了`zfs-dkms`、`zfsutils-linux`包，以及`cockpit-zfs-manager`管理器。确认相关服务已启动并正常运行（安装完成后需重启）。

在`ZFS`栏找到`Create Storage Pool`：

![](image_9rbj6ZwyQg.png)

关于 Virtual Device 选项，Disk 代表直接使用这块硬盘，Mirror 代表 RAID 1 级别。

![](image_8jVVkcub76.png)

根据自己的需求选择 RAID 级别及对应磁盘即可。

Cockpit 的 ZFS 界面会出现图标不显示的问题。45Drives 表示他们已知晓[此问题](https://github.com/45Drives/cockpit-zfs-manager/issues/15)并选择忽视，但提供了一个解决此问题的脚本。

```Bash
#!/usr/bin/env bash

echo Fixing cockpit fonts

mkdir -p /usr/share/cockpit/base1/fonts

curl -sSL https://scripts.45drives.com/cockpit_font_fix/fonts/fontawesome.woff -o /usr/share/cockpit/base1/fonts/fontawesome.woff
curl -sSL https://scripts.45drives.com/cockpit_font_fix/fonts/glyphicons.woff -o /usr/share/cockpit/base1/fonts/glyphicons.woff
curl -sSL https://scripts.45drives.com/cockpit_font_fix/fonts/patternfly.woff -o /usr/share/cockpit/base1/fonts/patternfly.woff

mkdir -p /usr/share/cockpit/static/fonts

curl -sSL https://scripts.45drives.com/cockpit_font_fix/fonts/OpenSans-Semibold-webfont.woff -o /usr/share/cockpit/static/fonts/OpenSans-Semibold-webfont.woff

echo Done
```

<br/>
{% post_link 'NAS-6' '下一章节：SMB & WebDAV' %}