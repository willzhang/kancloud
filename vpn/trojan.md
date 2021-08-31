## trojan简介

一般的科学上网采用强加密和随机混淆来欺骗GFW的过滤机制。然而，Trojan采用最常见的协议HTTPS，以诱骗GFW认为它是HTTPS。当Trojan客户端连接到服务器时，它首先执行真正的 TLS握手。如果握手成功，则所有后续流量都将受到保护TLS; 否则，服务器将立即关闭连接，就像任何HTTPS服务器一样。Trojan反侦查采用主动检测与被动检测，

主动检测：没有正确结构和密码的所有连接将被重定向到预设端点，因此HTTP如果可疑探针连接（或者只是您连接到博客XD的粉丝），则特洛伊木马服务器的行为与该端点完全相同（默认情况下）。

被动检测：由于流量受到保护TLS（用户有责任使用有效证书），如果您正在访问某个HTTP站点，则流量看起来与HTTPS（握手RTT后只有一个TLS）相同; 如果您没有访问某个HTTP站点，那么流量看起来就像HTTPS保持活动一样WebSocket。因此，木马也可以绕过ISP QoS限制。 理论上来说，Trojan可以永久地穿越Great FireWall，而不会被识别出来。

github地址： <https://github.com/trojan-gfw/trojan>

官方文档：<https://trojan-gfw.github.io/trojan/>

docker镜像：<https://hub.docker.com/r/trojangfw/trojan>

## 部署trojan

创建目录，申请并上传证书到该目录下

```
mkdir /etc/trojan-go && cd /etc/trojan-go
```

创建配置文件：

```json
# vi config.json

{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "123456"
    ],
    "ssl": {
        "cert": "/etc/trojan-go/willzhangcloud.com_chain.crt",
        "key": "/etc/trojan-go/willzhangcloud.com_key.key",
        "sni": "willzhangcloud.com"
    }
}
```

运行nginx容器

```docker
docker run -d --restart always \
  --name nginx \
  -p 80:80 \
  nginx 
```

运行trojan容器

```docker
docker run -d --restart always \
  -p 443:443 \
  --name trojan-go \
  --restart=always \
  -v /etc/trojan-go:/etc/trojan-go \
  teddysun/trojan-go
```

下载windows客户端：<https://github.com/trojan-gfw/trojan/releases>

修改config.json配置文件格式如下：

```json
{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1081,
    "remote_addr": "willzhangcloud.com",
    "remote_port": 443,
    "password": [
        "123456"
    ],
    "log_level": 1,
    "ssl": {
        "verify": true,
        "verify_hostname": true,
        "cert": "",
        "cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES128-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA:AES128-SHA:AES256-SHA:DES-CBC3-SHA",
        "cipher_tls13": "TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
        "sni": "willzhangcloud.com",
        "alpn": [
            "h2",
            "http/1.1"
        ],
        "reuse_session": true,
        "session_ticket": false,
        "curves": ""
    },
    "tcp": {
        "no_delay": true,
        "keep_alive": true,
        "reuse_port": false,
        "fast_open": false,
        "fast_open_qlen": 20
    }
}
```

说明：此处需要修改一下”remote_addr”为你的域名，还有设置的password。配置好后，启动trojan。

改好后保存并关闭文件，双击文件夹内的 trojan.exe 文件，trojan程序运行，出现如下黑窗口：

与SS/SSR/v2ray等客户端不同，trojan运行出现上述界面后，浏览器无法直接上外网，需要进行额外的设置。本文介绍两种方式：1. 设置系统代理；2. 借助v2rayN。

借助v2rayN上网

设置系统代理方式不能方便的切换pac和全局模式，本节介绍使用v2rayN客户端达到灵活上外网。

1\. 从 v2ray客户端 下载v2rayN，解压进入v2rayN-Core文件夹。双击文件夹内的 v2rayN.exe 启动，在桌面右下角找到v2rayN的图标（logo是V），双击打开配置界面，按下图添加socks5服务器：

![](../images/screenshot\_1625804751975.png)

## 申请免费ssl证书

freessl:

<https://freessl.cn/>

![](../images/screenshot\_1625804841572.png)

根据提示，本地安装KeyManager

godaddy添加TXT类型记录

![](../images/screenshot\_1625804865277.png)

域名验证

![](../images/screenshot\_1625804880725.png)
