## shadowsocks简介

dockerhub地址

https\://hub.docker.com/r/mritd/shadowsocks 

## python版shadowsocks

安装git、pyhton、pip

```
yum install git –y git python-setuptools
```

安装shadowsocks

```
pip install git+https://github.com/shadowsocks/shadowsocks.git@master
```

如果想启用ss的chacha20高级加密，需要安装libsodium

libsodium是给SS提供chacha20、salsa20、chacha20-ietf等高级加密所必须的扩展库，因为chacha20加密，安全性与aes-256-cfb相近，但效率比aes-256-cfb高，所以推荐启用chacha20加密.为了启用chacha20加密，我们先安装libsodium。

```
yum install -y epel-release
yum install -y libsodium python34-pip
pip3 install  git+https://github.com/shadowsocks/shadowsocks.git@master
```

源码编译安装：

\#因为这库是基于C语言的，所以我们先去安装GCC

```
 yum -y groupinstall "Development Tools"
```

```
#下载最新稳定版本
wget https://download.libsodium.org/libsodium/releases/LATEST.tar.gz
#解压
tar xf LATEST.tar.gz && cd libsodium-1.0.11
#编译
./configure && make -j4 && make install
echo /usr/local/lib > /etc/ld.so.conf.d/usr_local_lib.conf
ldconfig
```

然后我们就可以修改ss的配置文件来开启效率更高的chacha20加密.

手动配置shadowsocks

```
sudo ssserver -p 8381 -k 123456 -m aes-256-cfb
```

后台启动服务

```
sudo ssserver -p 8381 -k 123456 -m aes-256-cfb --user nobody -d start
```

停止服务

```
sudo ssserver -d stop
```

检查日志文件

```
sudo less /var/log/shadowsocks.log
```

查看命令选项

```
sudo ssserver -h
```

创建shadowsocks配置文件

单用户配置文件

```
# vim /etc/shadowsocks.json
{
    "server":"0.0.0.0",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

多用户配置文件

```
# vim /etc/shadowsocks.json
{
    "server": "0.0.0.0",
    "port_password": {
        "8381": "password1",
        "8382": "password2",
        "8383": "password3",
        "8384": "password4"
    },
    "timeout": 300,
    "method": "xchacha20-ietf-poly1305"
}
```

配置文件参数说明

```
Name
Explanation
server
the address your server listens
server_port
server port
local_address
the address your local listens
local_port
local port
password
password used for encryption
timeout
in seconds
method
default: "aes-256-cfb", see Encryption
fast_open
use TCP_FASTOPEN, true / false
workers
number of workers, available on Unix/Linux
```

前端启动shadowsocks服务

```
ssserver -c /etc/shadowsocks.json
```

后台启动shadowsocks服务

```
ssserver -c /etc/shadowsocks.json -d start
```

关闭shadowsocks服务

```
ssserver -c /etc/shadowsocks.json -d stop
```

查看shadowsocks服务运行状态

```
ssserver -c /etc/shadowsocks.json -d status
```

重启shadowsocks服务

```
ssserver -c /etc/shadowsocks.json -d restart
```

编写自动化安装脚本

```
#！/bin/bash
hostnamectl set-hostname mycentos
systemctl stop firewalld.service && systemctl disable firewalld.service
yum update -y
yum install -y git epel-release libsodium python34-pip
yum install python-setuptools && easy_install pip
pip3 install  git+https://github.com/shadowsocks/shadowsocks.git@master
cat > /etc/shadowsocks.json  << EOF
{
    "server": "0.0.0.0",
    "port_password": {
        "8728": "123456",
        "8729": "123456",
        "8730": "123456",
        "8731": "123456"
    },
    "timeout": 300,
    "method": " xchacha20-ietf-poly1305"
}
EOF
ssserver -c /etc/shadowsocks.json -d start
```

github下载脚本地址：

进入对应仓库，点击脚本文件，点击raw获取下载路径

下载并执行脚本

```
# wget https://raw.githubusercontent.com/zhwill/shadowsocks/master/shadowsocks.sh
# chmod +x shadowsocks.sh
# ./shadowsocks.sh
```

Shadowsocks-libevy源码安装

CentOS7源码编译安装Shadowsocks-libev

如果使用CENTOS 7，则需要安装这些预编译以从源代码构建：

```
# yum install gcc gettext autoconf libtool automake make pcre-devel asciidoc xmlto c-ares-devel libev-devel libsodium-devel mbedtls-devel –y
# yum install git vim wget epel-release -y
```

下载源码包

```
# wget https://github.com/shadowsocks/shadowsocks-libev/releases/download/v3.2.0/shadowsocks-libev-3.2.0.tar.gz
```

编译安装

```
# tar xf shadowsocks-libev-3.2.0.tar.gz
# cd shadowsocks-libev-3.2.0
# ./configure && make && make install
```

创建配置文件，添加如下内容

```
[root@CentOS7 ~]# vim /etc/shadowsocks-libev/config.json
{
    "server":["0.0.0.0"],
    "server_port":8381,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"123456",
"timeout":600,
"fast_open": false,
    "method":"chacha20-ietf-poly1305"
}
```

多端口配置文件

```
{
    "server":["0.0.0.0"],
    "local_address":"127.0.0.1",
    "local_port":1080,
    "port_password": {
        "8381": "123456",
        "8382": "123456"
     }
    "timeout": 600,
    "method": "chacha20-ietf-poly1305"
}
```

设置开机自动启动

```
vi /etc/systemd/system/shadowsocks.service
/etc/systemd/system/shadowsocks.service
[Unit]
Description=Shadowsocks Server
After=network.target
[Service]
ExecStart=/usr/bin/ss-server -c /etc/shadowsocks-libev/config.json -u
Restart=on-abort
[Install]
WantedBy=multi-user.target
```

运行

启动 shadowsocks 服务：

```
systemctl start shadowsocks
```

此时，我们还不能通过外网访问服务器，因为防火墙并没有开启相应的端口，编辑防火墙开放的端口服务：

```
vi /etc/firewalld/zones/public.xml
添加如下行：
<port protocol="tcp" port="服务器端口"/>
<port protocol="udp" port="服务器端口"/>
```

使新规则生效：

```
firewall-cmd --complete-reload
```

至此，shadowsocks 已经可以使用。可以查看服务状态：

```
systemctl status shadowsocks
```

## 工具测速

3安装测速工具 Speedtest.net是比较广泛的用来测试宽带速度的网站，Speedtest.net的工作原理并不复杂：它在你的浏览器中加载JavaScript代码并自动检测离你最近的Speedtest.net服务器，然后向服务器发送HTTP GET and POST请求来测试上行/下行网速。 

安装pip speedtest是用python写的，没使用过pip的需要先安装pip， 

```
yum install python-pip –y
```

安装speedtest-cli 

```
pip install speedtest-cli 
```

安装完成测试，speedtest-cli命令会查找距离最近的Speedtest.net服务器。并打印出你的下行和上行带宽。 

```
[root@VM_0_12_centos ~]# speedtest-cli 
Retrieving speedtest.net configuration...
Testing from Tencent cloud computing (119.28.116.254)...
Retrieving speedtest.net server list...
Selecting best server based on ping...
Hosted by Beijing Telecom (Beijing) [1.69 km]: 77.193 ms
Testing download speed................................................................................
Download: 99.83 Mbit/s
Testing upload speed................................................................................................
Upload: 74.80 Mbit/s
```

使用“–share”参数，会将测试结果上传到Speedtest.net服务器并以图形的方式展示。通过链接地址获得图形报告。 

speedtest-cli --share 

其他用法 使用speedtest-cli -h获取使用帮助

## 编写ssr规则

选择编辑GFWList的用户规则：

![](../images/screenshot\_1625805866713.png)

格式如下

编辑user-rule.txt

```
! Put user rules line by line in this file.
! See https://adblockplus.org/en/filter-cheatsheet
||https://www.gitbook.com/^
```



