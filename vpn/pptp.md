## pptp

部署pptp

<https://hub.docker.com/r/mobtitude/vpn-pptp>

创建配置文件

```
mkdir /data/pptp/

cat > /data/pptp/chap-secrets <<EOF
# client    server      secret      acceptable local IP addresses
username    *           password    *
EOF
```

运行容器

```bash
docker run -d --name pptpserver \
  --restart always \
  --privileged \
  --net=host \
  -v /data/pptp/chap-secrets:/etc/ppp/chap-secrets \
  mobtitude/vpn-pptp
```

