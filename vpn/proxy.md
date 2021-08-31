## vpn代理

配置本地代理依然无法ping通www\.google.com原因：

SOCKS是一种网络传输协议，主要用于客户端与外网服务器之间通讯的中间传递。

SOCKS协议是传输层 (第四层)，ICMP协议是网络层(第三层)

ping使用ICMP协议

浏览器能够访问google是使用web通过http协议应用层(第七层)

ssr的socks代理是介于传输层(第四层)和会话层(第五层)

而我们在ping的时候，则是基于网络层(第三层)

众所周知上一层协议的代理 对下层没有任何作用\~

所以说当我们尝试ping谷歌的时候，当然是ping不通的。

但可以通过http进行访问，这里使用代理ssr后的curl 演示

## socks4和socks5的区别

SOCKS代理与其他类型的代理不同，它只是简单地传递数据包，而并不关心是何种应用协议，既可以是HTTP请求，

所以SOCKS代理服务器比其他类型的代理服务器速度要快得多。

SOCKS代理又分为SOCKS4和SOCKS5

二者不同的是SOCKS4代理只支持TCP协议（即传输控制协议），而SOCKS5代理则既支持TCP协议又支持UDP协议

（即用户数据包协议），还支持各种身份验证机制、服务器端域名解析等。SOCK4能做到的SOCKS5都可得到，

但SOCKS5能够做到的SOCK4则不一定能做到，比如我们常用的聊天工具QQ在使用代理时就要求用SOCKS5代理，

因为它需要使用UDP协议来传输数据