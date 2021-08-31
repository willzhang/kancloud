## openvpn简介

OpenVPN：一个实现 VPN 的开源软件，OpenVPN 是一个健壮的、高度灵活的 VPN 守护进程。它支持 SSL/TLS 安全、Ethernet bridging、经由代理的 TCP 或 UDP 隧道和 NAT。另外，它也支持动态 IP 地址以及 DHCP，可伸缩性足以支持数百或数千用户的使用场景，同时可移植至大多数主流操作系统平台上。

官网：<https://openvpn.net>

GitHub 地址：<https://github.com/OpenVPN/openvpn>

## docker安装openvpn

<https://github.com/kylemanna/docker-openvpn>

<https://github.com/kylemanna/docker-openvpn/tree/master/docs>

<https://hub.docker.com/r/dperson/openvpn-client>

定义3个变量：

```bash
OVPN_DATA="ovpn-data-example-us01"
VPN_SERVERNAME="vpn.example.com"
CLIENTNAME="us01.vpn.example.com"
```

参数说明：

* $OVPN_DATA，它用于数据卷容器。建议使用ovpn-data-前缀与参考systemd服务无缝操作。
* $VPN_SERVERNAME，该值应该是域名或 IP 地址。
* $CLIENTNAME，它将用作客户端定义。你可以用一些令人难忘的东西来命名它，这样你就可以注意到你在哪里连接。

创建`$OVPN_DATA`卷

```
docker volume create --name $OVPN_DATA
```

初始化容器用于保存配置文件和证书，容器将提示输入密码来保护新生成的证书颁发机构使用的私钥。

```
docker run -v $OVPN_DATA:/etc/openvpn \
--rm kylemanna/openvpn ovpn_genconfig -u udp://${VPN_SERVERNAME}

docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki
```

启动 OpenVPN 服务器进程

```
docker run -d --name openvpn \
  --restart always \
  -p 1194:1194/udp \
  -v $OVPN_DATA:/etc/openvpn \
  --cap-add=NET_ADMIN \
  kylemanna/openvpn
```

生成没有密码的客户端证书

```
docker run -v $OVPN_DATA:/etc/openvpn \
--rm -it kylemanna/openvpn easyrsa build-client-full $CLIENTNAME nopass
```

使用嵌入式证书检索客户端配置，该证书默认有效期三年

```
docker run -v $OVPN_DATA:/etc/openvpn \
--rm kylemanna/openvpn ovpn_getclient CLIENTNAME > CLIENTNAME.ovpn
```

## 客户端安装

下载地址：<https://openvpn.net/vpn-client/>

1. 启动客户端，右键，选择 import file, 导入 ovpn 文件，文件请 联系管理员发给你
2. 右键 connect,如果弹出框提示输入密码，输入默认密码 123456 ，等待连接成功即可

