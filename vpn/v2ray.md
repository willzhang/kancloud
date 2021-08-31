## v2ray简介

官方网站：<https://www.v2ray.com/>

github地址：<https://github.com/v2ray/v2ray-core>

android客户端：<https://github.com/2dust/v2rayNG>

## 服务端安装

安装docker

```
curl -sS https://get.docker.com | sh && systemctl enable --now docker
```

创建配置文件

```
mkdir /etc/v2ray
cat > /etc/v2ray/config.json << EOF
{
  "inbounds": [{
    "port": 8888, // 服务器监听端口，必须和客户端一致
    "protocol": "vmess",
    "settings": {
      "clients": [{ "id": "b831381d-6324-4d53-ad4f-8cda48b30811" }]
    }
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  }]
}
EOF
```

运行docker容器

```
docker run -itd --name v2ray \
  --restart always \
  -v /etc/v2ray:/etc/v2ray \
  -p 8888:8888 v2ray/official \
  v2ray -config=/etc/v2ray/config.json
```

## 客户端配置

下载客户端（v2rayN-Core.zip）：

<https://github.com/2dust/v2rayN/releases>

或单独下载core文件，对v2rayN-Core.zip进行解压，然后将V2RayN.exe复制到v2ray-windows-64.zip解压后的目录：

<https://github.com/v2ray/v2ray-core/releases>

![](../images/screenshot\_1625805591363.png)

图2

![](../images/screenshot\_1625805601696.png)

编辑配置

UUID 与服务端配置一致即可

![](../images/screenshot\_1625805618404.png)

保存后点击右下角图标，勾选以下2项：

![](../images/screenshot\_1625805640880.png)
