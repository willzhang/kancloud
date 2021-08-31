## docker安装ssr

准备vps服务器

国外vps服务器推荐：

https\://www\.vultr.com/

https\://bandwagonhost.com/

阿里云或腾讯云轻量服务器：

https\://cloud.tencent.com/product/lighthouse

https\://cn.aliyun.com/product/swas

## Docker安装shadowsocks

docker镜像官方地址：

https\://hub.docker.com/r/shadowsocks/shadowsocks-libev

https\://github.com/shadowsocks/shadowsocks-libev

集成kcptun加速:

https\://hub.docker.com/r/xtaci/kcptun

https\://github.com/xtaci/kcptun

安装shadowsocks-libev

```
docker network create ssr
docker run -d --name ssserver \
  --restart=always \
  --net ssr \
  -e PASSWORD=123456 \
  -e METHOD=aes-256-gcm \
  -p 20000:8388 \
  -p 20000:8388/udp \
  shadowsocks/shadowsocks-libev
```

创建kcptun配置文件

```
mkdir /kcptun
cat > /kcptun/config.json <<EOF
{
  "listen":":20001",
  "target":"ssserver:8388",
  "key":"123456",
  "crypt":"aes-192",
  "mode":"fast3",
  "autoexpire":60
}
EOF
```

安装kcptun

```
docker run -d --name kcptun \
  --restart=always \
  --net ssr \
  -p 20001:20001 \
  -p 20001:20001/udp \
  -v /kcptun:/kcptun \
  xtaci/kcptun \
  server -c "/kcptun/config.json"
docker-compose文件
# cat docker-compose.yml
version: "3"

services:
  ssserver:
    image: shadowsocks/shadowsocks-libev
    container_name: ssserver
    restart: always
    ports:
      - "20000:8388"
      - "20000:8388/udp"
    networks:
      - ssr
    environment:
      - PASSWORD=123456
      - METHOD=aes-256-gcm
               
  kcptun:
    image: xtaci/kcptun
    container_name: kcptun
    command: server -c "/kcptun/config.json"  
    volumes:
      - /kcptun/config.json:/kcptun/config.json
    ports:
      - "20001:20001"
      - "20001:20001/udp"  
    networks:
      - ssr
    depends_on:
      - ssserver

networks:
  ssr:
```

docker-compsoe运行容器

```
docker-compose up -d
```

## 客户端配置

下载shadowsocks windows客户端

 https\://github.com/shadowsocks/shadowsocks-windows/releases

不使用kcptun加速的配置方式：

![](../images/screenshot\_1625806010709.png)

使用kcptun的配置方式：

下载kcptun客户端：

https\://github.com/xtaci/kcptun/releases

选择kcptun-windows-amd64-20200701.tar.gz下载解压后将client_windows_amd64.exe复制到shadowsocks根目录下：

![](../images/screenshot\_1625806036148.png)

配置shadowsocks客户端

插件程序填写：

```
client_windows_amd64.exe
```

勾选需要命令行参数，插件参数填写：

```
-l %SS_LOCAL_HOST%:%SS_LOCAL_PORT% -r %SS_REMOTE_HOST%:%SS_REMOTE_PORT% --key 123456 --crypt aes-192 --mode fast3 --autoexpire 60
```

配置示例

![](../images/screenshot\_1625806066805.png)



## mritd方式

这里使用dockerhub上的镜像，建议访问该网站参考更多内容：

镜像地址：https\://hub.docker.com/r/mritd/shadowsocks

拉取镜像并运行：

```
docker run -dt \
  --name ssserver \
  -p 6443:6443 \
  -p 6500:6500/udp \
  mritd/shadowsocks \
  -m "ss-server" \
  -s "-s 0.0.0.0 -p 6443 -m chacha20-ietf-poly1305 -k 123456" \
  -x -e "kcpserver" \
  -k "-t 127.0.0.1:6443 -l :6500 -mode fast2"
```

ssr安装脚本

```
#!/bin/bash
#this is a shadowsocks shell
hostnamectl set-hostname ss_server
yum install -y epel-release git python-setuptools && easy_install pip
pip install git+https://github.com/shadowsocks/shadowsocks.git@master
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
    "method": "aes-256-ctr"
}
EOF
systemctl restart firewalld
firewall-cmd --zone=public --add-port=8728-8731/tcp --permanent
firewall-cmd --reload
ssserver -c /etc/shadowsocks.json -d start
```

\#获取安装信息

```
get_ip=$(ip a | grep eth0 | grep inet | awk '{print $2}' |sed 's/\/.*//')
shadowsocks_port_pwd=$(sed -n '4,7p' /etc/shadowsocks.json | sed 's/^[ \t]*//g')
shadowsockscipher=$(awk -F \" '/method/ {print $4}' /etc/shadowsocks.json)
echo " " > ss_info.txt
echo "##############################################################"
echo -e "Shadowsocks server install completed!" >> ss_info.txt
echo " " >> ss_info.txt
echo -e "Your server ip： \n$get_ip" >> ss_info.txt
echo " " >> ss_info.txt
echo -e "Your Server Port and password：\n$shadowsocks_port_pwd" >> ss_info.txt
echo " " >> ss_info.txt
echo -e "Your Encryption Method: \n$shadowsockscipher" >> ss_info.txt
cat ss_info.txt
echo
echo -e 'see this info again run the follow command \n# cat ss_info.txt'
echo
echo "##############################################################"
ssserver -c /etc/shadowsocks.json -d status
```

snap安装

https\://github.com/shadowsocks/shadowsocks-libev#build-from-source-with-centos



shadowsocksR客户端

https\://github.com/shadowsocksrr/shadowsocksr-csharp



