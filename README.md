## 免费最终版镜像4.19：manfly17/v2ray_v3:go
## 收费版4.22.1.6镜像：manfly17/v2ray_v3:go_4.22.1.6

## 使用方法：

mkdir v2ray-agent  &&  \
cd v2ray-agent && \
curl https://raw.githubusercontent.com/manfly17/v2ray-sspanel-v3-mod_Uim-plugin/dev/install.sh -o install.sh && \
chmod +x install.sh && \
bash install.sh

中转用法是在前端节点地址后面加上|outside_port=中转端口|relayserver=中转ip

// ws完整写法示例：
```
xxxxx.com;10550;16;ws;;path=/xxxxx|host=oxxxx.com|outside_port=中转端口|relayserver=中转ip
```
其他写法自行添加

感恩原作者rico辛苦付出
本人仅做备份和后续维护
caddy镜像更新支持tls1.3

## 作为 ss 后端

面板配置是节点类型为 Shadowsocks，普通端口。

加密方式只支持：

- [x] aes-256-cfb
- [x] aes-128-cfb
- [x] chacha20
- [x] chacha20-ietf
- [x] aes-256-gcm
- [x] aes-128-gcm
- [x] chacha20-poly1305 或称 chacha20-ietf-poly1305
- [x] xchacha20-ietf-poly1305
## 作为 SS + WS(tls) 配置，单端口
### 节点配置
添加一个节点

节点类型为 Shadowsocks - V2Ray-Plugin

节点地址写法
以下是节点地址的基本格式：

> 没有CDN的域名或者IP;外部端口;;协议层;附加协议;额外参数

**额外参数：**

额外参数使用 "|" 来分隔。

- **path** 为访问路径
- **server** 为 TLS 域名和用于当节点藏在 CDN 后时覆盖第一个地址
- **host** 用于定义 headers
配置示例：
```
// ws
没有CDN的域名或IP;端口;;ws;;path=/hls/cctv5phd.m3u8

// ws + tls
没有CDN的域名或IP;443;;ws;tls;path=/hls/cctv5phd.m3u8|server=TLS域名

// ws+tls+中转

没有CDN的域名或IP;443;;ws;tls;path=/hls/cctv5phd.m3u8|server=TLS域名|host=TLS域名|relayserver=中转地址|outside_port=中转端口

// ws + tls+CDN
没有CDN的域名或IP;443;;ws;tls;path=/hls/cctv5phd.m3u8|server=这里写CDN的域名

// obfs-http
没有CDN的域名或IP;端口;;obfs;http;
```
## 作为 V2ray 后端

这里面板设置是节点类型v2ray, 普通端口。 v2ray的API接口默认是2333

支持 tcp,kcp、ws+(tls 由镜像 Caddy或者ngnix 提供,默认是443接口哦)。或者自己调整。

[面板设置说明 主要是这个](https://github.com/NimaQu/ss-panel-v3-mod_Uim/wiki/v2ray-%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B)

~~~
没有CDN的域名或者ip;端口（外部链接的);AlterId;协议层;;额外参数(path=/xxxxx|host=xxxx.win|inside_port=10550这个端口内部监听))

// ws 示例
xxxxx.com;10550;16;ws;;path=/xxxxx|host=oxxxx.com

// ws + tls (Caddy 提供)
xxxxx.com;0;16;tls;ws;path=/xxxxx|host=oxxxx.com|inside_port=10550
xxxxx.com;;16;tls;ws;path=/xxxxx|host=oxxxx.com|inside_port=10550



// nat🐔 ws 示例
xxxxx.com;11120;16;ws;;path=/xxxxx|host=oxxxx.com

// nat🐔 ws + tls (Caddy 提供)
xxxxx.com;0;16;tls;ws;path=/xxxxx|host=oxxxx.com|inside_port=10550|outside_port=11120
xxxxx.com;;16;tls;ws;path=/xxxxx|host=oxxxx.com|inside_port=10550|outside_port=11120
~~~

目前的逻辑是

- 如果为外部链接的端口是0或者不填，则默认监听本地127.0.0.1:inside_port
- 如果外部端口设定不是 0或者空，则监听 0.0.0.0:外部设定端口，此端口为所有用户的单端口，此时 inside_port 弃用。
- 默认使用 Caddy 镜像来提供 tls，控制代码不会生成 tls 相关的配置。Caddyfile 可以在Docker/Caddy_V2ray文件夹里面找到。
- Nat🐔，如果要用ws+tls，则需要使用outside_port=xxx，php后端会生成订阅时候，使用outside_port覆盖port部分。 outside_port是内部映射端口，
 建议内网和外网的两个端口数值一致。

tcp 配置：

~~~
xxxxx.com;非0;16;tcp;;
~~~

kcp 支持所有 v2ray 的 type：

- none: 默认值，不进行伪装，发送的数据是没有特征的数据包。

~~~
xxxxx.com;非0;16;kcp;noop;
~~~

- srtp: 伪装成 SRTP 数据包，会被识别为视频通话数据（如 FaceTime）。

~~~
xxxxx.com;非0;16;kcp;srtp;
~~~

- utp: 伪装成 uTP 数据包，会被识别为 BT 下载数据。

~~~
xxxxx.com;非0;16;kcp;utp;
~~~

- wechat-video: 伪装成微信视频通话的数据包。

~~~
xxxxx.com;非0;16;kcp;wechat-video;
~~~

- dtls: 伪装成 DTLS 1.2 数据包。

~~~
xxxxx.com;非0;16;kcp;dtls;
~~~

- wireguard: 伪装成 WireGuard 数据包(并不是真正的 WireGuard 协议) 。

~~~
xxxxx.com;非0;16;kcp;wireguard;
~~~

