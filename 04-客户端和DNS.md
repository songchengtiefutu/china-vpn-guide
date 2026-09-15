# 科学上网第四课：客户端、DNS，和那些坑

> 各平台只推荐一个客户端，为什么老 Clash 不能用。节点和订阅怎么导。DNS 分流为什么不能填 8.8.8.8，不认识的域名为什么要交给代理去解析，实在不行怎么自己搭一个。最后是一张常见坑的表和上线验证。
>
> 网页版：https://blog.songchengtie.me/posts/kexue-shangwang-04/ · 给 AI 的执行版：[AGENT.md](./AGENT.md)


服务端装好了，剩下的都在你自己的设备上。这一课回答四个问题：装哪个客户端，节点怎么导进去，DNS 怎么配才不会莫名其妙卡住，以及出了问题先看哪。

## 01 各平台装哪个客户端？

每个平台只推荐一个，别纠结：

| 平台 | 用这个 | 备注 |
| --- | --- | --- |
| Windows、macOS、Linux | Clash Verge Rev | Mihomo 内核，Reality、XHTTP、Hysteria2 都支持，分流规则最好写 |
| Android | CFA（Clash Meta for Android） | 和 Clash Verge Rev 同一个内核，配置文件通用 |
| iPhone、iPad | Shadowrocket（小火箭） | 美区 Apple ID，收费一次。不想花钱就用 Hiddify，免费 |

两个坑。一是「老 Clash」不能用：Clash for Windows、老版 Clash for Android、Clash 原版内核这一批在 2023 年就停更了，不认识 Reality、XHTTP、Hysteria2 这些新协议，导进去要么报错要么连不上。现在说 Clash 一律指 Mihomo 内核（也叫 Clash Meta），下载的时候看清楚。二是 v2rayNG 我不推荐新手用，功能是全，但我用起来小 bug 不断，订阅时灵时不灵，排查起来比装服务端还费劲。

## 02 节点怎么导进去？

3x-ui 每个入站右边有二维码和复制链接的按钮，手机扫二维码，电脑复制 `vless://` 或者 `hysteria2://` 开头的链接粘进客户端。面板设置里开了订阅的话，还有一条订阅链接，把它加进客户端，以后服务端改了节点，客户端更新订阅就同步。勇哥脚本装完直接把链接和订阅打印在终端里，一样的用法。

Clash 系的客户端，我建议不要只靠订阅，自己维护一个配置文件，节点写死在里面，分流和 DNS 自己控制。下面是一个最小的 Mihomo 配置，三个节点各一个，对应上一课面板上的格子：

```yaml
mixed-port: 7890
mode: rule
ipv6: false

proxies:
  - name: Reality
    type: vless
    server: 1.2.3.4
    port: 443
    uuid: 面板上的 ID
    flow: xtls-rprx-vision
    tls: true
    servername: www.microsoft.com
    client-fingerprint: chrome
    reality-opts:
      public-key: 面板上的公钥
      short-id: 面板上的 Short ID
    network: tcp

  - name: XHTTP-CDN
    type: vless
    server: 你的域名
    port: 2053
    uuid: 面板上的 ID
    tls: true
    servername: 你的域名
    client-fingerprint: chrome
    network: xhttp
    xhttp-opts:
      path: /a9f3k2
      host: 你的域名
      mode: packet-up

  - name: Hy2
    type: hysteria2
    server: 1.2.3.4
    ports: 20000-50000
    password: 服务端 config.yaml 里的密码
    sni: www.bing.com
    skip-cert-verify: true
    up: 100
    down: 300

proxy-groups:
  - name: PROXY
    type: select
    proxies: [Reality, XHTTP-CDN, Hy2]

rules:
  - GEOSITE,cn,DIRECT
  - GEOIP,cn,DIRECT,no-resolve
  - MATCH,PROXY
```

规则只有三条：国内域名直连，国内 IP 直连，剩下的全走代理。`no-resolve` 那个词后面会解释，它和 DNS 卡住直接相关。

## 03 DNS 怎么配才不会卡住？

DNS 是客户端配置里最容易出玄学问题的地方，症状是网页一会儿能开一会儿转圈、订阅更新失败、某些站死活打不开，节点本身却是好的。

先说最常见的错法：把 `nameserver` 填成 `8.8.8.8` 或者 `1.1.1.1`。国内到这两个地址的 UDP 53 要么被污染，要么直接丢包，你的客户端每解析一个域名先等它超时，再退到别的服务器，网页就是这么转圈的。我实测过，订阅链接的域名解析卡在这一步，客户端直接报订阅失败，你会以为是服务端坏了。

正确的思路分三条。国内域名用国内的 DoH（阿里、腾讯），快而且不会被丢。国外域名根本不在本地解析，直接把域名交给代理，让 VPS 那边去解析，这就是 fake-ip 的作用：客户端拿到一个假 IP 先把连接建起来，域名原样传给服务端。规则里遇到需要真实 IP 才能匹配的条目（GEOIP 这类），加 `no-resolve`，不认识的域名不要为了匹配规则去本地查一次，查了就可能被污染、被丢，直接按 MATCH 交给代理。

```yaml
dns:
  enable: true
  ipv6: false
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  fake-ip-filter:
    - '*.lan'
    - '+.local'
    - 'localhost.ptlogin2.qq.com'
    - '+.stun.*.*'
    - '+.msftconnecttest.com'
  default-nameserver:          # 只用来解析下面几个 DoH 的域名，必须填 IP
    - 223.5.5.5
    - 119.29.29.29
  nameserver:                  # 国内域名走这里
    - https://dns.alidns.com/dns-query
    - https://doh.pub/dns-query
  proxy-server-nameserver:     # 解析你自己节点和订阅的域名，也走国内
    - https://dns.alidns.com/dns-query
  nameserver-policy:
    'geosite:cn':
      - https://dns.alidns.com/dns-query
    'geosite:geolocated-!cn':  # 真要在本地解析国外域名时，通过代理去问
      - https://1.1.1.1/dns-query#PROXY
```

最后那行的 `#PROXY` 是 Mihomo 的写法，意思是这条 DoH 请求本身通过 PROXY 这个组发出去，1.1.1.1 就不会在国内被丢了。

国内 DNS 也有乱丢的时候，尤其一些地方的运营商会劫持 53 端口。实在不行就自己搭一个私有 DNS：在 VPS 上装 AdGuard Home 或者 mosdns，开 DoH，域名指过去，然后客户端里 `nameserver` 换成 `https://你的域名/dns-query#PROXY`。这样所有解析都经过你自己的服务器，国内谁也碰不到。多半人用不上，留个后路。

配好之后到 ipleak.net 看一眼，DNS 服务器那一栏不该出现你运营商的地址。

## 04 常见坑一览

装的时候和装完之后会碰到的，按出现频率排：

| 现象 | 原因和处理 |
| --- | --- |
| 命令粘进去报错，`command not found` 或者奇怪的字符 | 复制的时候多了空格、换行，或者把提示符 `$`、`#` 一起复制了。用代码块右上角的复制按钮，别手动选 |
| 客户端完全连不上 | 上一课 01 节的三层防火墙，商家后台、ufw、甲骨文的 iptables，十有八九是其中一层没开 |
| Reality 连不上，服务端日志有 `REALITY: processed invalid connection` | 客户端 SNI 和服务端填的不一样；或者本机时间和服务器差太多，Windows 同步一下时间，VPS 上 `timedatectl` 看一眼 |
| Reality 连上了但只有一部分网站能开 | 目标站选了一个会跳转的，换 `www.microsoft.com` 或者同机房的站 |
| XHTTP 连不上 | Cloudflare 小云朵没开橙；加密模式不是完全（严格）；端口不在 Cloudflare 支持的那几个里；客户端 path、host、mode 和服务端对不上；证书和私钥贴反了 |
| Hysteria2 连不上 | 商家后台没放行 UDP 端口段；ufw 只开了 tcp；客户端没打开跳过证书校验 |
| Hysteria2 跑几分钟就断或者掉速 | 运营商 UDP QoS，换端口段没用的话主用切回 Reality |
| 移动用户 443 连不上，换个端口就好 | 移动静默丢 443，Reality 换 8443 之类的端口，或者走 XHTTP 2053 |
| 订阅更新失败、网页转圈 | 03 节，DNS 填了 8.8.8.8 之类的公共 DNS |
| 导入节点报「不支持的协议」 | 用的是老 Clash，换 Clash Verge Rev 或 CFA；小火箭更新到最新版 |
| 面板打不开 | 面板端口没放行，或者干脆走 SSH 隧道 |
| 关了密码登录之后自己进不去 | 密钥没塞对就关了密码，机器只能在商家后台重装或者用 VNC 救 |
| 用 Windows 记事本改过的配置文件在 VPS 上报错 | 记事本存的是 CRLF 换行，用 nano 在 VPS 上直接改，或者 `sed -i 's/\r$//' 文件名` |
| 某个国内网站打不开或者变慢 | 它被规则分到代理了，加一条 `DOMAIN-SUFFIX,那个域名,DIRECT` |
| 突然全部连不上，国内 ping 不通 VPS，国外通 | IP 被墙了，上一课 04 节，换 IP 或者套 CDN |
| 玩游戏、局域网设备连不上 | fake-ip 把它们的域名也假了，加进 `fake-ip-filter` |

## 05 怎么确认真的连好了？

四步，全过才算：

```bash
# 1. 跑 10 次完整请求，看 P95 有没有超过 1 秒
for i in $(seq 10); do
  curl -x socks5h://127.0.0.1:7890 -o /dev/null -s \
    -w '%{time_total}\n' https://www.gstatic.com/generate_204
done

# 2. 出口 IP 应该是你的 VPS（XHTTP 节点也应该是 VPS，不是 Cloudflare）
curl -x socks5h://127.0.0.1:7890 https://ipinfo.io/json
```

3. ipleak.net：DNS 里不出现你运营商的服务器。
4. browserleaks.com/webrtc：不出现你的真实公网 IP。第一课 07 节那三个开关（IPv6 关掉、fake-ip、WebRTC Control）这时候一起检查。

## 06 之后写什么

这四课把从选机器到连上跑通了。后面单独写几个进阶的：在 VPS 上先跑一个 nginx，根路径放一个真网站，XHTTP 和 Reality 的回落都指过去，谁来看都是个正常站；Cloudflare 优选 IP 怎么筛出落在香港的那批；Reality 和 Hysteria2 在同一台机器上的实测对照；分流规则怎么按自己的习惯写。

## 参考

- Clash Verge Rev — https://github.com/clash-verge-rev/clash-verge-rev
- Clash Meta for Android（CFA） — https://github.com/MetaCubeX/ClashMetaForAndroid
- Hiddify — https://github.com/hiddify/hiddify-app
- Mihomo DNS 配置文档（fake-ip、nameserver-policy、proxy-server-nameserver） — https://wiki.metacubex.one/config/dns/
- Mihomo VLESS 配置（network 支持 xhttp） — https://wiki.metacubex.one/config/proxies/vless/
- Clash DNS 配置完全指南：fake-ip 与 DNS 泄露 — https://tanqingbo.cn/clash-dns-guide/index.html
- AdGuard Home — https://github.com/AdguardTeam/AdGuardHome
- 上一课：连上去，装起来 — ./03-连上去装起来.md
