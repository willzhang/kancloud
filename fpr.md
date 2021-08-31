## fpr

https\://github.com/fatedier/frp/releases



​https\://www\.itcoder.tech/posts/docker-frp/​



server端创建配置文件

```
wget https://github.com/fatedier/frp/releases/download/v0.34.0/frp_0.34.0_linux_amd64.tar.gz
tar -zxvf frp_0.34.0_linux_amd64.tar.gz
cd frp_0.34.0_linux_amd64/
cp frps.ini /etc/frp/
```

启动docker

```
docker run -d --name frps \
  --network host \
  -v /etc/frp/frps.ini:/etc/frp/frps.ini \
  snowdreamtech/frps
```

客户端windows

```
wget https://github.com/fatedier/frp/releases/download/v0.34.0/frp_0.34.0_windows_amd64.zip
```

sakura frp配置

![](images/screenshot\_1625806506671.png)

