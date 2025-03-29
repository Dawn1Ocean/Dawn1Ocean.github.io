---
title: 已迁移至新域名！
date: 2024-08-11 14:41:53
categories:
- 博客
tags: 
- DNS
---

# 迁移原因

> 据著名安全网站 Krebsonsecurity 的消息，7 月 16 日，ICANN 致信 .top 域名注册局的所有者江苏邦宁科技有限公司，要求其在 2024 年 8 月中旬之前建立起管理网络钓鱼举报和暂停滥用域名的系统，否则将吊销其许可证。

> ICANN 十分罕见地单独指出负责维护整个顶级域名（TLD）的域名注册管理机构，ICANN 还指责该注册局未能对涉及 .top 域名的网络钓鱼攻击报告做出响应。ICANN 在信中 (PDF) 写道： “根据数周来收集的信息和记录，我们确定 .TOP Registry 并未制定一套流程来及时、全面、合理地调查和处理有关 DNS 滥用的报告。”

> Interisle 咨询集团今天发布的一份新的网络钓鱼报告中，以 .top 结尾的域名占据了显著位置，调查发现在 2023 年 5 月至 2024 年 4 月期间，钓鱼网站占所有新注册 .top 域名的 4%

刚好旧的域名也快到期了，就顺带着注册了一个新域名一起搬过来了，还简短一些。

自 2024 年 8 月 11 日起，本博客已迁移至`deans.fun`域名下。

简要谈谈关于域名等方面配置的一些心得吧。

# SSL 证书

如果你的博客是部署在自己服务器上，由`Nginx`等服务进行托管，那么可以使用 [acme.sh](https://github.com/acmesh-official/acme.sh) 向 [Let's Encrypt](https://letsencrypt.org/zh-cn/) 申请域名 SSL 证书，具体流程就不详述了，也可以使用 [Certbot](https://github.com/certbot/certbot) 这样的自动化工具。

我的博客目前是托管在 GitHub Pages，因此无法自签证书。对于这种情况，我们有两个办法：

1. 在购买域名的平台（如腾讯云、阿里云等）处申请 SSL 证书。服务商一般会提供免费的 SSL 证书（一般期限为 90 天）。
2. 使用第三方平台（如 Cloudflare）托管域名解析服务，让第三方服务商帮你申请 SSL 证书。

以腾讯云为例，在`控制台 - SSL 证书`处可以申请到免费的 SSL 证书。如果你的域名是在腾讯云处购买的，可以选择自动 DNS 验证，较为方便。

![腾讯云申请 SSL 证书](new-domain/image.png)

# 使用 Cloudflare 托管站点

注册 [Cloudflare](https://www.cloudflare-cn.com/enterprise/) 账号，添加你在其他服务商处购买的域名。这里它会要求你将域名的解析服务器更改为 Cloudflare 分配的两个服务器，照做即可。

添加后，等待 Cloudflare 解析验证完毕，域名即被 Cloudflare 托管成功。

进入`SSL/TLS`栏，将 SSL/TLS 加密模式设置为`完全（严格）`，否则无法通过 HTTPS 访问站点。

![SSL/TLS](new-domain/image2.png)

进入`SSL/TLS - 边缘证书`栏，我们可以看到 Cloudflare 已经为域名申请了证书。如果证书处于等待 TXT 验证状态，等待验证完毕即可成功访问站点。

![](new-domain/image3.png)
