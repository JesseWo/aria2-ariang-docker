Aria2 + AriaNg

[English](https://github.com/wahyd4/aria2-ariang-docker/blob/master/README.md) | 简体中文

[![](https://images.microbadger.com/badges/image/wahyd4/aria2-ui.svg)](https://microbadger.com/images/wahyd4/aria2-ui "Get your own image badge on microbadger.com")
[![Docker Pulls](https://img.shields.io/docker/pulls/wahyd4/aria2-ui.svg)](https://hub.docker.com/r/wahyd4/aria2-ui/)
[![Github Build](https://github.com/wahyd4/aria2-ariang-docker/workflows/Docker%20Image%20CI/badge.svg)](https://github.com/wahyd4/aria2-ariang-docker/actions)

本镜像包含 Aria2、AriaNg，主要方便那些用户期望只运行一个镜像就能实现图形化下载文件。（类似离线下载的功能），只使用一个 Docker 镜像也方便用户在群晖NAS 中运行本程序。

- [功能特性](#%e5%8a%9f%e8%83%bd%e7%89%b9%e6%80%a7)
- [推荐使用的docker image tag](#%e6%8e%a8%e8%8d%90%e4%bd%bf%e7%94%a8%e7%9a%84docker-image-tag)
- [安装于运行](#%e5%ae%89%e8%a3%85%e4%ba%8e%e8%bf%90%e8%a1%8c)
  - [快速运行](#%e5%bf%ab%e9%80%9f%e8%bf%90%e8%a1%8c)
  - [开启所有功能](#%e5%bc%80%e5%90%af%e6%89%80%e6%9c%89%e5%8a%9f%e8%83%bd)
  - [使用docker-compose 运行](#%e4%bd%bf%e7%94%a8docker-compose-%e8%bf%90%e8%a1%8c)
  - [支持的 Docker 环境变量](#%e6%94%af%e6%8c%81%e7%9a%84-docker-%e7%8e%af%e5%a2%83%e5%8f%98%e9%87%8f)
  - [支持的 Docker volume 属性](#%e6%94%af%e6%8c%81%e7%9a%84-docker-volume-%e5%b1%9e%e6%80%a7)
- [自动 SSL](#%e8%87%aa%e5%8a%a8-ssl)
- [自行构建镜像](#%e8%87%aa%e8%a1%8c%e6%9e%84%e5%bb%ba%e9%95%9c%e5%83%8f)
- [Docker Hub](#docker-hub)
- [常见问题](#%e5%b8%b8%e8%a7%81%e9%97%ae%e9%a2%98)

AriaNG
![Screenshot](https://github.com/wahyd4/aria2-ariang-x-docker-compose/raw/master/images/ariang.jpg)

~~File Browser~~

## 功能特性

  * Aria2 (SSL 支持)
  * AriaNg 通过 UI 来操作，下载文件
  * 自动 HTTPS （Let's Encrypt）
  * 支持绑定自定义用户ID，可以主机上的非`root`用户，也可以管理下载的文件
  * Basic Auth 用户认证
  * ~~文件管理和视频播放 ([File Browser](https://filebrowser.xyz/)，注意默认情况下，只能访问和管理 `/data` 目录下的文件)~~
  * 支持ARM CPU 架构，因此可以在树莓派中运行，请下载对应的[ARM TAG](https://cloud.docker.com/repository/docker/wahyd4/aria2-ui/tags) 版本, `arm32`或`arm64`

## 推荐使用的docker image tag

* wahyd4/aria2-ui:latest
* wahyd4/aria2-ui:arm32
* wahyd4/aria2-ui:arm64

## 安装于运行

### 快速运行

```shell
  docker run -d --name aria2-ui -p 80:80 wahyd4/aria2-ui
```

* Aria2: <http://yourip>

### 开启所有功能
```bash
  docker run -d --name ariang \
  -p 80:80 \
  -p 443:443 \
  -e PUID=1000 \
  -e PGID=1000 \
  -e ENABLE_AUTH=true \
  -e RPC_SECRET=Hello \
  -e DOMAIN=https://example.com \
  -e ARIA2_SSL=false \
  -e ARIA2_USER=user \
  -e ARIA2_PWD=pwd \
  -e ARIA2_EXTERNAL_PORT=443 \
  -v /yourdata:/data \
  -v /app/a.db:/app/filebrowser.db \
  -v /yoursslkeys/:/app/conf/key \
  -v <the folder of aria2.conf and aria2.session>:/app/conf \
  wahyd4/aria2-ui
```
### 使用docker-compose 运行

如果你不想记住那些命令行，你也可以使用docker-compose来将配置放在`docker-compose.yaml`文件中
```yaml
version: "3.5"
services:
  aria2-ui:
    restart: unless-stopped
    image: wahyd4/aria2-ui:latest
    environment:
      - ENABLE_AUTH=true
      - ARIA2_USER=hello
      - ARIA2_PWD=world
      - DOMAIN=http://toozhao.com
    volumes:
      - ./data:/data
```
然后使用 `docker-compose up -d` 运行即可

### 支持的 Docker 环境变量

  * ENABLE_AUTH 启用 Basic auth(网页简单认证) 用户认证
  * ARIA2_USER Basic Auth 用户认证用户名
  * ARIA2_PWD Basic Auth 密码
  * ARIA2_EXTERNAL_PORT 从外部可以访问到的 Aria2 端口，默认为 HTTP 的`80`
  * PUID 需要绑定主机的Linux用户ID，可以通过`cat /etc/passwd` 查看用户列表， 默认UID 是`1000`
  * PGID 需要绑定的主机的Linux 用户组ID，默认GID 是`1000`
  * RPC_SECRET Aria2 RPC 加密 token
  * DOMAIN 绑定的域名, 当绑定的域名为`HTTPS`时，即为启用`HTTPS`， 例： `DOMAIN=https://toozhao.com`


### 支持的 Docker volume 属性
  * `/data` 用来放置所有下载的文件的目录
  * `/app/conf/key` 用户来放置 Aria2 SSL `certificate`证书和 `key` 文件. `注意`: 证书的名字必须是 `aria2.crt`， Key 文件的名字必须是 `aria2.key`
  * `/app/conf` 该目录下可以放置你的自定义`aria2.conf`配置文件，`aria2.session`，且必须包含这两个文件。第一次使用`aria2.session`时，创建一个空文件即可，该文件会包含aria2当前的下载列表，这样即使容器被销毁也不用担心文件列表丢失了。你也可以直接拷贝当前项目下`conf`目录中的两个文件并使用。

## 自动 SSL

请在绑定域名前，设置`DNS`的一条`A`记录，将运行docker的主机IP绑定到该域名。然后你仅仅需要在运行时添加`e`设置即可。

```bash
docker run -d --name aria2-ui -p 80:80 -p 443:443 -e DOMAIN=https://toozhao.com wahyd4/aria2-ui
```

## 自行构建镜像

```
docker build -t aria2-ui .
```

## Docker Hub


## 常见问题
  1. 当你以非其他`80` 端口或以启用了HTTPS`443`端口运行程序时，会出现`Aria2 状态 未连接`的错误，这是因为在最新版本里面，我们去掉aria2的独立6800端口，转而使用和网站同一个端口。你可以设置`ARIA2_EXTERNAL_PORT`后重建你的容器。
  2. 下载的BT或者磁力完全没有速度怎么办？ 建议先下载一个热门的BT种子文件，而不是磁力链接。这样可以帮助缓存DHT文件，渐渐地，速度就会起来了。比如试试下载树莓派操作系统的BT种子？[前往下载](https://www.raspberrypi.org/downloads/raspbian/)
  3. 如果你遇到了和 `setcap` 相关的错误，很大程度说明你说运行的Linux不支持使用非`root`用户来运行本Docker 镜像，因此请显式地设置环境变量`PUID`  `PGID` 为 `0` ，也就是使用`root` 来运行
