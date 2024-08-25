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

# 进阶：自建 DERP 服务器

建议阅读这篇[来自 Tailscale 官方的文章](https://tailscale.com/blog/how-nat-traversal-works/ "来自 Tailscale 官方的文章")，当中详细介绍了 Tailscale 是如何解决 NAT 穿透问题的。

阅读这篇文章可知，当 STUN 不可用时，Tailscale 会采用 DERP（Detoured Encrypted Routing Protocol，由 Tailscale 创建的协议）来进行中继通信。

DERP 同时也是 Tailscale 连接升级的基础设施。在进行局域网打洞时，Tailscale 会先通过 DERP 服务器来实现两台设备之间的连接，再继续寻找延迟更低的链路（直连方式）等。

由于众所周知的原因，Tailscale 在中国大陆并没有官方的 DERP 服务器。为了解决国外 DERP 服务器高延迟以及被很多用户使用的安全性问题，我们可以采取自建 DERP 服务器的方式。

以下部分参考[这篇文章](https://blog.baldcoder.top/articles/self-hosted-tailscale-derp-relay-server "这篇文章")。

## 准备工作

如果部署 DERP 服务的机器在国内（一般情况下），则需要有一个已备案的域名以及有效的 SSL 证书（无域名的方法请自行寻找）。

根据后续脚本，先创建存放证书的文件夹：

```bash
mkdir /usr/local/cert
```

以阿里云为例，我们可以申请多达 20 张的免费 SSL 证书。确保有对应子域名指向你的云服务器。申请证书的过程在此略过，建议为此子域名单独申请证书，最好不要使用泛域名证书。

**不要使用 CDN 加速。**

在阿里云控制台下载 Nginx 格式的证书，将`.pem`公钥后缀改为`.crt`，再上传至刚才创建的`/usr/local/cert`路径中。

![](image_qR1gQY58s-.png)

## 安装 Tailscale

通过 SSH 连接到云服务器后，运行如下命令：

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

## 安装 Golang

打开 [https://go.dev/doc/install](https://go.dev/doc/install "https://go.dev/doc/install")，查看 Go 的最新版本号。

![](image_ZI5ptpX84u.png)

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

## 安装 derper 服务

建立目录：

```bash
mkdir -p /usr/local/gopath/bin
```

输入以下命令来安装：

```bash
go env -w GOPROXY=https://goproxy.cn,direct
go install tailscale.com/cmd/derper@main
```

建立启动脚本：

```bash
vim /usr/local/gopath/bin/runderper
```

输入以下内容：

```bash
#!/bin/sh
cd /usr/local/gopath/bin
nohup ./derper -hostname <你的域名> -c=derper.conf -a :<服务端口> -http-port -1 -certdir /usr/local/cert -certmode manual -verify-clients -stun > console.log 2>&1 &
echo $! > app.pid
```

> 参数 `-verify-clients` 用来防止别人（知道你的域名后）白嫖你的中继节点，只认服务器上 Tailscale 客户端登录的账号。

建立停止脚本：

```bash
vim /usr/local/gopath/bin/stopderper.sh
```

输入以下内容：

```bash
#!/bin/sh
kill `cat app.pid`
rm -rf app.pid
```

给脚本赋权：

```bash
chmod +x /usr/local/gopath/bin/runderper
chmod +x /usr/local/gopath/bin/stopderper.sh
```

建立服务：

```bash
vim /etc/systemd/system/derper.service
```

输入以下内容：

```text
Description=derper服务
After=network.target
 
[Service]
Type=forking
ExecStart=/usr/local/gopath/bin/runderper
ExecStop=/usr/local/gopath/bin/stopderper.sh
 
[Install]
WantedBy=multi-user.target
```

在云服务器控制台防火墙处，放行你刚刚设置的 derper 服务端口（TCP）与 `3478`端口（UDP，这是 STUN 服务的端口）。

## 启动服务

先启动 Tailscale：

```bash
tailscale up
```

访问提示的网址进行授权即可。

接着启动 derper 服务：

```bash
systemctl enable derper
systemctl start derper
systemctl status derper
```

确认状态为 active(running) 后，访问网址 `https://<你的域名>:56473/`，出现以下页面就是部署成功。

![](image_DcmAHiZT8P.png)

## 添加中继节点

在 Tailscale 网页控制台中的`Access controls`栏的文件中，`ssh`项的上方加入以下内容：

```json
  "derpMap": {
    // OmitDefaultRegions 用来忽略官方的中继节点，一般自建后就看不上官方小水管了
    "OmitDefaultRegions": true,
    "Regions": {
      // 这里的 901 从 900 开始随便取数字
      "901": {
        // RegionID 和上面的相等
        "RegionID": 901,
        // RegionCode 自己取个易于自己名字
        "RegionCode": "阿里云-深圳",
        "Nodes": [
          {
            // Name 保持 1 不动
            "Name":     "1",
            // 这个也和 RegionID 一样
            "RegionID": 901,
            // 域名
            "HostName": "<你的域名>",
            // 端口号
            "DERPPort": 56473,
          },
        ],
      },
    },
  },
```

## 测试延迟

在加入了 Tailscale 的设备中输入：

```bash
tailscale netcheck
```

![](image_smQlaugW-o.png)

显示了中继节点名称，DERP 中继已成功建立。

## Extra：阿里云与 Tailscale

具体说明可以查看[这篇文章](https://blog.baldcoder.top/articles/resolving-the-incompatibility-between-tailscale-and-alibaba-cloud-episode-1/ "这篇文章")。

如果在阿里云服务器上根据上述步骤配置 DERP 服务器后，会发现服务器不能访问外网了。

简单来说，阿里云使用的内网 DNS 服务器 IP 地址为 100.100.2.136 与 100.100.2.138，而 Tailscale 在启动时会设置一条 Drop 100.64.0.0/10 IP 段的防火墙规则，而阿里云内网的一些服务就位于这些 IP 段。

查看 IPTables：

```bash
iptables -L --line-numbers
```

规则 ts-input 如下：

```text
Chain ts-input (1 references)
num  target     prot opt source               destination         
1    ACCEPT     all  --  100.117.68.82        anywhere            
2    RETURN     all  --  100.115.92.0/23      anywhere            
3    DROP       all  --  100.64.0.0/10        anywhere            
4    ACCEPT     all  --  anywhere             anywhere            
5    ACCEPT     udp  --  anywhere             anywhere             udp dpt:41641
```

删除这条 Drop 规则可以解决问题：

```bash
iptables -D ts-input 3 # 输入对应的 num
```

**但请注意：这条规则并不是可有可无的。大部分较新的 Linux 发行版存在**[**漏洞 CVE-2019-14899**](https://seclists.org/oss-sec/2019/q4/122 "漏洞 CVE-2019-14899")**，该漏洞可使攻击者劫持 VPN 链接。使用 iptables 过滤掉一些地址可以作为缓解措施之一。**

**删除这条规则可能会将正在使用基于 WireGuard 的 VPN（例如 Tailscale）的 Linux 主机置于风险之中，请务必仔细权衡利弊。**

如果要恢复，重启服务器或者 Tailscale 即可。
