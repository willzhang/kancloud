## clash简介

Clash 是一个基于规则的跨平台代理软件核心程序，是一款用 Go开发的支持 Linux/MacOS/Windows等多平台的代理工具，Clash支持 Vmess, Shadowsocks, Snell , SOCKS5 , ShadowsocksR协议，支持规则分流。
Clash分为开源版本与闭源版本（Premium版）：
闭源版本（Premium版）比开源版本多以下功能：TUN、Script（脚本模式）、Rule Providers。
详见：<https://github.com/Dreamacro/clash/wiki/Premium-Core-Features>
闭源版本内核（Premium版）下载地址：<https://github.com/Dreamacro/clash/releases/tag/premium>

各个组件说明：

* [Clash](https://github.com/Dreamacro/clash)：一个 Go 语言开发的多平台代理客户端，Github

* [ClashX](https://github.com/yichengchen/clashX/releases)：Clash 的 Mac 图形客户端，Github

* [ClashForAndroid](https://github.com/Kr328/ClashForAndroid/releases)：Clash 的 Android 图形客户端，Github

* [Clash for Windows](https://github.com/Fndroid/clash_for_windows_pkg/releases)：Clash 的 Windows/macOS 图形客户端

参考文档：
<https://docs.cfw.lbyczf.com/>
<https://github.com/Dreamacro/clash/wiki>
<https://lancellc.gitbook.io/clash/>

## Clash for Windows

Clash for Windows 是运行在 Windows 上的一图形化 Clash 分支。通过 Clash API 来配置和控制 Clash 核心程序，便于用户可视化操作和使用。

返回到「General」部分，将「System Proxy」的开关更改为绿色状态即可开始使用。此外，建议将「Start with Windows」也更改为绿色来让 Clash for Windows 在开机时自动启动。

![](../images/screenshot_1626065945903.png)

## 配置参数

Clash for Windows 界面简介

* `General（常规）`：

  * `Port`；为 HTTP、SOCKS 代理端口，点击终端图案可以打开一个配置了代理的命令行窗口，点击端口数字可以复制该命令；
  * `Allow LAN`：启用局域网共享代理功能；‘
  * `Log Level`：日志等级；
  * `Home Directory`：点击下方路径直达 `C:\Users\用户名\.config\clash` 文件夹；
  * `UWP Loopback` ：可以用来使 UWP 应用解除回环代理限制；
  * `Tap Device` ：安装 cfw-tap 网卡，可用于处理不遵循系统代理的软件；
  * `General YML`：编辑 `config.yml` 文件，可用于配置部分 **General** 页面内容；
  * `Mixin`:
  * `System Proxy`：启用系统代理；
  * `Start with Windows`：设置开机自启；

* `Proxies（代理）`：选择代理方式（Global - 全局、Rule - 规则、Direct - 直连）及策略组节点选择；

* `Profiles（配置管理）`：

  * 用来下载远端配置文件和创建本地副本，且可在多个配置文件间切换；
  * 对配置进行节点、策略组和规则的管理（添加节点、策略组和规则在各自编辑界面选择 `Add`, 调整策略组顺序、节点顺序及策略组节点使用拖拽的方式）；

* `Logs（日志）`：显示当前请求命中规则类型和策略；

* `Connections (连接)`: 显示当前的 TCP 连接，可对某个具体连接执行关闭操作；

* `Feedback（反馈）`：显示软件、作者相关信息，内含捐赠码，欢迎打赏 [Fndroid](https://github.com/Fndroid) 大佬以感谢和支持开发。

## 关于策略组的理解

<https://github.com/Fndroid/jsbox_script/wiki/>

## linux使用clash

下载安装clash

```
wget https://github.com/Dreamacro/clash/releases/download/v1.6.5/clash-linux-amd64-v1.6.5.gz
gunzip -c clash-linux-amd64-v1.6.5.gz > /usr/local/bin/clash
chmod +x /usr/local/bin/clash
clash -v
```

创建clash配置文件，

```yaml
[root@master ~]# cat /root/.config/clash/config.yaml 
mixed-port: 7890
allow-lan: true
bind-address: '*'
mode: rule
log-level: info
external-controller: '127.0.0.1:9090'
dns:
    enable: true
    ipv6: false
    default-nameserver: [223.5.5.5, 119.29.29.29]
    enhanced-mode: redir-host
    fake-ip-range: 198.18.0.1/16
    use-hosts: true
    nameserver: ['https://doh.pub/dns-query']
    fallback: ['tls://1.0.0.1:853', 'https://cloudflare-dns.com/dns-query', 'https://dns.google/dns-query']
    fallback-filter: { geoip: true, ipcidr: [240.0.0.0/4, 0.0.0.0/32] }
proxies:
    - { name: 'SG Netflix', type: ss, server: ap.01.ysdns.cyou, port: 22002, cipher: aes-128-gcm, password: ac41f43a-452e-49eb-a0ba-3e3c0e6671ed, udp: true }
    - { name: '新加坡 02', type: ss, server: ap.01.ysdns.cyou, port: 22001, cipher: aes-128-gcm, password: ac41f43a-452e-49eb-a0ba-3e3c0e6671ed, udp: true }
    - { name: '香港 01', type: ss, server: ap.01.ysdns.cyou, port: 22007, cipher: aes-128-gcm, password: ac41f43a-452e-49eb-a0ba-3e3c0e6671ed, udp: true }
    - { name: '日本 01', type: ss, server: ap.01.ysdns.cyou, port: 22004, cipher: aes-128-gcm, password: ac41f43a-452e-49eb-a0ba-3e3c0e6671ed, udp: true }
    - { name: '日本 02', type: ss, server: ap.01.ysdns.cyou, port: 22003, cipher: aes-128-gcm, password: ac41f43a-452e-49eb-a0ba-3e3c0e6671ed, udp: true }
    - { name: '凤凰城 01', type: ss, server: ap.01.ysdns.cyou, port: 22010, cipher: chacha20-ietf-poly1305, password: ac41f43a-452e-49eb-a0ba-3e3c0e6671ed, udp: true }
    - { name: 'TJ 新加坡 01', type: trojan, server: ap.01.ysdns.cyou, port: 26801, password: ac41f43a-452e-49eb-a0ba-3e3c0e6671ed, udp: true, skip-cert-verify: true }
proxy-groups:
    - { name: YunSails, type: select, proxies: [自动选择, 故障转移, 'SG Netflix', '新加坡 02', '香港 01', '日本 01', '日本 02', '凤凰城 01', 'TJ 新加坡 01'] }
    - { name: 自动选择, type: url-test, proxies: ['SG Netflix', '新加坡 02', '香港 01', '日本 01', '日本 02', '凤凰城 01', 'TJ 新加坡 01'], url: 'http://www.gstatic.com/generate_204', interval: 86400 }
    - { name: 故障转移, type: fallback, proxies: ['SG Netflix', '新加坡 02', '香港 01', '日本 01', '日本 02', '凤凰城 01', 'TJ 新加坡 01'], url: 'http://www.gstatic.com/generate_204', interval: 7200 }
rules:
    - 'DOMAIN,safebrowsing.urlsec.qq.com,DIRECT'
......
```

建议从clash for windows客户端导出现有config.yaml配置文件
![](../images/screenshot_1630129721260.png)

启动clash进程

```
nohup clash &
```

配置代理

```
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
export ALL_PROXY=socks5://127.0.0.1:7890
```

测试访问

```
curl www.google.com
```

## docker方式安装

创建clash配置文件目录

```
mkdir -p /root/.config/clash/
```

创建clash配置文件

```
# cat /root/.config/clash/config.yaml
mixed-port: 7890
allow-lan: true
bind-address: '*'
mode: rule
log-level: info
external-controller: '127.0.0.1:9090'
dns:
    enable: true
    ipv6: false
    default-nameserver: [223.5.5.5, 119.29.29.29]
    enhanced-mode: redir-host
    fake-ip-range: 198.18.0.1/16
    use-hosts: true
    nameserver: ['https://doh.pub/dns-query']
    fallback: ['tls://1.0.0.1:853', 'https://cloudflare-dns.com/dns-query', 'https://dns.google/dns-query']
    fallback-filter: { geoip: true, ipcidr: [240.0.0.0/4, 0.0.0.0/32] }
proxies:
    - { name: 'SG Netflix', type: ss, server: ap.01.ysdns.cyou, port: 22002, cipher: aes-128-gcm, password: ac41f43a-452e-49eb-a0ba-3e3c0e6671ed, udp: true }
    - { name: '新加坡 02', type: ss, server: ap.01.ysdns.cyou, port: 22001, cipher: aes-128-gcm, password: ac41f43a-452e-49eb-a0ba-3e3c0e6671ed, udp: true }
    - { name: '香港 01', type: ss, server: ap.01.ysdns.cyou, port: 22007, cipher: aes-128-gcm, password: ac41f43a-452e-49eb-a0ba-3e3c0e6671ed, udp: true }
    - { name: '日本 01', type: ss, server: ap.01.ysdns.cyou, port: 22004, cipher: aes-128-gcm, password: ac41f43a-452e-49eb-a0ba-3e3c0e6671ed, udp: true }
    - { name: '日本 02', type: ss, server: ap.01.ysdns.cyou, port: 22003, cipher: aes-128-gcm, password: ac41f43a-452e-49eb-a0ba-3e3c0e6671ed, udp: true }
    - { name: '凤凰城 01', type: ss, server: ap.01.ysdns.cyou, port: 22010, cipher: chacha20-ietf-poly1305, password: ac41f43a-452e-49eb-a0ba-3e3c0e6671ed, udp: true }
    - { name: 'TJ 新加坡 01', type: trojan, server: ap.01.ysdns.cyou, port: 26801, password: ac41f43a-452e-49eb-a0ba-3e3c0e6671ed, udp: true, skip-cert-verify: true }
proxy-groups:
    - { name: YunSails, type: select, proxies: [自动选择, 故障转移, 'SG Netflix', '新加坡 02', '香港 01', '日本 01', '日本 02', '凤凰城 01', 'TJ 新加坡 01'] }
    - { name: 自动选择, type: url-test, proxies: ['SG Netflix', '新加坡 02', '香港 01', '日本 01', '日本 02', '凤凰城 01', 'TJ 新加坡 01'], url: 'http://www.gstatic.com/generate_204', interval: 86400 }
    - { name: 故障转移, type: fallback, proxies: ['SG Netflix', '新加坡 02', '香港 01', '日本 01', '日本 02', '凤凰城 01', 'TJ 新加坡 01'], url: 'http://www.gstatic.com/generate_204', interval: 7200 }
rules:
    - 'DOMAIN,safebrowsing.urlsec.qq.com,DIRECT'
......
```

启动clash

```
docker run -d --name clash-client \
  --restart always \
  -p 7890:7890 \
  -p 9090:9090 \
  -v /root/.config/clash/:/root/.config/clash/ \
  dreamacro/clash
```

## clash-dashboard

下载安装

```
mkdir -p /root/.config/clash/
wget https://github.com/Dreamacro/clash-dashboard/archive/gh-pages.zip
unzip gh-pages.zip -d /root/.config/clash/
```

修改clash配置文件

```
external-controller: 0.0.0.0:9090
secret: "123456"
external-ui: clash-dashboard-gh-pages
```

重启clash，浏览器访问：<http://192.168.93.51:9090/ui>

docker集成clash-dashboard

```
docker run -d --name clash-client \
  --restart always \
  -p 7890:7890 \
  -p 9090:9090 \
  -v /root/.config/clash:/root/.config/clash \
  dreamacro/clash
```

^
