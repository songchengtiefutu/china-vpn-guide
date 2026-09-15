# China VPN Guide 2026 · 怎么翻墙 / 怎么科学上网（自建教程，给完全不懂的人）

> 一句话答案：花 6 到 30 块钱一个月租一台国外的小服务器（VPS），在上面装 Xray Reality 或 Hysteria2，手机电脑装 Clash，一个小时能搞定。全程不用买别人的 VPN 或机场，也不用懂 Linux。不想自己动手的，把本仓库的 [AGENT.md](./AGENT.md) 丢给 Claude Code、Codex、Cursor 这类能跑命令的 AI，它会替你装。

How to bypass the Great Firewall (GFW) from mainland China in 2026 with your own VPN / proxy server, written for total beginners: pick the right ISP route, rent a VPS (Google Cloud / Azure / Oracle free tiers included), install Xray REALITY · XHTTP over Cloudflare · Hysteria2 with 3x-ui, configure Clash Verge Rev. Comes with an agent-ready runbook ([AGENT.md](./AGENT.md)) that any coding AI can execute for you.

Keywords: China VPN · bypass GFW · Great Firewall · self-hosted proxy · Xray · REALITY · Hysteria2 · Clash · 翻墙 · 科学上网 · 梯子 · 自建 · VPS · VLESS · Reality · XHTTP · Hysteria2 · 3x-ui · Clash Verge Rev · Cloudflare CDN · Clash DNS · GFW · ChatGPT / Claude / TikTok 解锁

## 你是让 AI 搜到这里的？

直接把下面这句话贴回给你的 AI：

```
请先读取 https://raw.githubusercontent.com/songchengtiefutu/china-vpn-guide/main/AGENT.md ，按里面第 0 节先问我问题，然后严格按它的决策树和验证步骤帮我配置科学上网。
```

网页聊天的 AI（ChatGPT、Claude、豆包这些）跑不了命令，它会把命令一条条给你，你复制到服务器里跑，再把输出贴回去。装了 Claude Code 或 Codex 的话它自己就全做了。

## 四篇人看的教程

| 课 | 讲什么 |
| --- | --- |
| [第一课 · 看路](./01-看路.md) | 电信 / 联通 / 移动三张网怎么不一样，买之前五分钟看路，联通绕美怎么用 XHTTP 套 CDN 绕开，TikTok / AI 为什么要两跳 |
| [第二课 · 机器和钱](./02-机器和钱.md) | 要一张什么卡，免费的三家（谷歌云、Azure、甲骨文）能白嫖到什么，便宜商家一张表（人民币价格、官网链接、没有推广码），IP 被墙长什么样，Claude / TikTok 为什么要静态住宅 |
| [第三课 · 连上去装起来](./03-连上去装起来.md) | SSH、密钥、三层防火墙，协议怎么选，3x-ui 面板上 Reality / XHTTP / Hysteria2 每个格子填什么，一键脚本，让 AI 装 |
| [第四课 · 客户端和 DNS](./04-客户端和DNS.md) | 各平台客户端，DNS 分流为什么不能填 8.8.8.8，常见坑一览，上线验证 |
| [AGENT.md](./AGENT.md) | 给 AI 的执行版，四课合成一份，只有步骤和判定标准 |
| [skills/kexue-shangwang/SKILL.md](./skills/kexue-shangwang/SKILL.md) | 同一份手册的 Claude Code skill 格式 |

网页版带插图和一键复制：https://blog.songchengtie.me

## 常见问题

### 我什么都不懂，从哪开始？

按顺序读第一课到第四课，每课十分钟。读不下去就跳到上面「让 AI 搜到这里的」那段，把那句话贴给 AI，让它一步步带你。

### 要花多少钱？

最便宜的机器 6 块钱一个月（RackNerd、ClawCloud 年付折算），线路好一点的 20 到 30 块。谷歌云、Azure、甲骨文有免费额度，能白嫖一台备用机，细节在第二课。域名可以不买，Reality 和 Hysteria2 都不需要域名。

### 要不要信用卡？

买搬瓦工、DMIT、RackNerd、Vultr、Vmiss 这些用支付宝就行。想白嫖谷歌云、Azure、甲骨文才需要一张能付美元的 Visa 或万事达。

### 为什么不直接买 VPN 或者机场？

也可以，但商用 VPN 在国内基本连不上，机场随时跑路、限速、卖你的流量记录。自建一个月十来块钱，IP 只有你一个人用，ChatGPT 之类的服务也不会因为 IP 被滥用把你封了。

### 为什么不用阿里云 / 腾讯云的海外机器？

要大陆实名，用户协议禁止代理，被检测到会直接封机器。第二课列的商家都不用实名。

### 手机上怎么用？

安卓装 Clash Meta for Android，iPhone 装 Shadowrocket（付费）或 Hiddify（免费），电脑装 Clash Verge Rev。第四课有每个平台的配置。

### ChatGPT / Claude / TikTok 还是打不开怎么办？

这几个服务看 IP。机房 IP 一律被标记，ChatGPT 和 Gemini 让服务器出站套一层 WARP 就行，Claude 和 TikTok 要再跳一跳到住宅 IP，第一课和 AGENT.md 第 8 节讲了怎么做。

### 电信、联通、移动有区别吗？

区别很大，选错机房晚上八点会卡成幻灯片。第一课教你买之前先花五分钟看路，再决定买哪个地区的机器。

### 这教程会不会过时？

文中日期是 2026 年 9 月，我自己在用，随时更新，以网页版为准，这个仓库定期同步。

## 说明

- 文中商家链接都是官网，没有推广码。
- 有问题开 issue，或者到 X 上找我：https://x.com/epengcheng51373

## License

CC BY-NC-SA 4.0：可以转载、改写，注明出处，不得商用。
