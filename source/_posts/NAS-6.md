---
title: NAS（6）—— SMB & WebDAV
date: 2024-08-26 02:43:35
categories:
- NAS
tags:
- SMB
- WebDAV
---

{% post_link 'NAS-Router-0' '回到目录' %}
<br/>

# 概述

经过前面几个模块的配置，我们已经能够在 NAS 上存放我们的数据，并且通过组建 RAID 与接入 UPS 等方式加强了数据存储的安全性。在这个部分，我们需要通过配置来实现通过各种设备来访问 NAS 中存储的数据，并且进一步加强数据存储的安全性。

## SMB / Samba

SMB（Server Message Block）可用于在计算机间共享文件、打印机、串口等，电脑上的网上邻居就是靠它实现的。通过 SMB 协议，客户端应用程序可以在各种网络环境下读、写服务器上的文件，以及对服务器程序提出服务请求。此外通过 SMB 协议，应用程序可以访问远程服务器端的文件、以及打印机等资源。

Samba 能让 Linux 服务器实现文件服务器、身份授权和认证、名称解析和打印服务等功能。

Samba 也是 SMB 协议的实现，它允许 Windows 客户访问 Linux 系统上的目录、打印机和文件（就像访问 Windows 服务器时一样）。重要的是，Samba 可以将 Linux 服务器构建成一个域控制器。这样一来，就可以直接使用 Windows 域中的用户凭据，免去手动在 Linux 服务器上重新创建的麻烦。

相比于 NFS，SMB/Samba 对 Windows 系统有着更好的支持，兼容性更优。

## 异地容灾

在数据存储与容灾方面，有一条重要的原则，3-2-1 原则：

- **3**：同一数据至少保存 **3** 份，包括原数据；
- **2**：保存到 **2** 种以上存储介质上，如机械硬盘/固态硬盘；
- **1**：至少有 **1** 份异地备份，防止本地发生灾难时数据完全丢失，如选择保存到云服务器上。

# SMB 配置

## NAS 端 —— 45Drive's File Sharing

首先确保安装了 45Drive's 库的 File Sharing 包。

在 Cockpit 控制台中`File Sharing - Shares`处添加 SMB 共享：

![](image_pW-C22fE2_.png)

按照下图进行配置即可。

![](image1.png)

需要共享多个文件夹，就在 Shares 处创建多个共享配置。

为了让其他设备具有写入权限，请配置共享文件夹的权限为 744。

```Bash
sudo chmod 744 -R /dir
```

在配置了 IPv6 后，我们的 SMB 服务相当于**暴露在学校大内网中**，因此更应做好访问控制。

Samba 有自己的用户系统，但与系统用户名一致的用户拥有相同的访问权限。

我们创建一个与当前用户一致的 Samba 用户：

```Bash
sudo smbpasswd -a <你正在使用的用户名>
```

根据提示设置好密码后（请设置高强度密码），修改共享文件夹及其文件的拥有者以匹配 Samba 用户：

```Bash
sudo chown <你正在使用的用户名>:<所在用户组> -R /dir
```

在 Cockpit 中的防火墙放行 TCP 139 与 445 端口的请求。

## 其他设备端

对于 Windows 系统，确认其已连接到 NAS 的局域网中。打开运行，输入`\\<NAS IP>`，回车：

![](image_pkBmWbVj-b.png)

访问对应文件夹，确认能够正常读写其中的文件。

![](image_Zucjtsg4JD.png)

可以将文件夹映射到驱动器，便于访问。右键文件夹，选择映射网络驱动器：

![](image_oA0kP8OPt4.png)

指定驱动器号，即可在此电脑中看到对应文件夹。

![](image_uca3CXn-Gf.png)

# WebDAV

随着云计算和移动互联网的快速发展，远程访问、编辑和共享文件的需求日益增强。为了满足这一需求，WebDAV（Web-based Distributed Authoring and Versioning）应运而生，成为了一种基于 HTTP 协议的通信协议，扩展了 HTTP 1.1，提供了应用程序直接对 Web 服务器进行读写操作的能力。

## 自建 WebDAV 服务

如果你恰好有一台云服务器，并且带宽足够，连接稳定，就可以在上面部署 WebDAV 服务。

我们采用 Go 语言实现的 [Standalone WebDAV](https://github.com/hacdias/webdav "Standalone WebDAV") 服务。

### 安装 Golang

打开 [https://go.dev/doc/install](https://go.dev/doc/install "https://go.dev/doc/install")，查看 Go 的最新版本号。

![](image_K3ubkRc5GT.png)

```bash
wget https://go.dev/dl/go<最新版本号>.linux-amd64.tar.gz
tar -C /usr/local -xzf $HOME/go<最新版本号>.linux-amd64.tar.gz
```

编辑`/etc/profile`文件：

```bash
vim /etc/profile
```

在文件末尾添加以下内容：

```bash
export GOROOT=/usr/local/go
export GOPATH=/usr/local/gopath
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOROOT/bin
export PATH=$PATH:$GOPATH/bin
```

输入如下命令：

```bash
source /etc/profile
```

再输入以下命令检查 Go 是否安装成功：

```bash
go version
```

### 安装 WebDAV

输入以下指令：

```bash
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct # 换源

go install github.com/hacdias/webdav/v5@latest
```

### 配置 WebDAV

创建共享文件夹，最好不要在`/root`目录下，避免出现权限问题。

```bash
mkdir /home/webdav
```

创建配置文件：

```bash
vim webdav.yaml
```

输入以下内容，根据具体情况进行修改：

```yaml
# Server related settings
address: 0.0.0.0
port: 11111  # 端口自定，记得在防火墙当中放行
auth: true
tls: false  # 是否配置 HTTPS，若更改为 true，请对应更改 cert 以及 key 地址
cert: cert.pem
key: key.pem
# Default user settings (will be merged)
scope: .
modify: true
rules: []
users:  # 建议以 user 的形式单独配置权限以及文件夹位置
  - username: test
    password: "{bcrypt}$2a$12$of2hfif89pyh293of89o4gg20pgfy22rfuikhsdfio2i.f22uj8h3"
    # 密码最好不要以明文存储，此服务支持使用 bcrypt 进行哈希
    directory: /home/webdav # 指定文件夹
    permissions: CRUD # C - Create; R - Read; U - Update; D - Delete，即增删改查，按需添加
```

可以使用[这个网站](https://bcrypt-generator.com/ "这个网站")在线生成 bcrypt 加密。

### 运行 WebDAV

尝试开启服务：

```bash
webdav --config webdav.yaml
```

确认没问题后，可以使用Systemd来控制开机自启和后台运行。

编辑`/lib/systemd/system/webdav.service`文件，写入以下内容：

```text
[Unit]
Description=WebDAV server
After=network.target
[Service]
Type=simple
User=root
ExecStart=/usr/local/gopath/bin/webdav --config /root/webdav.yaml
# 使用 which 命令查看 webdav 位置，需要完整路径；配置文件位置根据实际情况进行修改
[Install]
WantedBy=multi-user.target
```

在防火墙中放行对应端口。启动服务：

```bash
systemctl enable webdav
systemctl start webdav
```

## WebDAV 服务商

目前国内支持 WebDAV 的云盘服务商只有坚果云。不过，阿里云盘通过开放平台接口可以实现 WebDAV 服务。这里是[项目地址](https://github.com/messense/aliyundrive-webdav "项目地址")。

坚果云对于免费用户每月提供 1GB 的上传流量以及 3GB 的下载流量。可以在第三方应用管理处创建应用密码。

![](image_pFn_muetp9.png)

## 挂载 WebDAV 为本地磁盘

以下部分来自于[这篇文章](https://www.dujin.org/17037.html "这篇文章")。

### 安装 [davfs2](https://www.dujin.org/tag/davfs2 "davfs2") 工具

```bash
sudo apt install davfs2
```

### 挂载 WebDAV 磁盘

先在 Linux 系统上创建要挂载的文件目录，名称自定：

```bash
mkdir /Aliyun
```

挂载远端 WebDAV 至本地的`/Aliyun`目录下：

```bash
mount -t davfs http(s)://<服务器地址/域名>:<端口>/ /Aliyun # 请根据实际情况配置
```

回车后输入自己的用户名和密码，完成。可以输入`df -h`指令来查看是否成功。

![](image_QBMxoN8z84.png)

### 挂载坚果云 WebDAV

如果使用上述方法挂载坚果云 WebDAV，会出现如下报错信息：

```纯文本
/sbin/mount.davfs: mounting failed; the server does not support WebDAV
```

出现此情况的原因是，mount.davfs 命令在挂载时和 WebDAV 服务器建立连接，并通过 libneon 库的 `ne_options()` 函数发送 HTTP OPTIONS 请求获取 WebDAV 服务器能力，虽然返回成功但判断坚果云 WebDAV 服务器不支持 Class 1 WebDAV，因此直接报错挂载失败。

解决办法：修改`/etc/davfs2/davfs2.conf`：

```bash
sudo vim /etc/davfs2/davfs2.conf
```

将 ignore\_dav\_header 取消注释，并将 0 改为 1，保存并退出。

再次挂载，成功。

### 开机自动挂载 WebDAV

#### 启用 davfs2 用户锁

编辑`/etc/davfs2/davfs2.conf`文件：

```bash
sudo vim /etc/davfs2/davfs2.conf
```

将 use\_locks 取消注释，并将 1 改为 0，保存并退出。

#### 记住 WebDAV 账号密码

编辑`/etc/davfs2/secrets`文件：

```bash
sudo vim /etc/davfs2/secrets
```

在末尾增加一行内容：

```纯文本
http(s)://<服务器地址/域名>:<端口>/ <username> <password>
```

这里的 username 和 password 请更改为 webdav.yaml 中对应的用户名和密码。

#### 开机执行挂载

编辑`/etc/rc.local`文件：

```bash
sudo vim /etc/rc.local
```

文件末尾加上挂载指令，与上文一致。

```bash
mount -t davfs http(s)://<服务器地址/域名>:<端口>/ /Aliyun # 请根据实际情况配置
```

保存退出即可。

<br/>
{% post_link 'NAS-7' '下一章节：Docker 服务' %}