
## 基本配置
分流用到以下三个
|类型|说明|
|-|-|
|inbounds|入站|
|outbounds|出战|
|routing|路由|


|类型|说明 ||
|---|---|---|
|direct|直连|
|blackhole|黑洞|
|Tunnel|旧称 dokodemo-door（任意门），|
|Freedom|出站协议可以用来向任意网络发送（正常的） TCP 或 UDP 数据。|
|"type"|"field"|该项暂时没有特别定义，但是不能省略，所以记得写上就好|

`balancerTag` 和 `outboundTag` 须二选一。当同时指定时，`outboundTag` 生效。

+ 监听本地端口10808。
+ 监听端口收到数据转到路由模块，路由模块在这里处理分流该走哪。
+ `type`路由标识，不能重复

### 数据包处理流程
```json
APP数据 ----> 入站 ----> 路由(Xray) ----> 出战 ----> VPS

{
	// 日志
    "log": {
        "loglevel": "warning"
    },
	// 路由
    "routing": {
        "domainStrategy": "AsIs",
        "rules": [
            {
                "type": "field",	// 协议类型所有类型；可匹配：ip、domain、port、inboundTag、outboundTag、protocol、network 等。
                "ip": [
                    "geoip:private"		// 局域网IP
                ],
                "outboundTag": "direct"		// 内置默认直连出站
            },
            {
                // 协议类型匹配全部，"http-in"入站，标签流量转发到"to-socks"出战
                "type": "field",
                "inboundTag": ["http-in"],	// 对应57行 标签
                "outboundTag": "to-socks"	// 对应71行 标签
            }
        ]
    },
	// 入站
    "inbounds": [
        {
            "listen": null,
            "port": 10808,
            "protocol": "vmess",
            "settings": {
                "clients": [
                    {
                        "id": "0c41318a-32b6-4017-9938-4033bf26b111"
                    }
                ]
            },
            "streamSettings": {
                "network": "ws",
                "security": "none"
            }
        },
        {
            "listen": null,
            "port": 10809,
            "protocol": "vmess",
            "settings": {
                "clients": [
                    {
                        "id": "0c41318a-32b6-4017-9938-4033bf26b111"
                    }
                ]
            },
            "streamSettings": {
                "network": "ws",
                "security": "none"
            },
            "tag": "http-in"
        }
    ],
	// 出战
    "outbounds": [
        {
            "protocol": "freedom",
            "tag": "direct"
        },
        {
            "protocol": "blackhole",
            "tag": "block"
        },
        {
            "tag": "to-socks",
            "protocol": "socks",	// 类型为socks，详细可见官网配置
            "settings": {
                "servers": [
                    {
                        "address": "127.0.0.1",
                        "port": 1080
                    }
                ]
            }
        }
    ]
}
```

## 配置说明
### "任意门"
```json 
/*
任意门主要有两个用处 一个是用作透明代理(见下)，另一个是映射一个端口。
有时一些服务并不支持使用 Socks5 这样的正向代理，使用 Tun 或者 Tproxy 又有些小题大做了，而这些服务又只和一个 IP 一个端口通信 (比如: iperf, Minecraft server, Wireguard endpoint), 就可以用到任意门。
如以下 Config (假设默认出站为一有效代理)
*/

{
  "listen": "127.0.0.1",
  "port": 25565,
  "protocol": "tunnel",
  "settings": {
    "address": "mc.hypixel.net",
    "port": 25565,
    "network": "tcp",
    "followRedirect": false,
    "userLevel": 0
  },
  "tag": "mc"
}

// 这时候核心会监听 127.0.0.1:25565 并通过默认出站转发至 mc.hypixel.net:25565 (一个MC服务器), 这时候再通过 Minecraft 客户端连接 127.0.0.1:25565, 就相当于通过代理连接了 Hypixel 服务器
```

## 内置DNS服务器
Xray 内置的 DNS 模块，主要有三大用途：
+ 在路由阶段，解析域名为 IP, 并且根据域名解析得到的 IP 进行规则匹配以分流。是否解析域名及分流和路由配置模块中 domainStrategy 的值有关，只有在设置以下两种值时，才会使用内置 DNS 服务器进行 DNS 查询：
  + "IPIfNonMatch", 请求一个域名时，进行路由里面的 domain 进行匹配，若无法匹配到结果，则对这个域名使用内置 DNS 服务器进行 DNS 查询，并且使用查询返回的 IP 地址再重新进行 IP 路由匹配。
"IPOnDemand", 当匹配时碰到任何基于 IP 的规则，将域名立即解析为 IP 进行匹配。
+ 解析目标地址进行连接：
  + 如 在 freedom 出站中，将 domainStrategy 设置为 UseIP, 由此出站发出的请求, 会先将域名通过内置服务器解析成 IP, 然后进行连接。
  + 如 在 sockopt 中，将 domainStrategy 设置为 UseIP, 此出站发起的系统连接，将先由内置服务器解析为 IP, 然后进行连接。
+ 透明代理时劫持 DNS 流量；或直接对外暴露 53 端口充当递归 DNS 服务器。

```json
  "dns": { //内置DNS配置
    "servers": [
       {
         "address": "114.114.114.114",
         "domains": [
           "geosite:cn" //国内域名用腾讯DNS解析
         ],
         "skipFallback": true
       },
       {
         "address": "8.8.8.8",
         "domains": [
           "geosite:geolocation-!cn" //国外域名用谷歌DNS解析
         ],
         "skipFallback": true
       },
       {
         "address": "1.1.1.1", //不匹配上述domain列表的用此服务器解析
         "skipFallback": false
       }
    ],
    "queryStrategy": "UseIP",   // 允许查询 A + AAAA
    "disableCache": false,      // DNS缓存
    "disableFallback": false,
    "disableFallbackIfMatch": false,
    "tag": "inner-dns"
  },
```