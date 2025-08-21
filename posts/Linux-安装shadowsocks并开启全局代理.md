---
title: Linux 安装shadowsocks并开启全局代理
date: 2022-10-25 15:09:38
tags:
  - shadowsocks
  - kcp
  - linux
  - proxy
---

linux安装ss客户端与kcp并开启全局代理

<!--more-->

# 1.安装ss客户端和kcp客户端

- kcp客户端：https://github.com/xtaci/kcptun/releases 选择对应版本下载即可
- shadowsocks客户端linux版本：https://github.com/shadowsocks/shadowsocks-rust/releases 选择对应版本

# 2.修改配置启动kcp与ss客户端

## 2.1 配置

- 创建kcp.json, 主要配置远程vps服务地址以及本地端口如: `:10080`
- 创建ss.json, 配置服务地址为kcp配置的本地端口，本地端口设置如：`10088`

## 2.2 启动

```
kcp-client -c kcp.json
sslocal -c ss.json
```





# 3.开启全局代理

```
export ALL_PROXY="socks5://代理服务器IP地址:代理端口"  # 开启sock代理
unset ALL_PROXY  # 关闭代理
```



# 4.使用polipo将socks代理转为http代理

```
apt install polipo
修改配置：/etc/polipo/config： 协议端口指定为socks地址
service start polipo
```

开启代理：

```
export HTTP_PROXY=http://xxxx:xxx
export HTTPS_PROXY=https://xxxx:xxx
```



# 5.开启代理后python使用requets报错

错误信息：ValueError: check_hostname requires server_hostname

解决方式：降低urllib3版本至1.25.8

