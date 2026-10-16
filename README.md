# 🚀 Shadowrocket 自用分流配置（白名单 / 黑名单 两种模式）
> 🆕 **2026-10-16 新增第四份：`懒人版·改版`（`shadowrocket-lazy-revised.conf`）** —— 骨架取自 [johnshall/Shadowrocket-ADBlock-Rules-Forever](https://johnshall.github.io/Shadowrocket-ADBlock-Rules-Forever/lazy.conf) 的**精简版**（原文 346 行 = 注释 204 + 空行 74 + **生效仅 68**，只保留生效行）；**`[Rule]` 段逐行取自 `通用版`**（14 条一致）。**定位：johnshall 的 `[General]` 骨架，但 DNS 已按 owner 的取舍对齐 `通用版`** —— ① 主/直连/备用/代理 DNS 全部改成 **DoT 裸 IP**（`tls://223.5.5.5, tls://1.12.12.12`）；② `fallback-dns-server` 由 `system` 改 DoT（原来会回退到 ISP 明文 DNS）；③ `hijack-dns` 由 `8.8.8.8:53,8.8.4.4:53` 改 **`*:53`**（原值只劫持 Google DNS，用 114/HTTPDNS 的 App 会绕过小火箭）；④ **`ipv6 = false`**（`GEOIP,CN` 不覆盖 IPv6）；⑤ `dns-direct-fallback-proxy` 改 `false`；⑥ 补 `tun-excluded-routes` 的 **`100.64.0.0/10`**（Tailscale 保护）。保留 johnshall 的 `stun-response-ip` / `stun-response-ipv6`（**防 WebRTC 真 IP 泄漏**）、`private-ip-answer`、`icmp-auto-reply`、`block-quic`。详见「注意事项」。
> 🔋 **2026-10-15 通用版：省电精简** —— 按 owner 指令**停用两张 4 万级 DOMAIN 大表**（`Privacy_Domain` 39,916 + `AdvertisingLite_Domain` 37,692 = **77,608 条，占原规则 91%**）⇒ 规则总数 **85,336 → 7,728**、规则集 **13 → 11**。**保留三个小表**（`BlockHttpDNS` 63 / `AdvertisingLite.list` 376 关键词+IP / `Privacy.list` 20）⇒ **关键词/正则级拦截仍有效**，去掉的只是"逐域名大表"。动机：小火箭实测 **2.5–9 %/h** vs Clash **1.0–1.3 %/h**，"规则规模"是配置侧剩下的最大变量。被停用的两行在文件里注释保留，**恢复 = 去掉行首 `# `**。
> 🔄 **2026-10-14 通用版：拦截段"先清后恢复"**。白天为排查朋友那边的耗电，曾按 owner 指令把 4 个拦截规则集（`AdvertisingLite_Domain` 37,692 / `AdvertisingLite` 376 / `Privacy_Domain` 39,916 / `Privacy` 20）临时停用（规则 85,336 → 7,332），停用期间只保留 `BlockHttpDNS`；**当晚 owner 指示全部恢复** ⇒ 拦截段与 2026-10-10 的状态**逐字节一致**，规则总数回到 **85,336**、规则集 **13 个**。**结论：规则规模不是那台手机的耗电主因**（同口径实测 Shadowrocket 85,747 条 ≈ **9.9 %/h** vs Clash iOS 8,428 条 ≈ **0.70 %/h**，差距在客户端实现）。**三份 `[General]` 已对齐**（主 DNS → DoT、删 `dns-fallback-system`、白名单两份补 `GEOIP,CN,...,no-resolve`），并正在做 **`ipv6` 单变量测试：当前三份都是 `ipv6 = false`**（其余修复保留）—— 白天"修泄漏"时同时动了 3 个变量，`ipv6` 那步从未被单独验证。
> 🔧 **2026-10-14 晚续改（修 DNS 泄漏）**：① `GEOIP,CN,DIRECT` → **`GEOIP,CN,DIRECT,no-resolve`** —— 手册 716-718：IP 类规则遇到域名请求会**触发一次本地解析**，缺 `no-resolve` 就是"运营商解析器 + 节点解析器"混在一起的指纹来源（治本）；② `dns-server` / `direct-dns-server` 由阿里/腾讯 **DoH 改成 DoT + 裸 IP**（`tls://223.5.5.5, tls://1.12.12.12`）—— DoH 要先解析 `dns.alidns.com`，手册说这步走系统 DNS；DoT 无 bootstrap，实测首查 54ms vs 121ms；③ 删掉手册里**根本不存在**的 `dns-fallback-system`。
> 🔥 **2026-10-14 深夜：卡顿回归修复**（owner 实测「刷抖音/红果卡，红果最明显」）。手机 DB（23:31）对照定位到两条：
> ① **`no-resolve` 是主因** —— 它让"不在国内域名清单里的域名"跳过 `GEOIP,CN`、直落 `FINAL,PROXY`；实测**红果 77%、抖音 28% 的连接走了代理**，而当时选中的节点是「🇯🇵 日本1」⇒ 国内内容绕日本。旧配置下这些主机 100% DIRECT。⇒ **`no-resolve` 已摘掉**（回到 `GEOIP,CN,DIRECT`），泄漏改由 DoT 那条路兜（需复核 dnsleaktest）。
> ② **字节系被普通 `REJECT` 秒级重试** —— 恢复的广告表拦了字节系主机 125 条，最高重试 **56 次**（4 个主机 ≥20 次）⇒ `BlockHttpDNS.list` 与 `AdvertisingLite.list` 的动作由 `REJECT` 改为 **`REJECT-DROP`**（09-27 的结论：DROP 才治重试风暴）。
> 🔄 **2026-10-10 改名**：`测试版` → **`自用版`**（内容只改名称）；`稳定版` → **`通用版`**（同时**去掉自建规则集**，纯公开上游）。本仓库现为**三份配置**：两份**白名单模式**（`自用版` 作者自用 / `通用版` 对外公开）+ 一份**黑名单模式**（`黑名单·极简版`）。
> 📌 更早还有一份**旧通用版**（`shadowrocket-白名单.通用版.conf`，2026-10-07 删除）与**旧自用版**（`shadowrocket-白名单.conf`，2026-09-30 删除）—— 它们与现在同名的配置**不是同一份文件**，别混淆。

个人自用的 Shadowrocket（小火箭）**规则分流配置**，只含规则，不含任何节点与凭据。

> [!CAUTION]
> 本项目**不提供**任何代理服务器节点、鉴权凭据或订阅服务，仅包含规则分流逻辑。

两种模式：
* **白名单**（`自用版` / `通用版` / `懒人版·改版`）—— 兜底走代理（`FINAL,PROXY`）；明确指定的大陆域名、局域网 IP 与 Apple 服务走 `DIRECT`。
* **黑名单**（`黑名单·极简版`）—— 兜底直连（`FINAL,DIRECT`）；只拦广告/追踪，只把特定境外服务（Google/YouTube/Telegram/GitHub 等）走代理。

---

## 📄 四份配置

| 文件 | 面向 | 内联 / 规则集 | 订阅链接（自建加速，国内可用） |
| :--- | :--- | :--- | :--- |
| [`shadowrocket-白名单.自用版.conf`](./shadowrocket-白名单.自用版.conf) | **作者自用主用**：自建表全挂（`direct-custom` → `bytedance-ad` → `soul-ad` → `reject-custom`）+ 两张广告/隐私 `_Domain` 大表；另有 1 条内联拦截（`www.qchannel01.cn`，排直连表之前） | 4 / 17 | [订阅](https://git.521989.xyz/https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.%E8%87%AA%E7%94%A8%E7%89%88.conf) · [二维码](./qr/selfuse-accelerator.png) |
| [`shadowrocket-白名单.通用版.conf`](./shadowrocket-白名单.通用版.conf) | **公开版 / 给别人用**：**纯公开上游**，一张自建表都不挂（2026-10-10 起）；**拦截段只剩三个小表**（两张 4 万级 DOMAIN 表 2026-10-15 停用） | 3 / 11 | [订阅](https://git.521989.xyz/https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.%E9%80%9A%E7%94%A8%E7%89%88.conf) · [二维码](./qr/generic-accelerator.png) |
| [`shadowrocket-lazy-revised.conf`](./shadowrocket-lazy-revised.conf) | **懒人版·改版**：骨架 = johnshall 懒人配置的**精简版**（只留生效行）；**`[Rule]` 与 `通用版` 逐行一致**（14 条）；`[General]` 是 johnshall 原文，但 **DNS/IPv6 组已对齐 `通用版`**（DoT / `hijack-dns = *:53` / `ipv6 = false`）+ 补了 Tailscale 保护段 | 3 / 11 | [订阅](https://git.521989.xyz/https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/main/shadowrocket-lazy-revised.conf) · [二维码](./qr/lazy-accelerator.png) |
| [`shadowrocket-黑名单.极简版.conf`](./shadowrocket-黑名单.极简版.conf) | **黑名单模式**：`FINAL,DIRECT`；只挂拦广告/隐私的公开表 + 自建 `bytedance-ad` / `direct-custom`，代理段只列境外服务与 `Proxy` 大表 | 1 / 18 | [订阅](https://git.521989.xyz/https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/main/shadowrocket-%E9%BB%91%E5%90%8D%E5%8D%95.%E6%9E%81%E7%AE%80%E7%89%88.conf) · [二维码](./qr/blacklist-accelerator.png) |

> 「内联 / 规则集」里的**规则集**只数 `RULE-SET` / `DOMAIN-SET` 两种引用**类型**（自用版 **17** = 上游 13 + 自建 4；通用版 **11** = 上游 11 + 自建 0；懒人版 **11** = 上游 11 + 自建 0；黑名单版 **18** = 上游 16 + 自建 2）。

* **计数口径**（全部从四份 conf 的 `[Rule]` 段实际数出，不含 `[General]` / `[URL Rewrite]` / `[MITM]`）：
  * **自用版**：内联 **4**（`DOMAIN-SUFFIX,521989.xyz,DIRECT` / `DOMAIN,www.qchannel01.cn,REJECT-DROP` / `GEOIP,CN,DIRECT` / `FINAL,PROXY`；第二条必须在直连表之前）；远端引用行 **17** = `RULE-SET` 13 + `DOMAIN-SET` 4 = 上游 13 + 自建 4（`direct-custom.list` / `bytedance-ad.list` / `soul-ad.list` / `reject-custom.list`）。
  * **通用版**：内联 **3**；远端引用行 **11** = `RULE-SET` 9 + `DOMAIN-SET` 2 = 上游 11 + **自建 0**（规则总数 **7,728**；2026-10-15 停用两张 4 万级 DOMAIN 表，共 77,608 条）。
  * **懒人版·改版**：内联 **3**；远端引用行 **11** = `RULE-SET` 9 + `DOMAIN-SET` 2 = 上游 11 + **自建 0** —— **`[Rule]` 段与 `通用版` 逐行一致**（含 14 条生效行 + 2 行「停用·省电」标记，脚本断言），规则条目实测 **7,725**；差别只在外层骨架与 `[General]` 的取舍（见下）。
  * **黑名单·极简版**：内联 **1**（只有 `FINAL,DIRECT`）；远端引用行 **18** = `RULE-SET` 15 + `DOMAIN-SET` 3 = 上游 16 + 自建 2（`bytedance-ad.list` / `direct-custom.list`）。
* **自用版 与 通用版 的差异**：自用版多挂 4 张自建表（`direct-custom` 直连 + `bytedance-ad` / `soul-ad` / `reject-custom` 拦截）；通用版**纯公开上游**，不带任何自建表 —— 别人的拦截/放行目标与作者不同，挂自建表对别人没意义。（2026-10-14 通用版曾临时停用 4 个广告/隐私拦截集，当晚已恢复，两份拦截段重新一致。）
* ⚠️ **顺序不是小事（2026-10-07 踩过）**：`bytedance-ad.list` 含 `-ad-sign`、`-misc-lf` 等**族关键字**，必须排在 `direct-custom.list` **之后**；它一度被排在前面，于是 `p3-reading-sign.fqnovelpic.com` / `p9-…`（番茄/红果的**图片 CDN**，直连表里专门为它加的 `fqnovelpic.com`）被关键字抢先 `REJECT-DROP` ⇒ **红果/番茄图片加载失败、App 报网络异常**。给别人的通用版已不含自建表，天然没有这个坑。
* `proxy-custom.list` **三份都没有引用**（设计如此：白名单靠 `FINAL,PROXY`、黑名单靠 `FINAL,DIRECT` 兜底；要显式控制再加一行）。
* 通用版保留 `block-quic = all-proxy` / `hijack-dns = *:53` / `udp-policy-not-supported-behaviour = REJECT` 等**强切 TCP**项（去掉自建表时未动）。

删除留档：**旧通用版**（`shadowrocket-白名单.通用版.conf`）已于 **2026-10-07 删除**（与当时的稳定版重复），**旧自用版**（`shadowrocket-白名单.conf`）已于 **2026-09-30 删除**。两者的订阅地址均已作废，不要再导入。⚠️ `shadowrocket-白名单.通用版.conf` 这个**文件名**自 2026-10-10 起被改名后的稳定版复用 —— 现在仓库里这份是**在用的通用版**。

每段为什么这么写、每个实验的来龙去脉与变更历史：见 [`配置说明.md`](./配置说明.md)。

---

## 📱 扫码导入（自建加速）

**自用版（作者自用主用）**

<img src="./qr/selfuse-accelerator.png" width="300" alt="自用版 · 自建加速链接二维码">

**通用版（公开版 / 给别人用）**

<img src="./qr/generic-accelerator.png" width="300" alt="通用版 · 自建加速链接二维码">

**黑名单·极简版（FINAL,DIRECT）**

<img src="./qr/blacklist-accelerator.png" width="300" alt="黑名单极简版 · 自建加速链接二维码">

**懒人版·改版（johnshall 精简骨架 + 通用版规则）**

<img src="./qr/lazy-accelerator.png" width="300" alt="懒人版改版 · 自建加速链接二维码">

手动添加：小火箭 → **配置** → 右上角 `+` → 粘贴上面的订阅链接 → **下载** → 点该配置 → **使用配置**。

> ⚠️ 2026-10-10 改名后，**手机上原本指向 `测试版` 的订阅地址已 404**：请重新添加 `自用版` 的订阅链接（或扫上面的二维码），再「使用配置」。

---

## 🧩 用到的规则集

清单下表**从四份 conf 的 `[Rule]` 段实际解析**（✅ = 该版本挂了这条；下表两列是**白名单模式**的两份；`懒人版·改版` 与 `通用版` 逐行相同）。全部上游规则集来自 **[blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)**，经作者自建加速站 `git.521989.xyz` 拉取：

| 用途 | 规则集（动作） | 自用版 | 通用版 |
| :--- | :--- | :---: | :---: |
| **自建 · 直连** | `direct-custom.list`（`DIRECT`） | ✅ | — |
| **自建 · 拦截** | `bytedance-ad.list`（`REJECT-DROP`，字节系） | ✅ | — |
| **自建 · 拦截** | `soul-ad.list`（`REJECT-DROP`，Soul） | ✅ | — |
| **自建 · 拦截** | `reject-custom.list`（`REJECT-DROP`） | ✅ | — |
| **拦截** | `BlockHttpDNS.list`（**`REJECT-DROP`**，防 App 绕过隧道自行解析；2026-10-14 由 `REJECT` 改） | ✅ | ✅ |
| **拦截** | `AdvertisingLite_Domain.list`（`REJECT-DROP`，域名表 37,692 条） | ✅ | — <br>（2026-10-15 停用·省电） |
| **拦截** | `AdvertisingLite.list`（**`REJECT-DROP`**；2026-10-14 由 `REJECT` 改，防字节系秒级重试） | ✅ | ✅ |
| **拦截** | `Privacy_Domain.list`（`REJECT-DROP`，域名表 39,916 条） | ✅ | — <br>（2026-10-15 停用·省电） |
| **拦截** | `Privacy.list`（`REJECT-DROP`） | ✅ | ✅ |
| **直连** | `Lan.list`（`DIRECT`） | ✅ | ✅ |
| **直连** | `STUN.list`（`DIRECT`） | ✅ | ✅ |
| **直连** | `Apple.list`（`DIRECT`，排在拦截段之前） | ✅ | ✅ |
| **直连** | `Apple_Domain.list`（`DIRECT` 域名表，排在拦截段之前） | ✅ | ✅ |
| **直连** | `China_Domain.list`（`DIRECT` 域名表） | ✅ | ✅ |
| **直连** | `ChinaMedia.list`（`DIRECT`） | ✅ | ✅ |
| **直连** | `Download.list`（`DIRECT`） | ✅ | ✅ |
| **代理** | `GlobalMedia.list`（`PROXY`） | ✅ | ✅ |

* **上游**（blackmatrix7）规则集：自用版 **13 条**、通用版 **11 条**（2026-10-15 通用版停用两张 DOMAIN 大表后不再一致）、懒人版·改版 **11 条**（与通用版同一批）；此外自用版另有 4 张自建表。黑名单版的上游是另一套（见下）。

**黑名单·极简版 与上面两列的差异**（`FINAL,DIRECT` 兜底，只有这里列出的条目不同）：

| 用途 | 规则集（动作） | 黑名单·极简 |
| :--- | :--- | :---: |
| **代理** | `YouTube.list` / `Google.list` / `Facebook.list` / `Pinterest.list` / `Telegram.list` / `Twitter.list` / `Protonmail.list` / `GitHub.list`（`PROXY`） | ✅ |
| **代理** | `Proxy_Domain.list`（域名表）/ `Proxy.list`（`PROXY`） | ✅ |
| **直连·拦截之外的直连** | `Lan.list`（`DIRECT`） | ✅ |
| **直连** | `STUN.list` / `Apple.list` / `Apple_Domain.list` / `China_Domain.list` / `ChinaMedia.list` / `Download.list`（白名单两份都有） | — |
| **代理** | `GlobalMedia.list`（白名单两份都有） | — |
| **自建·拦截** | `reject-custom.list` / `soul-ad.list` | — |
| **兜底** | `FINAL,DIRECT`（白名单两份是 `FINAL,PROXY`） | ✅ |

> 黑名单版**不挂** `STUN/Apple/ChinaMedia/China_Domain/Download/GlobalMedia`：`FINAL,DIRECT` 下国内流量本来就是直连，不需要这些表。
* 两张**广告/隐私域名大表**（`AdvertisingLite_Domain` / `Privacy_Domain`，本地留档副本实测约 **37,692** / **39,916** 条）**自用版挂着；通用版 2026-10-15 起停用**（省电，见文件头）。
* ⚠️ **条目数**：远端规则集的条数随上游 `master` 变动（本文件不放编造数字）；本地可测的自建表 + 两张 `_Domain` 大表规模见 [`配置说明.md`](./配置说明.md) 的「🧩 规则集清单」。

相关链接：

* 上游规则集：<https://github.com/blackmatrix7/ios_rule_script>
* 自建清单：拦截表由手机 db 分析（`--sr-analyze`）生成、直连表同源；ADH 侧走 `adh-custom.txt` 过滤清单订阅：<https://github.com/henrysha1989/shadowrocket-adr-rules>
* 加速站用法：把 GitHub 链接直接接在 `https://git.521989.xyz/` 后面

---

## 🔄 更新规则集

配置里的规则集都是**远端引用**（非快照）。手动全量重拉：**配置 → 点该配置 → 使用配置 / 编译配置**。
自动：**设置 → 自动更新 → 配置 → 自动后台更新**（间隔 1–7 天；每次会重下约 0.2 MB 规则集并重新编译）。

> `设置 → 订阅` 里的自动更新只管**节点订阅**，与规则集无关。

---

## ⚠️ 注意事项

* **依赖**：四份配置的规则集都经作者自建加速站 `git.521989.xyz` 拉取（上游是 raw 的 Fastly 缓存，5 分钟级）。加速站不可达时规则集会全部拉取失败、整份退化成各自的兜底（白名单 `FINAL,PROXY`、黑名单 `FINAL,DIRECT`）；此时把前缀换成自备镜像即可。
* **给别人的是通用版**：通用版 = 纯公开上游（不含作者自建表），拿过去就能用；**拦截段与 `自用版` 一致**（5 个拦截集全开，含广告/隐私/HTTPDNS）；`自用版` 是作者自用主用配置（含自建清单与个人取舍），别人导入可能出现「本来该直连的域掉进代理」之类的差异。
* **懒人版·改版的 DNS / 隐私 / 功耗取舍（2026-10-16）**：`[Rule]` 与 `通用版` 逐行一致，差别在外层骨架与 `[General]` 的 DNS 组。
  * **已对齐 `通用版` 的 7 处**：`dns-server` / `direct-dns-server` / `fallback-dns-server` / `proxy-dns-server` → **DoT 裸 IP**（`tls://223.5.5.5, tls://1.12.12.12`；2026-10-14 实测 DoT 首查 **54ms vs DoH 121ms**，且 DoH 要先解析 `dns.alidns.com`——手册规定这步走系统 DNS）；`fallback-dns-server` 由 **`system` → DoT**（原值 = 回退到 **ISP 明文 DNS**，是最大的泄漏面）；`hijack-dns` 由 `8.8.8.8:53,8.8.4.4:53` → **`*:53`**（原值只劫持 Google DNS，用 `114.114.114.114` 或自带 HTTPDNS 的 App **会绕过小火箭的 DNS**）；**`ipv6 = false`**（`GEOIP,CN` 不覆盖 IPv6；日志实测 26 条 v6 字面量里中国运营商段 11 条有 **7 条绕日本**）；`dns-direct-fallback-proxy = false`；并补 `tun-excluded-routes` 的 **`100.64.0.0/10`**（Tailscale tailnet 保护，`通用版` 2026-10-05 加过）。
  * **保留的 johnshall 隐私项**：`stun-response-ip = 1.1.1.1` / `stun-response-ipv6 = ::1`（**防 WebRTC 真 IP 泄漏**，`通用版` 没有）、`private-ip-answer = true`、`icmp-auto-reply = true`、`block-quic = all-proxy`。
  * **功耗**：规则总量回到 **7,725**（不是 A1 实验的 ≈65,581）、每次更新下载 **0.16 MB**；`ipv6 = false` 只查 A 不查 AAAA ⇒ **DNS 查询量约减半**；DoT 长连接比 DoH 少一层 HTTP/2 与 bootstrap 开销。
  * **还剩下的隐私缺口**：`[Host]` 里 `*.apple.com` / `*.icloud.com = server:system` —— 这两族的 DNS 仍走**系统（ISP）解析**，是唯一的系统 DNS 路径（`通用版` 没有 `[Host]` 段）。另：上游用的是阿里 + 腾讯，传输加密但**解析记录对这两家可见**。
* **懒人版·改版的已知项**（2026-10-16 记录，**按 owner 指令保留 johnshall 原文、未改**）：
  1. **`[MITM]` 盖不住 `g.cn`** —— `[URL Rewrite]` 里有两条 302（`g.cn` / `google.cn` → `google.com`），但骨架的 `hostname = *.google.cn` **不匹配 `www.g.cn`** ⇒ 那条重写在 HTTPS 下解不开密、实际不生效。通用版的写法是 `hostname = google.cn, *.google.cn, g.cn, *.g.cn`（外加 `h2 = true`）。
  2. **`tun-excluded-routes` 里的 `ff02::fb/128`** 与 `ipv6 = false` 矛盾（`通用版` 2026-09-29 已删），本次未动。
  3. **`[Host]` 的两行 `server:system`** —— 见上「还剩下的隐私缺口」。
* **该用哪份**：想「全局走代理、国内直连」= 白名单（`通用版`）；想用 johnshall 的 `[General]` 调优（STUN 应答、ICMP 自动回复等）又要自建规则 = **懒人版·改版**；想「平时全直连、只拦广告 + 少数境外服务走代理」= **黑名单·极简版**。
* **改名提醒**：2026-10-10 起 `测试版` → `自用版`、`稳定版` → `通用版`；旧的订阅地址（含 `测试版`/`稳定版` 的）已失效，请用上表新地址。
* **隐私**：本仓库不含任何服务器地址、密码或订阅凭据。
* **来源**：规则集是远端引用的第三方列表，可用性取决于上游仓库。

### 免责声明

* 本项目仅供个人学习与网络管理自用，不对使用后果承担任何直接、间接或连带责任。
* 请遵守当地法律法规，严禁将本项目规则用于任何违法、违规用途。

---
<p align="center">Maintained by henrysha1989</p>
