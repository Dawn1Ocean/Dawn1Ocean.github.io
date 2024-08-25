---
title: NAS（12）—— Immich 照片管理
date: 2024-08-26 02:44:00
categories:
- Immich
---

{% post_link 'NAS-Router-0' '回到目录' %}
<br/>

# 概述

Immich 是一个直接从 iPhone、Android 手机上备份照片与视频的开源解决方案，通过部署在自己的电脑、NAS、服务器中，使用 App 进行备份。界面酷似 Google Photos，支持多用户、照片和相册分享、好友上传、地理位置、机器学习识别事件等功能。是居家备份照片的好帮手。

![](image_MnJL4-IsHE.png)

# 通过 Docker 部署

使用 Docker Compose：

进入部署文件夹，下载`docker-compose.yml`文件：

```bash
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
```

下载`.env`文件：

```bash
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
```

（可选）下载`hwaccel.transcoding.yml`文件：

```bash
wget -O hwaccel.transcoding.yml https://github.com/immich-app/immich/releases/latest/download/hwaccel.transcoding.yml
```

（可选）下载`hwaccel.ml.yml`文件：

```bash
wget -O hwaccel.ml.yml https://github.com/immich-app/immich/releases/latest/download/hwaccel.ml.yml
```

根据实际情况更改`.env`文件：

```text
# You can find documentation for all the supported env variables at https://immich.app/docs/install/environment-variables

# The location where your uploaded files are stored
UPLOAD_LOCATION=./library
# The location where your database files are stored
DB_DATA_LOCATION=./postgres

# To set a timezone, uncomment the next line and change Etc/UTC to a TZ identifier from this list: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List
# TZ=Etc/UTC

# The Immich version to use. You can pin this to a specific version like "v1.71.0"
IMMICH_VERSION=release

# Connection secret for postgres. You should change it to a random password
# Please use only the characters `A-Za-z0-9`, without special characters or spaces
DB_PASSWORD=postgres

# The values below this line do not need to be changed
###################################################################################
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
```

输入如下命令启动 Immich：

```bash
sudo docker compose up -d
```

Immich 镜像拉取建议使用科学上网服务。

若要更新 Immich，请输入以下命令：

```bash
sudo docker compose pull && sudo docker compose up -d
```

# 配置

## NAS 端

### 机器学习模型

Im­mich 是支持使用描述文字搜索图片的，但默认的机器学习模型是 `ViT-B-32__openai`，这个模型并不支持中文。好在官方提供的[模型](https://huggingface.co/immich-app "模型")中，有一个 [XLM-Roberta-Large-Vit-B-16Plus](https://huggingface.co/immich-app/XLM-Roberta-Large-Vit-B-16Plus "XLM-Roberta-Large-Vit-B-16Plus") 可以使用中文搜索。

![](image_DPdSNcC-E7.png)

当然也可以使用 XLM-Roberta-Large-Vit-B-32，但空间占用与处理时的 CPU 占用都会更高。

![](d823bc555cbf782927db1a63a9f64cac_Uo2RU3n_Hd.png)

如果 NAS 能够使用代理，我们直接前往 Immich 的`管理 - 设置 - 机器学习设置 - 智能搜索 - CLIP 模型`处修改 CLIP 模型名称。

![](image_oSvvt28mu-.png)

修改完成后，点击左侧`任务`一栏，找到`智能搜索`一项，点击`全部`。Immich 会先下载对应的模型，再对所有的图片进行机器学习处理。

![](image_jvGDWNbSi6.png)

我们可以在 Cockpit 的终端处查看 Docker 日志，确认模型下载与运行情况：

```bash
sudo docker logs immich_machine_learning
```

![](image_pTOftXM_P9.png)

可以看到，我们已经成功下载了对应的 CLIP 模型。

如果 NAS 不能使用代理，我们需要先在其他设备上下载对应的 CLIP 模型，再放入 Immich 机器学习的对应文件夹中。具体步骤此处不详述，可参见[这篇文章](https://www.bilibili.com/read/cv33865669/ "这篇文章")。

处理完成后，我们尝试使用中文进行语义搜索，结果非常完美。

![](image_RMZlFLME53.png)

## 客户端

手机端应用的配置非常简洁明了，这里就不详述了，主要用到的功能为备份功能。

![](c77e849951216e2d9c65f0ff0f5c8618_1jAwgdP7X0.png)

剩下的功能，就留给各位去探索吧！
