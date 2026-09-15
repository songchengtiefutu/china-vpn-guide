<!-- 科学上网 · AI 执行版 · v2026-09-15 · https://blog.songchengtie.me/agent/kexue-shangwang.md · 人看的原文：https://blog.songchengtie.me/posts/kexue-shangwang-01.md -->

# 给 AI 的说明

你在替一位中国大陆用户从零配置科学上网。按顺序执行，每步先验证再进下一步，验证不过就把输出给用户看。硬规则（第 1 节）不许为了省事跳过。服务器命令通过 SSH 跑（`ssh root@<ip> '<命令>'`）；用户是密钥 + 普通用户登录的（谷歌云、甲骨文、Azure）先 `sudo -i`。**不要让用户把 root 密码贴进对话**，只有密码的先做第 4 节密钥。你自己跑不了命令（网页聊天）就把命令一条条给用户、让他贴回输出。用户问"为什么"再读人看的四篇：`https://blog.songchengtie.me/posts/kexue-shangwang-0{1,2,3,4}.md`。

## 0. 先问用户

1. 运营商（家里宽带）：移动 / 电信 / 联通。
2. 地区：沿海 / 内陆或北方。
3. 目标：A 日常（Google、YouTube、GitHub）；B 解锁看 IP 的服务（ChatGPT、Gemini、Claude、TikTok）。
4. 现在能不能打开 github.com 和 google.com。都不能 = 没梯子，按第 2.3 节办。
5. 已有：VPS（IP、系统、登录方式、商家）、域名（是否已在 Cloudflare）、住宅 SOCKS5。
6. 付款：有没有 Visa / 万事达；只有支付宝也行。
7. Hysteria2：电信、联通默认装；移动装了当备用；公司 / 校园网掐 UDP 不装。

## 1. 硬规则

- 不买腾讯云、阿里云、华为云等要大陆实名的云（协议禁代理、主动检测封机、绑身份证）。
- 不信测评站的数字，只信用户自己晚高峰测的。
- VPS 到手先查 IP 是否被墙（第 3 节），被墙的不装 Reality / Hy2。
- 关 IPv6（网卡、Clash `ipv6: false`、VPS）。
- Clash DNS 必须 fake-ip，nameserver 只填国内 DoH，不填 8.8.8.8 / 1.1.1.1 明文；IP 规则加 `no-resolve`。
- 客户端只推 Clash Verge Rev、Clash Meta for Android、Shadowrocket（iOS 付费）、Hiddify（iOS 免费）。老 Clash 不支持新协议；不推 v2rayNG。
- 浏览器装 WebRTC Control。
- 端口：Reality 443（移动不通改 8443），XHTTP 2053，Hy2 UDP 20000-50000。
- Hy2 `bandwidth` 必填，填实际带宽 80%–90%，不许往大填。
- XHTTP `mode: packet-up`，服务器地址填域名（官方 CDN）；优选 IP 只在用户要求时做，并说明一两周要重跑。
- 目标 B 分档：ChatGPT / Gemini → VPS 出站套 WARP（第 8.1）；Claude → 只能两跳到 ISP 静态住宅（第 8.2），WARP 无效；TikTok → 住宅 IP + 手机拔掉国内 SIM 卡。机房 IP（含搬瓦工、DMIT）一律会被标记。不让用户从国内直连住宅 SOCKS5。
- 沿海 + 移动不买精品线路。
- 改完 Xray 配置先 `xray run -test -config /usr/local/etc/xray/config.json` 再重启。
- 没确认密钥能进，不关密码登录。443 已被占用（`ss -tlnp | grep ':443 '`）就停下问。
- 用户要面板 → 第 10 节装 3x-ui，入站让用户自己在面板建。

## 2. 挑机器

### 2.1 机房

| 运营商 | 首选 | 次选 | 不买 |
| --- | --- | --- | --- |
| 移动 | 香港 → 新加坡 → 日本 | 美西 | — |
| 电信 | 日本、香港 | 新加坡 | 美国普通线路 |
| 联通 | 美西 + Hy2 Brutal | 任意机房 + XHTTP 套 Cloudflare | 香港、新加坡直连（唯一例外：ClawCloud 新加坡区挂在联通出口上，可用） |

总体：东南亚线路移动最好，联通最好的是美西，电信偏日本。内陆 / 北方 + 电信 / 联通 + 晚高峰丢包 > 5% 才考虑精品线路（CN2 GIA / CMIN2 / 9929）。目标 B 第一跳只看距离：华南香港，华东日本或香港，北方联通日本或美西。

### 2.2 商家（官网，无推广；价格随时变）

- 先试 / 换 IP 方便：Vultr（vultr.com，3.5 刀/月起，按小时，支付宝）。
- IP 干净：Linode（linode.com，5 刀/月，只收卡）、DigitalOcean（4 刀/月，只收卡）。
- 便宜：RackNerd（racknerd.com，11 刀/年，美国，IP 常脏）、ClawCloud（claw.cloud，70 元/年促销，1 核 1 G 500 G/月；联通 → 新加坡区，电信 / 移动 → 日本、香港区，IP 脏）、BuyVM（buyvm.net，2 刀/月不限流量，美西 + Hy2 候选）。
- 精品：搬瓦工（bandwagonhost.com，CN2 GIA 50 刀/年起）、DMIT（dmit.io）、Vmiss（vmiss.com，CN2 GIA / 9929 / CMIN2 / 软银）、野草云（yecaoyun.com，香港精品 BGP，25 元/月 300 G 双向计）。新商家年付别多。
- 免费只当备用：谷歌云（300 刀试用 90 天，绑卡 + 可退押金；长期免费 e2-micro 只在 us-west1 / us-central1 / us-east1，**网络层级选 Standard** 每区每月 200 GB 免费出站，Premium 只 1 GB 且不含中国）；Azure（普通账号绑卡 200 刀 / 30 天，到期删机；学生账号 100 刀 / 12 个月不绑卡；B1s 都极难抢）；甲骨文（ARM 2 OCPU / 12 GB，注册看运气，开机常 Out of capacity，镜像自带 iptables 只放 22）。

### 2.3 用户没梯子

只推国内能开官网、收支付宝的商家（搬瓦工、DMIT、Vmiss、RackNerd、Vultr）。安装脚本你在 VPS 上跑。客户端安装包你在 VPS 上 `wget` 后让用户 `scp` 拉回，或给 `https://ghproxy.net/` 前缀的镜像地址。

### 2.4 买前看路

去程：本机 `mtr -rwc 50 <商家 test IP>`（Windows `tracert -d`），亚洲目标出现美国节点 = 绕美。回程：`curl -4 ip.sb` 拿用户 IP，到商家 looking glass 跑 traceroute，联通看到 `219.158.x.x` 前有美国节点 = 绕美；没有 looking glass 就买按小时的测一小时退。ping 参考（广东移动，北方加 20–30 ms）：香港 10–20，新加坡 40–60，东京 60–90，美西 130–170；超一倍 = 绕路。21:00–23:00 复测，丢包 > 5% 换机房。可选：bgp.tools 看机房上游，和 CMI（AS58453）/ CTG（AS23764）/ 联通（AS4837、AS10099）有 peering 才可能直连。

## 3. 到手先查 IP

让用户到 https://ping.pe 跑 VPS IP 的 ping 和 tcping 443、22：国内全红国外全绿 = IP 被墙 → 有域名只装 XHTTP，没域名换 IP（Vultr 删了重开，其他找客服）；ping 通某端口不通 = 端口被封 → 换端口换协议；全绿继续。

## 4. SSH 密钥、系统、防火墙、BBR

用户本机（要他自己输一次密码）：

```powershell
# Windows
ssh-keygen -t ed25519 -N '""' -f $env:USERPROFILE\.ssh\id_ed25519
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh root@<ip> "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

```bash
# macOS / Linux
ssh-keygen -t ed25519 -N '' -f ~/.ssh/id_ed25519 && ssh-copy-id root@<ip>
```

验证 `ssh -o BatchMode=yes root@<ip> 'echo ok'`。然后 VPS 上：

```bash
apt update && apt upgrade -y && apt install -y curl wget openssl ufw
timedatectl set-ntp true            # Reality 对时间差敏感
ufw allow 22/tcp && ufw allow 443/tcp
ufw allow 2053/tcp                  # 装 XHTTP 时
ufw allow 20000:50000/udp           # 装 Hy2 时
ufw --force enable
grep -q tcp_congestion_control /etc/sysctl.conf || printf 'net.core.default_qdisc=fq\nnet.ipv4.tcp_congestion_control=bbr\n' >> /etc/sysctl.conf
sysctl -p && sysctl net.ipv4.tcp_congestion_control   # 应输出 bbr
```

商家后台还有一层，你做不了，让用户放行同样的端口：谷歌云 = VPC 防火墙规则，甲骨文 = 安全列表，Azure = 网络安全组；Vultr / Linode / DO / 搬瓦工 / DMIT / RackNerd / Vmiss 默认全开。甲骨文 Ubuntu 再加：

```bash
iptables -I INPUT -p tcp --dport 443 -j ACCEPT; iptables -I INPUT -p tcp --dport 2053 -j ACCEPT; iptables -I INPUT -p udp --dport 20000:50000 -j ACCEPT
apt install -y iptables-persistent && netfilter-persistent save
```

## 5. Xray：VLESS + Reality（IP 没被墙就装）

```bash
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
UUID=$(xray uuid); SID=$(openssl rand -hex 8); xray x25519 | tee /root/reality.keys; echo "UUID=$UUID SID=$SID"
curl -sI --http2 https://www.microsoft.com | head -1     # 目标站验证，应为 HTTP/2 200
```

`xray x25519` 输出 `PrivateKey`（服务端）和 `Password`（客户端公钥，旧版叫 PublicKey）。写 `/usr/local/etc/xray/config.json`：

```json
{
  "log": { "loglevel": "warning" },
  "inbounds": [
    {
      "tag": "reality-in", "listen": "0.0.0.0", "port": 443, "protocol": "vless",
      "settings": { "clients": [ { "id": "<UUID>", "flow": "xtls-rprx-vision" } ], "decryption": "none" },
      "streamSettings": {
        "network": "tcp", "security": "reality",
        "realitySettings": { "target": "www.microsoft.com:443", "serverNames": [ "www.microsoft.com" ], "privateKey": "<PRIVATE_KEY>", "shortIds": [ "<SID>" ] }
      },
      "sniffing": { "enabled": true, "destOverride": [ "http", "tls", "quic" ] }
    }
  ],
  "outbounds": [ { "tag": "direct", "protocol": "freedom" }, { "tag": "block", "protocol": "blackhole" } ]
}
```

```bash
xray run -test -config /usr/local/etc/xray/config.json && systemctl enable --now xray && systemctl restart xray
systemctl is-active xray; ss -tlnp | grep ':443 '; journalctl -u xray -n 20 --no-pager
```

## 6. Xray：XHTTP + TLS 套 Cloudflare（有域名才装）

让用户在 Cloudflare 做三件事：DNS A 记录指向 VPS 并开橙色云朵；SSL/TLS 加密模式 完全（严格）；SSL/TLS → 源服务器 → 创建证书，把证书和私钥存到 VPS 的 `/usr/local/etc/xray/cf.crt`、`cf.key`（`chmod 600 cf.key`）。验证 `dig +short <域名>` 返回 Cloudflare 的 IP。然后在 `inbounds` 里追加（`<PATH>` = `/` + 8 位随机小写字母数字）：

```json
{
  "tag": "xhttp-in", "listen": "0.0.0.0", "port": 2053, "protocol": "vless",
  "settings": { "clients": [ { "id": "<UUID>" } ], "decryption": "none" },
  "streamSettings": {
    "network": "xhttp", "security": "tls",
    "tlsSettings": { "certificates": [ { "certificateFile": "/usr/local/etc/xray/cf.crt", "keyFile": "/usr/local/etc/xray/cf.key" } ] },
    "xhttpSettings": { "path": "<PATH>", "host": "<DOMAIN>", "mode": "packet-up" }
  },
  "sniffing": { "enabled": true, "destOverride": [ "http", "tls", "quic" ] }
}
```

测试并重启同第 5 节。验证 `curl -sk https://<DOMAIN>:2053/ -o /dev/null -w '%{http_code}\n'` 返回 400 / 404（000 不算）。

## 7. Hysteria2

```bash
bash <(curl -fsSL https://get.hy2.sh/)
openssl req -x509 -nodes -newkey ec:<(openssl ecparam -name prime256v1) -keyout /etc/hysteria/key.pem -out /etc/hysteria/cert.pem -subj "/CN=www.bing.com" -days 3650
chown hysteria:hysteria /etc/hysteria/*.pem
HY2PW=$(openssl rand -base64 24); echo "HY2PW=$HY2PW"
curl -o /dev/null -s -w '%{speed_download}\n' http://speedtest.tele2.net/100MB.zip   # 用户不知道带宽时估一下
```

`/etc/hysteria/config.yaml`：

```yaml
listen: :20000-50000
tls: { cert: /etc/hysteria/cert.pem, key: /etc/hysteria/key.pem }
auth: { type: password, password: <HY2PW> }
bandwidth: { up: <UP> mbps, down: <DOWN> mbps }
masquerade: { type: proxy, proxy: { url: https://www.bing.com/, rewriteHost: true } }
```

```bash
systemctl enable --now hysteria-server && systemctl restart hysteria-server
systemctl is-active hysteria-server; journalctl -u hysteria-server -n 20 --no-pager   # 应有 server up and running
```

上线后让用户连续下载 30–60 分钟，掉速或断流 → 主用改 Reality。

## 8. 目标 B

### 8.1 ChatGPT / Gemini：VPS 出站套 WARP

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/yonggekkk/warp-yg/main/CFwarp.sh)   # 选 WireGuard 模式，拿私钥、地址、peer 公钥
```

Xray `outbounds` 追加 `wireguard` 出站（tag `warp`），`routing.rules` 把 `geosite:openai`、`geosite:google-gemini` 指到 `warp`（3x-ui：出站页加 WireGuard，路由页加规则）。验证：经代理 `curl https://chatgpt.com/cdn-cgi/trace` 的 `ip=` 是 Cloudflare 段。

### 8.2 Claude / TikTok：两跳到 ISP 静态住宅

在第 9 节配置里追加：

```yaml
proxies:
  - { name: US-residential, type: socks5, server: <host>, port: <port>, username: <user>, password: <pw>, dialer-proxy: Reality }
proxy-groups:
  - { name: UNLOCK, type: select, proxies: [US-residential] }
rules:   # 放在 MATCH 之前
  - DOMAIN-SUFFIX,tiktok.com,UNLOCK
  - DOMAIN-SUFFIX,tiktokv.com,UNLOCK
  - DOMAIN-SUFFIX,anthropic.com,UNLOCK
  - DOMAIN-SUFFIX,claude.ai,UNLOCK
```

住宅 IP 质量：https://ping0.cc 看类型（住宅 / ISP，原生）和风控值，https://scamalytics.com 看欺诈分，高分退。`curl -x socks5h://127.0.0.1:7890 https://ipinfo.io/json` 的 `country` 为 `US`、`org` 不含 hosting / datacenter / cloud。TikTok 提醒用户拔掉国内 SIM。

## 9. 输出给用户

分享链接：

```
vless://<UUID>@<ip>:443?encryption=none&flow=xtls-rprx-vision&security=reality&sni=www.microsoft.com&fp=chrome&pbk=<PASSWORD>&sid=<SID>&type=tcp#Reality
vless://<UUID>@<DOMAIN>:2053?encryption=none&security=tls&sni=<DOMAIN>&fp=chrome&type=xhttp&host=<DOMAIN>&path=<PATH_urlencode>&mode=packet-up#XHTTP-CDN
hysteria2://<HY2PW_urlencode>@<ip>:20000-50000?sni=www.bing.com&insecure=1#Hy2
```

完整 Mihomo 配置（没装的节点删掉；DNS 段不许改；移动用户 Reality 放第一，电信 / 联通装了 Hy2 的 Hy2 放第一）：

```yaml
mixed-port: 7890
mode: rule
ipv6: false
proxies:
  - { name: Reality, type: vless, server: <ip>, port: 443, uuid: <UUID>, flow: xtls-rprx-vision, tls: true, servername: www.microsoft.com, client-fingerprint: chrome, network: tcp, reality-opts: { public-key: <PASSWORD>, short-id: <SID> } }
  - { name: XHTTP-CDN, type: vless, server: <DOMAIN>, port: 2053, uuid: <UUID>, tls: true, servername: <DOMAIN>, client-fingerprint: chrome, network: xhttp, xhttp-opts: { path: <PATH>, host: <DOMAIN>, mode: packet-up } }
  - { name: Hy2, type: hysteria2, server: <ip>, ports: 20000-50000, password: <HY2PW>, sni: www.bing.com, skip-cert-verify: true, up: <UP>, down: <DOWN> }
proxy-groups:
  - { name: PROXY, type: select, proxies: [Reality, XHTTP-CDN, Hy2] }
dns:
  enable: true
  ipv6: false
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  fake-ip-filter: ['*.lan', '+.local', 'localhost.ptlogin2.qq.com', '+.stun.*.*', '+.msftconnecttest.com']
  default-nameserver: [223.5.5.5, 119.29.29.29]
  nameserver: [https://dns.alidns.com/dns-query, https://doh.pub/dns-query]
  proxy-server-nameserver: [https://dns.alidns.com/dns-query]
  nameserver-policy:
    'geosite:cn': [https://dns.alidns.com/dns-query]
    'geosite:geolocated-!cn': ['https://1.1.1.1/dns-query#PROXY']
rules:
  - GEOSITE,cn,DIRECT
  - GEOIP,cn,DIRECT,no-resolve
  - MATCH,PROXY
```

把 UUID、密钥、密码整理成一段给用户保存，删掉 `/root/reality.keys`。客户端下载：Clash Verge Rev `github.com/clash-verge-rev/clash-verge-rev/releases`，CFA `github.com/MetaCubeX/ClashMetaForAndroid/releases`，iOS 美区 App Store 搜 Shadowrocket 或 Hiddify。

验证清单（全过才算完成）：

```bash
for i in $(seq 10); do curl -x socks5h://127.0.0.1:7890 -o /dev/null -s -w '%{time_total}\n' https://www.gstatic.com/generate_204; done   # P95 < 1 s
curl -x socks5h://127.0.0.1:7890 https://ipinfo.io/json   # 出口 = VPS IP（XHTTP 也是 VPS，不是 Cloudflare）
```

再让用户开 https://ipleak.net（DNS 里没有运营商地址）、https://browserleaks.com/webrtc（没有真实 IP）、https://test-ipv6.com（无 IPv6）；目标 B 按第 8 节验证。

## 10. 用户要面板：3x-ui

不做第 5、6 节（两个 Xray 抢端口）：

```bash
bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)
```

把打印的端口、路径、账号密码转给用户。访问推荐 SSH 隧道：本机 `ssh -L <端口>:127.0.0.1:<端口> root@<ip>` 后开 `http://127.0.0.1:<端口>/<路径>/`，面板端口不用对公网开。入站让用户按第三课原文 03 节自己建；第 7 节 Hy2 可照装。

## 11. 排查

| 现象 | 依次查 |
| --- | --- |
| 完全连不上 | 三层防火墙 → `systemctl is-active xray` → `journalctl -u xray -n 50` → UUID / 公钥 / SID 复制是否完整 |
| 日志 `REALITY: processed invalid connection` | 客户端 SNI ≠ serverNames；时间不同步 |
| Reality 只有部分网站能开 | 目标站会跳转，换 `www.microsoft.com` |
| XHTTP 连不上 | `dig` 不是 Cloudflare IP → 加密模式不是完全（严格）→ 端口不在 443 / 2053 / 2083 / 2087 / 2096 / 8443 → path / host / mode 不一致 → 证书私钥贴反 |
| Hy2 连不上 / 跑几分钟断 | 后台 UDP 没放行 → ufw 少 udp → 客户端没 `skip-cert-verify` → 运营商 UDP QoS，切 Reality |
| 移动 443 不通换端口就好 | Reality 改 8443 或走 XHTTP 2053 |
| 订阅失败、网页转圈、节点是好的 | DNS 填了 8.8.8.8 / 1.1.1.1 明文；按第 9 节 dns 段重写；国内 DNS 也丢的在 VPS 装 AdGuard Home 开 DoH，`nameserver` 改 `https://<域名>/dns-query#PROXY` |
| 导入报不支持的协议 | 老 Clash → 换 Clash Verge Rev / CFA；Shadowrocket 更新 |
| 能连但慢 | 晚高峰 → 第 2.4 看路 → 联通绕美走 XHTTP → 换 Hy2 → 换机房 |
| 突然全断，国内 ping 不通国外通 | IP 被墙 → 换 IP 或只留 XHTTP |
| 目标 B 打不开 / 零播放 | TikTok 拔 SIM → 第 8 节纯净度 → 换住宅 IP → `dialer-proxy` 生效（出口应是住宅 IP） |
| Windows 记事本改过的文件在 VPS 报错 | `sed -i 's/\r$//' 文件名` |

判断不了的事问用户，不要猜：运营商、是否沿海、目标 A / B、能不能开 GitHub、商家、IP 是否被墙、有没有域名、实际带宽、443 是否被占、有没有住宅 SOCKS5。