## proxychains-ng

ProxyChains 是一个强制应用的 TCP 连接通过代理的工具，支持 Tor、HTTP、与 Socks 代理。与 sshuttle 不同的是，ProxyChains 只会将当前应用的 TCP 连接转发至代理，而非全局代理。

github地址：
<https://github.com/rofl0r/proxychains-ng>
<https://github.com/haad/proxychains>
<http://proxychains.sourceforge.net/>

proxychains ng（新一代）是一个预加载程序，它在动态链接的程序中钩住对套接字的调用，并通过一个或多个socks/http代理重定向它。继续未维护的proxychains项目。sf.net页面当前未更新，请改用github发布页面中的版本。

如果您的 Linux 机器在代理服务器后面，那么您可以安装 Proxychains 包以通过给定的代理地址访问互联网。Proxychains 是一个开源软件，它通过 TOR（默认情况下）、SOCKS4、SOCKS5 和 HTTP 等代理强制给定应用程序的任何 TCP 连接。

linux下代理一般是通过http_proxy和https_proxy这两个环境变量，但是很多软件并不使用这两个变量，导致流量无法走代理。 在不使用vpn的前提下，linux并没有转发所有流量的真全局代理。但是可以用proxychains-ng为程序指定走代 理，proxychains-ng是proxychains的加强版，主要有以下功能：

* 支持http/https/socks4/socks5

* 支持认证

* 远端dns查询

* 多种代理模式

不足：

* 不支持udp/icmp转发
* 少部分程序和在后台运行的可能无法代理]

proxychains-ng支持多种代理模式：

* dynamic_chain ：按照代理列表顺序自动选取可用代理
* strict_chain ：按照代理列表顺序使用代理，所有代理必须可用
* round_robin_chain ：轮询模式，自动跳过不可用代理
* random_chain ：随机模式

yum安装

```
dnf install -y epel-release
dnf install -y proxychains-ng
```

编译安装：

```
wget https://ftp.barfooze.de/pub/sabotage/tarballs/proxychains-ng-4.15.tar.xz
tar -xf proxychains-ng-4.15.tar.xz
cd proxychains-ng-4.15/

dnf install -y gcc
./configure --prefix=/usr --sysconfdir=/etc
make && make install
make install-config
```

创建配置文件，搭配clash使用

```
cat >/etc/proxychains.conf<<EOF
dynamic_chain
chain_len = 1
proxy_dns
quiet_mode
remote_dns_subnet 224
tcp_read_time_out 15000
tcp_connect_time_out 8000
[ProxyList]
socks5  127.0.0.1 7890
EOF
```

使用方法：
```
proxychains4 程序 参数
```
测试连接
```
proxychains4 curl ip.cn
proxychains4 curl ipinfo.io
proxychains4 wget -qO- http://ipecho.net/plain ; echo
proxychains4 telnet google.com 80
proxychains4 wget https://www.google.com

proxychains4 git clone https://github.com/rofl0r/proxychains-ng
proxychains4 apt-get update
proxychains4 npm install
```

这样用每次都要在命令前输入proxychains4，比较麻烦，可以用proxychains4代理一个shell，在shell中执行的命令就会自动使用代理了，例如：

```
proxychains4  -q /bin/bash
wget https://www.google.com
```

docker方式安装
```
docker run -d --name proxychains-ng \
  --restart always \
  -p 7890:7890 \
 cheyoust/proxychains-ng:4.14
```

也有例外，这样使用并没有任何效果：

```
proxychains4 ping google.com

[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/local/lib/libproxychains4.dylib
PING google.com (172.217.27.142): 56 data bytes
Request timeout for icmp_seq 0
Request timeout for icmp_seq 1
Request timeout for icmp_seq 2
Request timeout for icmp_seq 3
Request timeout for icmp_seq 4
```

因为 proxychains 只会代理 TCP 连接，而 ping 使用的是 ICMP。记住这一点即可。
