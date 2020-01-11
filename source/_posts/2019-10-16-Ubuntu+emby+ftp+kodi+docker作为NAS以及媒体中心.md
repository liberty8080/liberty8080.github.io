---
layout:     post   				    # 使用的布局（不需要改）
title:      Ubuntu+emby+ftp+kodi+docker作为NAS以及媒体中心				# 标题 
subtitle:   Ubuntu+emby+ftp+kodi+docker作为NAS以及媒体中心 #副标题
date:       2019-07-26 				# 时间
author:     liberty						# 作者
header-img: img/post-bg-2015.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - Linux
    - 运维
---


## Ubuntu+emby+ftp+kodi+docker作为NAS以及媒体中心

新搭建了一台Ubuntu服务器，用来作NAS和媒体中心，基本配置如下：  

* 系统环境基本配置
* 在Docker下运行vsftpd服务并配置ssl证书
* nfs
* samba
* 使用docker搭建owncloud
* 使用Docker搭建L2TP服务器
* Calibre书库
* Docker下运行mysql

### 一、系统基本环境配置

如下[链接](_posts/2019-11-12-Ubuntu系统环境基本配置.md)

### 二、[Docker下搭建vsftpd作为文件共享](https://hub.docker.com/r/fauria/vsftpd)

```shell
docker run -d -v /home/jacob/Share/public:/home/vsftpd/illusion \
-v /home/jacob/Share/private:/home/vsftpd/realistic \
-p 20:20 -p 21:21 -p 21100-21110:21100-21110\
-e FTP_USER=illusion -e FTP_PASS=ez9u92mysvjb4q48 \
-e PASV_ADDRESS=127.0.0.1 -e PASV_MIN_PORT=21100 -e PASV_MAX_PORT=21110 \
--name vsftpd --restart=always fauria/vsftpd
```

添加用户

```shell
docker exec -i -t vsftpd bash
mkdir /home/vsftpd/myuser
echo -e "user\" >> /etc/vsftpd/virtual_users.txt
/usr/bin/db_load -T -t hash -f /etc/vsftpd/virtual_users.txt /etc/vsftpd/virtual_users.db
exit
docker restart vsftpd
```

### 三、使用docker搭建seafile

之前尝试使用了owncloud，由于ssl证书的格式转换始终有问题，改用seafile，配置成功。
seafile.sh

```bash
docker run -d --name seafile \
  -e SEAFILE_SERVER_HOSTNAME=seafile.liberty3306.top \
  -e SEAFILE_ADMIN_EMAIL=youremail@whatever.com \
  -e SEAFILE_ADMIN_PASSWORD=password \
  -v /opt/seafile-data:/shared \
  -p 8001:443 \
  -p 8000:80 \
  -e SEAFILE_SERVER_LETSENCRYPT=true \
  -e SEAFILE_SERVER_HOSTNAME=your.host.name \
  seafileltd/seafile:latest
```

### 四、使用dockers搭建l2tp服务

```bash
docker run \
--name l2tp-ipsec-vpn-server \
--env-file /home/jacob/.docker/l2tp-ipsec-vpn-server/config/vpn.env \
-p 500:500/udp \
-p 4500:4500/udp \
-v /lib/modules:/lib/modules:ro \
-d --privileged \
fcojean/l2tp-ipsec-vpn-server
```

### 五、搭建calibre书库

```bash
 docker create  \
    --name=calibre-web  \
    -p 10002:8083  \
    -p 10003:8080  \
    -v /home/jacob/.docker/calibre-web/config/:/config  \
    -v /home/jacob/Share/public/Calibre/:/library  \
    -e USER=用户名  \
    -e PASSWORD=用户密码 \
    --restart unless-stopped  \
    johngong/calibre-web:latest
```

配置ssl，将fullchain.crt改为fullchain.pem,private.pem改为private.key  
[配置推送邮箱](https://www.jianshu.com/p/085c22e3d7db)

### 六、配置aria2

```bash
sudo apt-get install aria2
```

配置文件如下：

```text
## 文件保存相关 ##

# 文件的保存路径(可使用绝对路径或相对路径), 默认: 当前启动位置
dir=/home/jacob/Downloads/aria2/

# 启用磁盘缓存, 0为禁用缓存, 需1.16以上版本, 默认:16M
disk-cache=32M
disk-cache=32M
# 文件预分配方式, 能有效降低磁盘碎片, 默认:prealloc
# 预分配所需时间: none < falloc ? trunc < prealloc
# falloc和trunc则需要文件系统和内核支持
# NTFS建议使用falloc, EXT3/4建议trunc, MAC 下需要注释此项
file-allocation=trunc
# 断点续传
continue=true

## 下载连接相关 ##

# 最大同时下载任务数, 运行时可修改, 默认:5
max-concurrent-downloads=10
# 同一服务器连接数, 添加时可指定, 默认:1
max-connection-per-server=10
# 最小文件分片大小, 添加时可指定, 取值范围1M -1024M, 默认:20M
# 假定size=10M, 文件为20MiB 则使用两个来源下载; 文件为15MiB 则使用一个来源下载
min-split-size=10M
# 单个任务最大线程数, 添加时可指定, 默认:5
split=15
# 整体下载速度限制, 运行时可修改, 默认:0
max-overall-download-limit=0
# 单个任务下载速度限制, 默认:0
max-download-limit=0
# 整体上传速度限制, 运行时可修改, 默认:0
max-overall-upload-limit=0
# 单个任务上传速度限制, 默认:0
max-upload-limit=0
# 禁用IPv6, 默认:false
disable-ipv6=true

## 进度保存相关 ##

# 从会话文件中读取下载任务
input-file=/etc/aria2/aria2.session
# 在Aria2退出时保存`错误/未完成`的下载任务到会话文件
save-session=/etc/aria2/aria2.session
# 定时保存会话, 0为退出时才保存, 需1.16.1以上版本, 默认:0
save-session-interval=60

## RPC相关设置 ##

enable-rpc=true
pause=false
rpc-allow-origin-all=true
rpc-listen-all=true
rpc-save-upload-metadata=true
rpc-secure=true

# 启用RPC, 默认:false
#enable-rpc=true
# 允许所有来源, 默认:false
#rpc-allow-origin-all=true
# 允许非外部访问, 默认:false
#rpc-listen-all=true
# 事件轮询方式, 取值:[epoll, kqueue, port, poll, select], 不同系统默认值不同
#event-poll=select
# RPC监听端口, 端口被占用时可以修改, 默认:6800
rpc-listen-port=6800
# 设置的RPC授权令牌, v1.18.4新增功能, 取代 --rpc-user 和 --rpc-passwd 选项
rpc-secret=<skyrim2014>
# 在 RPC 服务中启用 SSL/TLS 加密时的私钥文件
rpc-certificate=/etc/aria2/fullchain.crt
# 在 RPC 服务中启用 SSL/TLS 加密时的私钥文件
rpc-private-key=/etc/aria2/private.pem
# 设置的RPC访问用户名, 此选项新版已废弃, 建议改用 --rpc-secret 选项
#rpc-user=<USER>
# 设置的RPC访问密码, 此选项新版已废弃, 建议改用 --rpc-secret 选项
#rpc-passwd=<PASSWD>

## BT/PT下载相关 ##

# 当下载的是一个种子(以.torrent结尾)时, 自动开始BT任务, 默认:true
follow-torrent=true
# BT监听端口, 当端口被屏蔽时使用, 默认:6881-6999
listen-port=20003
# 单个种子最大连接数, 默认:55
bt-max-peers=1055
# 打开DHT功能, PT需要禁用, 默认:true
enable-dht=true
# 打开IPv6 DHT功能, PT需要禁用
enable-dht6=false
# DHT网络监听端口, 默认:6881-6999
dht-listen-port=20004-20102
#强制加密, 防迅雷必备
bt-require-crypto=true
# 本地节点查找, PT需要禁用, 默认:false
bt-enable-lpd=true
# 种子交换, PT需要禁用, 默认:true
enable-peer-exchange=true
# 每个种子限速, 对少种的PT很有用, 默认:50K
bt-request-peer-speed-limit=50K
# 客户端伪装, PT需要
peer-id-prefix=-TR2770-
user-agent=Transmission/2.92
#user-agent=netdisk;4.4.0.6;PC;PC-Windows;6.2.9200;WindowsBaiduYunGuanJia
# 当种子的分享率达到这个数时, 自动停止做种, 0为一直做种, 默认:1.0

seed-ratio=1.0
#作种时间大于30分钟，则停止作种
seed-time=30
# 强制保存会话, 话即使任务已经完成, 默认:false
# 较新的版本开启后会在任务完成后依然保留.aria2文件
force-save=false
# BT校验相关, 默认:true
bt-hash-check-seed=true
# 继续之前的BT任务时, 无需再次校验, 默认:false
bt-seed-unverified=true
# 保存磁力链接元数据为种子文件(.torrent文件), 默认:false
bt-save-metadata=true
#下载完成后删除.ara2的同名文件
#on-download-complete=/home/jacob/Downloads/aria2/delete_aria2
#on-download-complete=/home/pi/aria2/rasp.sh

#添加额外的tracker
bt-tracker=udp://tracker.coppersurfer.tk:6969/announce,udp://tracker.internetwarriors.net:1337/announce,udp://tracker.opentrackr.org:1337/announce
```

### 六、搭建emby

```bash
docker run -d \
    --volume /home/jacob/.docker/emby/config:/config \
    --volume /home/jacob/Share/public:/mnt/public \
    --volume /home/jacob/Share/private:/mnt/private \
    -e TZ=Asia/Shanghai \
    --publish 8096:8096 \
    --publish 8920:8920 \
    --env UID=1000 \
    --env GID=1000 \
    --env GIDLIST=1000 \
    emby/embyserver:latest
```
