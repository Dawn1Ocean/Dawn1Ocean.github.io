---
title: NAS（2）—— 基于原生 Debian 搭建系统
date: 2024-08-26 02:34:31
categories:
- NAS
- Debian
---

{% post_link 'NAS-Router-0' '回到目录' %}
<br/>

# 选择 Debian 的原因

选择 Debian 的原因有很多，包括但不限于：

- 完全自定义的配置
- 海量、可自由选择安装的软件包
- 喜欢折腾（不喜欢折腾也有[脚本](https://github.com/kekylin/Debian-HomeNAS "脚本")）
- ......

# Debian 最小化安装

以下部分主要来自于[这篇教程](https://docs.qq.com/doc/p/596d660489283e353db059185f2533d48e12a502 "这篇教程")。

## 下载安装包

进入官网：[https://www.debian.org/](https://www.debian.org/ "https://www.debian.org/")，点击其他下载链接

![](image_W2aermnKZR.png)

选择较庞大的完整安装映像，64 位 PC DVD-1 iso

![](image_2ahsxWNrjL.png)

因Debian系统在安装过程中需要联网更新安全源，国内访问速度有时候异常缓慢，动辄几个小时。故国内用户建议按上面教程下载完整系统镜像包，这样将避免此问题发生，完成系统安装只需要四分钟左右，大大提高系统安装效率。

推荐使用Ventoy工具辅助安装，此工具可以将U盘制作为启动盘，制作完成后只需将镜像拷入即可。官网：[https://www.ventoy.net/en/download.html](https://www.ventoy.net/en/download.html "https://www.ventoy.net/en/download.html")

## 选择安装方式（图形界面安装 Graphical install）

![](nhGaJTyjJbvrkQaypyvD9_Zk1xGLC1uQ.png)

## 设置系统默认语言（English，避免 bug）

![](2pNKRaXbfheDkeca4kEvUq_KZBmxqrRs5.png)

## 设置系统时区（other-Asia-China）

![](u1JCgNihDczYjFynaT26SM_RDOV_9wo0E.png)

## 设置地区（Asia-China）

![](qhJHP1twq3cpNMJoGwz53K_g3VpfnU-p1.png)

![](vaThjWE8nXf98hzG6xenEU_tNGh1GOFlv.png)

## 设置区域语言（United States en\_US.UTF-8）

![](2fcXVPnDBk25jh4Jxfe6Z_4Onzin6bgK.png)

## 设置键盘映射（American English）

![](pSubfyeCxsjBnPPhv1um5p_a9SpFGbKjX.png)

## 设置主机名

![](f2vxQoJauuBacnvDFqPq26_WWu-2HvDtc.png)

## 设置域名（留空）

![](kFQpy8X4jJ68qws9eYoKFg_YI7zCTMPPH.png)

## 设置管理员密码（root 用户）

![](pARYctmcwhNtA5iVBFmsRp_Tp_6hhMpwE.png)

## 创建新普通用户

![](biCpqTMgGwtkRbTcZfMsj7_iZM_2fmDUp.png)

## 新用户账户名称

登陆名称，建议和前面保持一致，避免出现混乱。

![](hxsbbDNW9FddVqbL41kyM6_oY75nHXW_x.png)

## 设置新用户密码

![](fBSZwEbXMsQuphGWi4d132_7bV7hVQ2k7.png)

## 系统盘磁盘分区

这里的选择依据具体情况而定，一般选择 Guided-use entire disk 使用整个磁盘。

![](gtp5wgg1v97WkBdSgo1w8N_0F7V6XGg0y.png)

## 选择系统磁盘

![](5C74eeovT8zHw5esDW4FaR_u85S29DLqC.png)

## 选择分区方案

这里的选择依据具体情况而定，建议选择 All files in one partition 将所有文件放在同一个分区。

![](53sUfHM8sQnEPfrm9WFcXi_2PSs6pQQPp.png)

## 完成分区操作并将修改写入磁盘

![](i53SQFSk3YMRBpqNMyujkM_IHHndcLPUf.png)

## 再次确认

![](posFGNBqFmWEKp8xeSWdt4_LqAiBSWLQD.png)

## 是否需要扫描额外安装介质（No)

![](rF2vymPiBdWsN9iDP4znRZ_LE7JgqQWFg.png)

## 是否需要使用网络镜像站点（No）

![](hm5cPTB5JjSLhqE586ZpMz_2xPz3Wv5sx.png)

## 是否要参加软件包流行度调查（No）

![](j8PX2LD7X3Wr5M7dXJiEU9_NhkSEYpbNE.png)

## 选择要安装的软件（建议只选最后两项）

![](5wrHHMDBoFK5Aq7HjeQdmj_A7DPcXkjUe.png)

## 系统正在安装中

![](2WFCy4UhxZaoPjFpU1u63P__x0HBSDvUh.png)

## 完成安装

![](kke6qD2EHZ62cQtRK1bQgg_uKNUVNXiW2.png)

点击 Continue 重启。

安装完成后用 root 账户登陆系统，输入命令查询 IP 地址。（去路由器管理后台看也可）

```bash
ip addr
```

![](bwgXzbbvuJGLeDZxRhybKn_M_oRZKrENr.png)

<br/>
{% post_link 'NAS-3' '下一章节：Cockpit & 系统调优' %}