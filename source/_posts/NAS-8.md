---
title: NAS（8）—— Nginx & HTTPS
date: 2024-08-26 02:43:43
categories:
- HTTPS
- Nginx
tags:
- SSL 证书
---

{% post_link 'NAS-Router-0' '回到目录' %}
<br/>

# 概述

Nginx 是 lgor Sysoev 为俄罗斯访问量第二的 rambler.ru 站点设计开发的。从 2004 年发布至今，凭借开源的力量，已经接近成熟与完善。

Nginx 功能丰富，可作为 HTTP 服务器，也可作为反向代理服务器，邮件服务器。支持 FastCGI、SSL、Virtual Host、URL Rewrite、Gzip 等功能。并且支持很多第三方的模块扩展。

![](NAS-8/image_723xGBuc8B.png)

## 反向代理

正向代理（Proxy）方式是指内部网络上的服务器通过代理服务器来访问 Internet 上的服务器，并把响应结果返回给内部网络服务器。

反向代理（Reverse Proxy）方式是指以代理服务器来接受 Internet 上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给 Internet 上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

通过反向代理功能，我们可以隐藏内部服务器的地址（暴露的是代理服务器），对于客户端而言，此代理是无感知的。通过 Nginx，我们可以将发送给不同子域名的请求转发到同一台机器的各个端口；我们还可以更进一步，将客户端的请求转发到不同的内部服务器，这样就实现了负载均衡。

![](NAS-8/image_XGn5XU8be0.png)

## SSL 证书

我们先来看看 HTTPS 的握手过程：

HTTPS 在传输数据之前需要客户端（浏览器）与服务端（网站）之间进行一次握手，在握手过程中将确立双方加密传输数据的密码信息。TLS/SSL 协议不仅仅是一套加密传输的协议，更是一件经过艺术家精心设计的艺术品，TLS/SSL 中使用了非对称加密，对称加密以及 HASH 算法。握手过程的具体描述如下：

![](NAS-8/image_8A5Ssr-fYl.png)

1. 浏览器将自己支持的一套加密规则发送给网站。
2. 网站从中选出一组加密算法与 HASH 算法，并将自己的身份信息以证书的形式发回给浏览器。证书里面包含了网站地址，加密公钥，以及证书的颁发机构等信息。
3. 浏览器获得网站证书之后浏览器要做以下工作：
   1. 验证证书的合法性（颁发证书的机构是否合法，证书中包含的网站地址是否与正在访问的地址一致等），如果证书受信任，则浏览器栏里面会显示一个小锁头，否则会给出证书不受信的提示。
   2. 如果证书受信任，或者是用户接受了不受信的证书，浏览器会生成一串随机数的 Key，并用证书中提供的公钥加密。
   3. 使用约定好的 HASH 算法计算握手消息，并使用生成的随机数 Key 对消息进行加密，最后将之前生成的所有信息发送给网站。
4. 网站接收浏览器发来的数据之后要做以下的操作：
   1. 使用自己的私钥将信息解密取出 Key，使用 Key 解密浏览器发来的握手消息，并验证 HASH 是否与浏览器发来的一致。
   2. 使用密码加密一段握手消息，发送给浏览器。
5. 浏览器解密并计算握手消息的 HASH，如果与服务端发来的 HASH 一致，此时握手过程结束，之后所有的通信数据将由之前浏览器生成的随机密码并利用对称加密算法进行加密。

了解了 HTTPS 握手过程后，你可能会产生如下疑问：如果我在给客户端发送证书的时候进行伪造，选取一对有效的公私钥，伪装成服务器，会产生安全问题吗？这个时候就轮到 CA 机构出马了。

CA（Certificate Authority，证书授权）机构是负责发放和管理数字证书的权威机构。CA 机构在网络安全领域扮演着重要的角色，它们通过验证和颁发数字证书，为网络通信提供身份认证的有效凭据，确保数据的安全性和可信度。

![](NAS-8/image_ikOKfaYjER.png)

在申请 CA 证书时，服务器端会生成一对有效的公私钥，并将公钥、申请者信息、域名等内容发送到 CA。CA 接收后，通过 HASH 算法生成证书摘要，再用 CA 的私钥对其进行加密，生成签名，最后返回证书。

这样一来，客户端就可以通过签名和公钥验证服务器返回的公钥是否由 CA 机构签发，也就确保了安全性。

但获取 CA 证书是需要成本的。或者在某些情况下，我们正在进行本地开发，需要获得`localhost`等主机名的证书，而这样的私有地址不是被某个人唯一拥有的，因此 CA 机构不会提供证书。在这种情况下，我们可能需要自己生成证书。

自签名证书是由不受信的 CA 机构颁发的数字证书，也就是自己签发的证书。与受信任的 CA 签发的传统数字证书不同，自签名证书是由一些公司或软件开发商创建、颁发和签名的。虽然自签名证书使用的是与 X.509 证书相同的加密密钥对架构，但是却缺少受信任第三方（如 Sectigo）的验证。在颁发过程中缺乏独立验证会产生额外的风险，这就是为什么对于面向公众的网站和应用程序来说，自签名证书是不安全的。

# Nginx 安装与配置

```bash
sudo apt install nginx
```

接下来为 Web 应用创建一个新的 Nginx 配置文件：

```bash
sudo vim /etc/nginx/sites-available/app.conf
```

将以下内容添加到配置文件中：

```纯文本
server {
    listen 80; # 替换为你需要的端口
    # listen 443 ssl; # 若要启用 HTTPS，在端口后添加 ssl
    # ssl_certificate /path/to/cert # 这里是证书的公钥
    # ssl_certificate_key /path/to/key # 这里是证书的私钥
    server_name app.example.com; # 替换为你自己的域名

    location / { # 这里的 “/” 意为网站根目录，请根据实际情况调整
        proxy_pass http://localhost:8080; # 这里是要反向代理转发到的端口
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

有关于域名 SSL 证书的申请，这里就不详述了。我们可以采用 [certbot](https://github.com/certbot/certbot "certbot") 向 Let's Encrypt 发送请求。

创建软链接以启用站点：

```bash
sudo ln -s /etc/nginx/sites-available/app.conf /etc/nginx/sites-enabled/
```

测试 Nginx 配置：

```bash
sudo nginx -t
```

如果配置有效，请重新加载 Nginx：

```bash
sudo systemctl reload nginx
```

记得放行防火墙对应端口。

# 局域网中的 HTTPS

正如前文所提到过的，在 NAS 环境下，我们的服务都是在局域网中运行的，因此无法使用 CA 机构签发的证书。如果我们部署的服务有要求必须使用 HTTPS，就需要我们自签名 SSL 证书。

以下部分主要参考[这篇文章](https://sanyers.github.io/blog/web/webrtc/%E6%90%AD%E5%BB%BA%E5%B1%80%E5%9F%9F%E7%BD%91https.html "这篇文章")。

## 自签名SSL证书

我们直接在 GitHub [对应仓库](https://github.com/FiloSottile/mkcert "对应仓库")的 Release 处下载软件，放入待生成证书的机器中。

先生成根证书：

```bash
mkcert -install

mkcert -CAROOT # 查看根证书路径
```

会自动生成 `rootCA.pem`与`rootCA-key.pem`两个文件。

再生成自签证书：

```bash
mkcert localhost 127.0.0.1 ::1 <待申请证书的局域网 IP 地址，可多个>
```

返回结果如下：

```纯文本
Using the local CA at "C:\Users\sanye\AppData\Local\mkcert" ✨

Created a new certificate valid for the following names 📜
 - "localhost"
 - "127.0.0.1"
 - "::1"
 - "192.168.xx.xx"

The certificate is at "./localhost+2.pem" and the key at "./localhost+2-key.pem"
```

可以将证书重命名为自己需要的名字比如 `localhost_key.pem` `localhost_cert.pem`。

## 局域网内其他设备安装自签证书

将刚才生成的根证书 `rootCA.pem` 拷贝一个副本，并命名为 `rootCA.crt` (因为 Windows 并不识别 .pem 扩展名，并且 Ubuntu 也不会将 .pem 扩展名作为 CA 证书文件对待)，将 `rootCA.crt` 文件复制给局域网内的其他用户，手工导入。

### Windows

双击这个证书文件，在常规选项卡，点击安装证书证书，在导入向导中将证书导入 `受信任的根证书颁发机构`:

![](NAS-8/image_ovVeUpRqIN.png)

![](NAS-8/image_2kWD-I5COb.png)

![](NAS-8/image_nUjQg9Qgb5.png)

Windows 命令行执行：

```bash
certutil -addstore -f "ROOT" selfsigned.crt
```

### Mac OS

```bash
sudo security add-trusted-cert -d -r trustRoot -k ~/Library/Keychains/login.keychain "selfsigned.crt"
```

如果想将自签名证书在全系统级别受信任，需要将上面的证书目标路径从 `~/Library/Keychains/login.keychain`替换成`/Library/Keychains/login.keychain`。

注意：在 Mac 下，浏览器和 curl 会自动信任新增的自签名证书，但许多编程语言并没有默认集成 keychain，因此不能自动通过自签名证书。

### Ubuntu

将证书文件（必须是 .crt 后缀）放入 `/usr/local/share/ca-certificates/`，然后执行 `sudo update-ca-certificates`。

```bash
sudo apt install ca-certificates -y
cp selfsigned.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates --verbose
```

<br/>
{% post_link 'NAS-9' '下一章节：Homepage 导航页' %}