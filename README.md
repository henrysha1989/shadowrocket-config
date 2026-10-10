# 🚀 Shadowrocket 自用分流配置（白名单 / 黑名单 两种模式）
> 🔄 **2026-10-10 改名**：`测试版` → **`自用版`**（内容只改名称）；`稳定版` → **`通用版`**（同时**去掉自建规则集**，纯公开上游）。本仓库现为**三份配置**：两份**白名单模式**（`自用版` 作者自用 / `通用版` 对外公开）+ 一份**黑名单模式**（`黑名单·极简版`）。
> 📌 更早还有一份**旧通用版**（`shadowrocket-白名单.通用版.conf`，2026-10-07 删除）与**旧自用版**（`shadowrocket-白名单.conf`，2026-09-30 删除）—— 它们与现在同名的配置**不是同一份文件**，别混淆。

个人自用的 Shadowrocket（小火箭）**规则分流配置**，只含规则，不含任何节点与凭据。

> [!CAUTION]
> 本项目**不提供**任何代理服务器节点、鉴权凭据或订阅服务，仅包含规则分流逻辑。

两种模式：
* **白名单**（`自用版` / `通用版`）—— 兜底走代理（`FINAL,PROXY`）；明确指定的大陆域名、局域网 IP 与 Apple 服务走 `DIRECT`。
* **黑名单**（`黑名单·极简版`）—— 兜底直连（`FINAL,DIRECT`）；只拦广告/追踪，只把特定境外服务（Google/YouTube/Telegram/GitHub 等）走代理。

---

## 📄 三份配置

| 文件 | 面向 | 内联 / 规则集 | 订阅链接（自建加速，国内可用） |
| :--- | :--- | :--- | :--- |
| [`shadowrocket-白名单.自用版.conf`](./shadowrocket-白名单.自用版.conf) | **作者自用主用**：自建表全挂（`direct-custom` → `bytedance-ad` → `soul-ad` → `reject-custom`）+ 两张广告/隐私 `_Domain` 大表；另有 1 条内联拦截（`www.qchannel01.cn`，排直连表之前） | 4 / 17 | [订阅](https://git.521989.xyz/https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.%E8%87%AA%E7%94%A8%E7%89%88.conf) · [二维码](./qr/selfuse-accelerator.png) |
| [`shadowrocket-白名单.通用版.conf`](./shadowrocket-白名单.通用版.conf) | **公开版 / 给别人用**：**纯公开上游**，一张自建表都不挂（2026-10-10 起） | 3 / 13 | [订阅](https://git.521989.xyz/https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.%E9%80%9A%E7%94%A8%E7%89%88.conf) · [二维码](./qr/generic-accelerator.png) |
| [`shadowrocket-黑名单.极简版.conf`](./shadowrocket-黑名单.极简版.conf) | **黑名单模式**：`FINAL,DIRECT`；只挂拦广告/隐私的公开表 + 自建 `bytedance-ad` / `direct-custom`，代理段只列境外服务与 `Proxy` 大表 | 1 / 18 | [订阅](https://git.521989.xyz/https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/main/shadowrocket-%E9%BB%91%E5%90%8D%E5%8D%95.%E6%9E%81%E7%AE%80%E7%89%88.conf) · [二维码](./qr/blacklist-accelerator.png) |

> 「内联 / 规则集」里的**规则集**只数 `RULE-SET` / `DOMAIN-SET` 两种引用**类型**（自用版 **17** = 上游 13 + 自建 4；通用版 **13** = 上游 13 + 自建 0；黑名单版 **18** = 上游 16 + 自建 2）。

* **计数口径**（全部从三份 conf 的 `[Rule]` 段实际数出，不含 `[General]` / `[URL Rewrite]` / `[MITM]`）：
  * **自用版**：内联 **4**（`DOMAIN-SUFFIX,521989.xyz,DIRECT` / `DOMAIN,www.qchannel01.cn,REJECT-DROP` / `GEOIP,CN,DIRECT` / `FINAL,PROXY`；第二条必须在直连表之前）；远端引用行 **17** = `RULE-SET` 13 + `DOMAIN-SET` 4 = 上游 13 + 自建 4（`direct-custom.list` / `bytedance-ad.list` / `soul-ad.list` / `reject-custom.list`）。
  * **通用版**：内联 **3**；远端引用行 **13** = `RULE-SET` 9 + `DOMAIN-SET` 4 = 上游 13 + **自建 0**。
  * **黑名单·极简版**：内联 **1**（只有 `FINAL,DIRECT`）；远端引用行 **18** = `RULE-SET` 15 + `DOMAIN-SET` 3 = 上游 16 + 自建 2（`bytedance-ad.list` / `direct-custom.list`）。
* **自用版 与 通用版 的差异**：自用版多挂 4 张自建表（`direct-custom` 直连 + `bytedance-ad` / `soul-ad` / `reject-custom` 拦截）；通用版**纯公开上游**，不带任何自建表 —— 别人的拦截/放行目标与作者不同，挂自建表对别人没意义。
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

手动添加：小火箭 → **配置** → 右上角 `+` → 粘贴上面的订阅链接 → **下载** → 点该配置 → **使用配置**。

> ⚠️ 2026-10-10 改名后，**手机上原本指向 `测试版` 的订阅地址已 404**：请重新添加 `自用版` 的订阅链接（或扫上面的二维码），再「使用配置」。

---

## 🧩 用到的规则集

清单下表**从三份 conf 的 `[Rule]` 段实际解析**（✅ = 该版本挂了这条；下表两列是**白名单模式**的两份）。全部上游规则集来自 **[blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)**，经作者自建加速站 `git.521989.xyz` 拉取：

| 用途 | 规则集（动作） | 自用版 | 通用版 |
| :--- | :--- | :---: | :---: |
| **自建 · 直连** | `direct-custom.list`（`DIRECT`） | ✅ | — |
| **自建 · 拦截** | `bytedance-ad.list`（`REJECT-DROP`，字节系） | ✅ | — |
| **自建 · 拦截** | `soul-ad.list`（`REJECT-DROP`，Soul） | ✅ | — |
| **自建 · 拦截** | `reject-custom.list`（`REJECT-DROP`） | ✅ | — |
| **拦截** | `BlockHttpDNS.list`（`REJECT`） | ✅ | ✅ |
| **拦截** | `AdvertisingLite_Domain.list`（`REJECT-DROP`，域名表） | ✅ | ✅ |
| **拦截** | `AdvertisingLite.list`（`REJECT`） | ✅ | ✅ |
| **拦截** | `Privacy_Domain.list`（`REJECT-DROP`，域名表） | ✅ | ✅ |
| **拦截** | `Privacy.list`（`REJECT-DROP`） | ✅ | ✅ |
| **直连** | `Lan.list`（`DIRECT`） | ✅ | ✅ |
| **直连** | `STUN.list`（`DIRECT`） | ✅ | ✅ |
| **直连** | `Apple.list`（`DIRECT`，排在拦截段之前） | ✅ | ✅ |
| **直连** | `Apple_Domain.list`（`DIRECT` 域名表，排在拦截段之前） | ✅ | ✅ |
| **直连** | `China_Domain.list`（`DIRECT` 域名表） | ✅ | ✅ |
| **直连** | `ChinaMedia.list`（`DIRECT`） | ✅ | ✅ |
| **直连** | `Download.list`（`DIRECT`） | ✅ | ✅ |
| **代理** | `GlobalMedia.list`（`PROXY`） | ✅ | ✅ |

* **上游**（blackmatrix7）规则集：**两份白名单配置都是 13 条、完全一致**；差异只在自建表（自用版 4 条 / 通用版 0 条）。黑名单版的上游是另一套（见下）。

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
* 两张**广告/隐私域名大表**（`AdvertisingLite_Domain` / `Privacy_Domain`，本地留档副本实测约 **37,692** / **39,916** 条）**两份都挂**。
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

* **依赖**：三份配置的规则集都经作者自建加速站 `git.521989.xyz` 拉取（上游是 raw 的 Fastly 缓存，5 分钟级）。加速站不可达时规则集会全部拉取失败、整份退化成各自的兜底（白名单 `FINAL,PROXY`、黑名单 `FINAL,DIRECT`）；此时把前缀换成自备镜像即可。
* **给别人的是通用版**：通用版 = 纯公开上游（不含作者自建表），拿过去就能用；`自用版` 是作者自用主用配置（含自建清单与个人取舍），别人导入可能出现「本来该直连的域掉进代理」之类的差异。
* **该用哪份**：想「全局走代理、国内直连」= 白名单（`通用版`）；想「平时全直连、只拦广告 + 少数境外服务走代理」= **黑名单·极简版**。
* **改名提醒**：2026-10-10 起 `测试版` → `自用版`、`稳定版` → `通用版`；旧的订阅地址（含 `测试版`/`稳定版` 的）已失效，请用上表新地址。
* **隐私**：本仓库不含任何服务器地址、密码或订阅凭据。
* **来源**：规则集是远端引用的第三方列表，可用性取决于上游仓库。

### 免责声明

* 本项目仅供个人学习与网络管理自用，不对使用后果承担任何直接、间接或连带责任。
* 请遵守当地法律法规，严禁将本项目规则用于任何违法、违规用途。

---
<p align="center">Maintained by henrysha1989</p>
