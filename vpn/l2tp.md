## l2tp简介

l2tp

## docker部署l2tp

创建环境变量文件

```
cat > /etc/l2tp.env <<EOF
VPN_IPSEC_PSK=aliyun
VPN_USER=user1
VPN_PASSWORD=Wise2c2019
VPN_ADDL_USERS=user2 user3
VPN_ADDL_PASSWORDS=Wise2c2019 Wise2c2019
EOF
```

运行容器

```
docker run -d \
    --name l2tpserver \
    --env-file /etc/l2tp.env \
    --restart=always \
    -p 500:500/udp \
    -p 4500:4500/udp \
    --privileged \
    hwdsl2/ipsec-vpn-server
```

配置客户端

<https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/clients-zh.md>
