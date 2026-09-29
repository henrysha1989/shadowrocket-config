# 🚀 Shadowrocket 自用分流配置（白名单模式）

个人自用的 Shadowrocket（小火箭）规则分流配置，只包含**规则分流逻辑**，不含任何节点与凭据。

> [!CAUTION]
> 本项目**不提供**任何代理服务器节点、鉴权凭据或订阅服务，仅包含规则分流逻辑。

---

## 📄 配置说明

* **模式**：白名单 —— 兜底走代理（`FINAL, PROXY`）。明确指定中国大陆域名、局域网 IP 与常见国内服务走 `DIRECT`，其余未知流量默认 `PROXY`。
* **文件**：三份，规则逻辑同源 ——
  * [`shadowrocket-白名单.conf`](./shadowrocket-白名单.conf)：**作者自用版（默认）**。43 条内联规则、24 条远端规则集，含自建 DoH 与自建加速站。
  * [`shadowrocket-白名单.通用版.conf`](./shadowrocket-白名单.通用版.conf)：**通用版**。25 条内联规则、20 条远端规则集；公共 DNS（阿里 / 腾讯 DoT）+ 自建加速站镜像。
  * [`shadowrocket-白名单.测试版.conf`](./shadowrocket-白名单.测试版.conf)：**测试通道**，作者自用。以自用版为基线，只叠加**当前正在验证的改动**（见下面 **🧪 测试版** 一节）。
  自用版 / 通用版是**纯规则、无注释**；测试版带 **5 行说明头** —— 测试通道里「在测什么」必须跟着文件走，导入后一眼能看见。
  「N 条内联规则」= `[Rule]` 段里直接写死在文件里的规则行；「N 条远端规则集」= `RULE-SET` / `DOMAIN-SET` 引用的远端列表。
* **规模**：自用版 4 个段（`[General]` / `[Rule]` / `[URL Rewrite]` / `[MITM]`）；通用版 3 个段（**无 `[MITM]`**，它不带假响应）。

**结构**：按「从具体到宽泛」分块 —— ① 精确直连（镜像站 / 局域网设备 / Apple 系统底层与证书链）→ ② 拦截 → ③ 代理（**`AppleProxy` 例外必须排在直连大表之前**，否则永不生效）→ ④ 直连大表（`Apple` / `China_Domain` / `ChinaMedia` / `Download`）→ `GEOIP,CN` → `FINAL,PROXY`。每块为什么在那个位置、每条特殊规则的来龙去脉，见 [`配置说明.md`](./配置说明.md)。

规则集全部来自 [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)（`Apple`（域名 + 规则两半）/ `ChinaMedia` / `GlobalMedia` / `AdvertisingLite` / `Privacy` / `AppleProxy` 等）—— **通用版 20 条、自用版 21 条**；自用版另有 **3 条**自用列表（`*-custom.list`，通用版没有）。**两份配置的公共规则集都经自建加速站 `git.521989.xyz` 拉取**。

> [!IMPORTANT]
> **通用版不再"完全不依赖作者服务"**：自 **2026-09-29** 起，两份配置的公共规则集**都经自建加速站 `git.521989.xyz` 拉取**（此前通用版走公共 CDN jsDelivr）。如果你的网络访问不了该加速站，20 条规则集会全部拉取失败、整份退化成 `FINAL,PROXY`；此时把规则集前缀换成自备镜像即可（形如 `https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/...`）。

> [!NOTE]
> **该选哪份？** 想复刻作者整套（自建 DoH / 自用清单 / 假响应）→ 看 **自用版**；只想要公开上游规则集、不要作者的自用清单（`*-custom.list`）→ 选 **通用版**。两份的逐条差异见 [`配置说明.md` 的「🆚 自用版 vs 通用版」](./配置说明.md#-自用版-vs-通用版)。

---

## 🧩 两个版本怎么选

| 文件 | 适合谁 | 规则集来源 | 远端加载量 |
| :--- | :--- | :--- | :--- |
| `shadowrocket-白名单.conf` | 作者本人 / 想复刻整套的人 | 自建加速站 · 24 条（21 公开 + 3 作者自用） | 约 **8.75 万** 条 |
| `shadowrocket-白名单.通用版.conf` | 只要公开上游规则集的人 | 自建加速站 · 20 条（全部公开上游） | 约 **0.94 万** 条 |

通用版的取舍：**不装两张最大的广告域名表**（`AdvertisingLite_Domain` 37,692 + `Privacy_Domain` 39,916），只保留轻量拦截（`BlockHttpDNS` + `AdvertisingLite.list` + `Privacy.list`，合计 469 条）。换来的是远端加载量 8.75 万 → 0.94 万、编译与更新开销大幅下降；代价是拦截覆盖面变小（广告拦截不再由大表兜底）。**分流判定不受影响** —— 这两张表只做 `REJECT`，不参与直连/代理判定。

---

## 🔗 订阅链接

> [!TIP]
> **国内网络环境**推荐优先用 **自建加速** 或 **jsDelivr CDN** 链接，连接更稳定。

### 自用版（默认）

* [原始 GitHub Raw 链接](https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/refs/heads/main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.conf)
* [⚡ jsDelivr CDN 加速链接](https://fastly.jsdelivr.net/gh/henrysha1989/shadowrocket-config@main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.conf)
* [🚀 自建加速链接（国内可用）](https://git.521989.xyz/https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.conf)

### 通用版（公开上游规则集）

* [原始 GitHub Raw 链接](https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/refs/heads/main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.%E9%80%9A%E7%94%A8%E7%89%88.conf)
* [⚡ jsDelivr CDN 加速链接](https://fastly.jsdelivr.net/gh/henrysha1989/shadowrocket-config@main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.%E9%80%9A%E7%94%A8%E7%89%88.conf)
* [🚀 自建加速链接（国内可用）](https://git.521989.xyz/https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.%E9%80%9A%E7%94%A8%E7%89%88.conf)

### 📱 扫码导入

用 Shadowrocket 的「扫一扫」扫下面的码，可直接添加配置。

#### 自用版（默认）

**🚀 自建加速（推荐，国内可用）**

<img src="./qr/accelerator.png" width="380" alt="自用版 · 自建加速链接二维码">

**⚡ jsDelivr CDN**

<img src="./qr/jsdelivr.png" width="380" alt="自用版 · jsDelivr 链接二维码">

#### 通用版（公开上游规则集）

**🚀 自建加速（推荐，国内可用）**

<img src="./qr/generic-accelerator.png" width="380" alt="通用版 · 自建加速链接二维码">

**⚡ jsDelivr CDN**

<img src="./qr/generic-jsdelivr.png" width="380" alt="通用版 · jsDelivr 链接二维码">

> 原始 Raw 链接不单独出码 —— 国内直连不稳定，需要用的时候复制上面的文字链接即可。

---

## 🧪 测试版（正在验证的改动）

> [!NOTE]
> [`shadowrocket-白名单.测试版.conf`](./shadowrocket-白名单.测试版.conf) 是**作者自用的测试通道**：
> 基线永远跟着**自用版**走，只叠加**当前正在验证的改动**。**订阅链接与二维码固定不变**，内容随实验更新 ——
> 扫一次码长期有效，随时看到「这一版在测什么」（配置文件开头的 5 行说明头）。

**当前实验（2026-09-27）· T1–T6**

| 编号 | 改动 | 状态 |
| :--- | :--- | :--- |
| T1 | `dns-fallback-system = true` | 保留 |
| T3 | `dns-direct-system` **回退为 `false`** | 实测 `true` 时直连域名解析绕过 ADH（系统 DNS = 192.168.3.1/运营商），失去 DNS 层拦截；已回退 |
| T2 | `[Rule]` 最前放行 `dig.bdurl.net`、`dns.weixin.qq.com.cn` | ✅ 生效（前者原来 12,186 次 REJECT/7.7h → 4 次 DIRECT） |
| T5 | 红果 `*reading-ad.qznovelvod.com` 改**假响应** `REJECT-ARRAY` + MITM | ✅ 生效（单分钟 65,430 次 REJECT → 0；不再弹广告） |
| T6 | `mon11-misc-lf.fqnovel.com` 同样假响应 | 观察中 |

> **假响应为什么需要模块**：`REJECT-ARRAY`（返回 200 + 空 JSON 数组）只有在**该域名被 HTTPS 解密**时才生效。
> 为避免主机名清单塞进主配置，解密清单放在 **[`ad-mitm.module`](./ad-mitm.module)**，用 `[MITM] %APPEND%` 追加。
> 详见下面「🧩 可选模块」一节。

<details>
<summary>T1 的两项分别是什么（点开）</summary>

| 参数 | 含义 |
| :--- | :--- |
| `dns-fallback-system` | 覆写 DNS 失败或查询超过 2s 时，回退到**系统 DNS**，而不是两个 DoT 服务器（`tls://223.5.5.5`、`tls://1.12.12.12`） |
| `dns-direct-system` | **直连的域名类规则**的解析交给谁：`true` = 系统 DNS（快但可能绕过 ADH）；`false` = 走隧道内自建 DoH（ADH 可见） |

</details>

* 规模不变：**43 条内联规则 / 24 条规则集**，与自用版一致。
* 为什么测、生效范围、代价与判读方法：见 [`配置说明.md`](./配置说明.md) 的「🧪 测试版 T1」。
* ⚠️ `dns-fallback-system` 是**手册未收录的隐式键**（写法沿用作者原配置）；想用有文档、UI 里可见的等价写法，
  应写成 `fallback-dns-server = system`。

**🚀 自建加速（推荐，国内可用）**

* [自建加速链接](https://git.521989.xyz/https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.%E6%B5%8B%E8%AF%95%E7%89%88.conf)

<img src="./qr/test-accelerator.png" width="380" alt="测试版 · 自建加速链接二维码">

**⚡ jsDelivr CDN**

* [jsDelivr CDN 加速链接](https://fastly.jsdelivr.net/gh/henrysha1989/shadowrocket-config@main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.%E6%B5%8B%E8%AF%95%E7%89%88.conf)

<img src="./qr/test-jsdelivr.png" width="380" alt="测试版 · jsDelivr 链接二维码">

> **↩️ 回退**：重新导入 [`shadowrocket-白名单.conf`](./shadowrocket-白名单.conf)（自用版）即可；或把上面两行改回 `false`。

---

## 🧩 可选模块：`ad-mitm.module`（假响应所需的解密清单）

**什么时候需要它**：某些广告 / 埋点 SDK 在「连接被拒」后会**死循环重试**（实测红果在弹广告时
单分钟产生 6.5 万次 TCP 重连、约 1,100 次/秒）。对这类域名，单纯 `REJECT` 只会喂大重试风暴；
正确做法是返回**假响应**（`REJECT-ARRAY` = 200 + 空 JSON 数组、`REJECT-VIDEO` = 空白 MP4 等），
让 App 认为「请求成功、没有广告」而放弃重试。

> ⚠️ **假响应必须配合 HTTPS 解密**（MITM）才生效；否则退化为普通拒绝。

**订阅方式（一次性）**：小火箭 → **配置 → 模块 → 右上角 `➕` → 填链接 → 下载**

```
https://git.521989.xyz/https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/main/ad-mitm.module
```

**设计原则**：

* **默认走 reject 规则**——普通广告域直接 `REJECT` 就够了，App 不重试就不用管。
* **只有「拒绝会引发死循环重试」的域名才进模块**——每个解密域名都吃 Network Extension 内存
  （iOS 15+ 上限 50 MB）与 CPU，没必要为普通域名付这个成本。
* 模块里 `hostname = %APPEND% ...` 的 `%APPEND%` **不能删**：不加会覆盖主配置 `[MITM]` 的
  `hostname`，并影响其他模块。
* 多数模块**仅在「全局路由 = 配置」时生效**。

**以后新增域名**：只改 `ad-mitm.module` 这一个文件的 `hostname` 行（三份主配置都不用动）。

**已知清单**（截至 2026-09-27）：

| 域名 | 用途 | 现象 |
| :--- | :--- | :--- |
| `*reading-ad.qznovelvod.com` | 红果短剧广告 | 被拒后单分钟 65,430 次重连 |
| `mon11-misc-lf.fqnovel.com` | 番茄小说（同厂）监控上报 | 被拒后持续 ~109 次/分 |
| `*-ad-sign.byteimg.com` | 字节广告签名域 | `p{N}` 编号轮换，逐个封拦不住 |
| `i.snssdk.com` | 字节 SDK 域 | ADH 从未放行、手机侧 25 次/分稳定重试 |

### 模块怎么更新（2026-09-27 实测结论）

**改了 `ad-mitm.module` 之后，手机上怎么拿到新版？** 按可靠度排序：

| 方式 | 路径 | 何时拿到 |
| :--- | :--- | :--- |
| **自动后台更新**（推荐常开） | 设置 → 模块 → 自动后台更新 + 更新提醒 | 按间隔，**最小 1 天** |
| **手动强制重拉** | 配置 → 本配置文件 `ⓘ` → **使用配置 / 编译配置** | 立即（手册：使远程资源重新拉取） |
| 删除重加 | 配置 → 模块 → 左滑删除 → `➕` 重新填链接 | 立即（应急） |

**为什么不能「秒级」生效**（三条链路都验证过，不是 GitHub 的问题）：

* 加速站/raw 的 `cache-control: max-age=300`（5 分钟），**加 `?参数` 也绕不过**（实测 `x-cache` 仍为 HIT）。
* **Shadowrocket 把模块缓存在本地**：只有「新建模块」那一刻打网络，之后**重载配置不一定会重拉模块**。
* 参考同类项目：blackmatrix7 模块靠客户端 1–7 天自动更新；yfamilys 的模块服务器直接设 `max-age=43200`（12 小时）
  —— **专职维护去广告模块的项目也不指望秒级**。

> 结论：GitHub 这条链路承担的是**集中维护 + 固定订阅分发**，不是「秒级推送」。
> 新增域名后，**手机侧主动重拉一次**即可；日常靠 1 天自动更新兜底。

**维护提醒**：改完模块顺手核对 `#!hostname=` 计数与 hostname 行的域名数一致（踩过）。

---

## 📖 使用方法

> 手机和屏幕在同一视线内时，直接用 Shadowrocket 的「扫一扫」扫上面的二维码最快；不在手边就按下面手动添加。

1. 打开 **Shadowrocket** 客户端。
2. 进入底部导航的 **配置** 页面。
3. 点击右上角 **`+`**。
4. 在弹出框的 **URL** 栏粘贴上方任一订阅链接，点击 **下载**。
5. 下载完成后，点击该配置并选择 **使用配置**。

### 🔄 更新远程规则集

配置里的规则集（自用版 24 条 = blackmatrix7 21 条 + 自用 3 条；通用版 20 条，全部 blackmatrix7）都是**远端引用**，不是快照。Shadowrocket 会把它们缓存起来，需要主动触发才会重新下载。

**手动**

| 方式 | 路径 | 效果 |
| :--- | :--- | :--- |
| 单条重拉 | 配置 → 点配置文件 → 编辑配置 → **规则集 URL** → 点某条地址 | 只重新拉取这一条（会弹状态提示），**不会**重新编译配置 |
| 全量重拉 | 配置 → 点配置文件 → **使用配置** / **编译配置** | 重新拉取全部规则集并重新编译 |

「规则集 URL」页每条地址后面显示「已成功下载数 / 总数」，✅ 成功、❎ 失败。自用版那 3 条是：

* `reject-custom.list` → 拦截
* `proxy-custom.list` → 代理
* `direct-custom.list` → 直连

**自动**

**设置 → 自动更新 → 配置** → 打开 **自动后台更新**，**更新间隔** 可选 1–7 天。每次自动更新都会连带更新当前配置所用的**全部远程规则集与脚本** —— 这才是"自动更新规则集"的开关。

> [!NOTE]
> **`设置 → 订阅` 里那组「打开时更新 / 自动后台更新」只管节点订阅**（周期 1–24 小时），与规则集无关，别指望它刷新规则集。

> [!WARNING]
> 远程配置自动更新时会走一次「更新配置」，**覆盖你在手机上对该配置做的本地修改**。若想定期更新规则集、又要保住本地改动，官方两条办法：① 在配置纯文本里删掉或注释掉 `update-url =`；② 改用扩展配置。

> [!TIP]
> **间隔建议 7 天**：每次更新要重下约 1.6 MB 规则集、重编译 8 万余条规则，这是本配置唯一确定的周期性网络突发，拉到 7 天可直接砍到 1/7。需要立刻刷新时按上表手动做一次即可。

#### ⏱ 各条列表的源头节奏

| 规则集 | 上游更新 | 中间缓存 | 手机上最快可见 |
| :--- | :--- | :--- | :--- |
| 自用 3 条（`*-custom.list`，仅自用版） | 每 4 小时自动（本机管线 → 仓库 Action） | 自建加速站 / Fastly `max-age=300` | 约 5 分钟后 |
| blackmatrix7 20/21 条 | 上游不定期 | **两版都是自建加速站**（Fastly `max-age=300`） | 约 5 分钟后 |

> 自用版 24 条规则集统一走自建加速站，通用版 20 条同样走自建加速站，两版缓存都是 5 分钟级。代价是每次刷新都要从自建加速站回源（自用版约 1.6 MB、通用版约 0.15 MB；`git.521989.xyz` 由本人维护）。**通用版不再走公共 CDN jsDelivr。**

---

## ⚠️ 注意事项

> [!WARNING]
> * **CDN 缓存延迟**：配置**文件本身**的 jsDelivr 订阅链接存在数小时缓存（两版的**规则集**都已改走自建加速站，不受影响）。若刚推送完没用上，请手动更新配置或临时改用 Raw / 自建加速链接。
> * **隐私与安全**：本仓库不含任何服务器地址、密码或订阅凭据；若你在自己的 fork 里添加节点，请勿提交明文凭据。
> * **规则来源**：规则集是远端引用的第三方列表，其可用性取决于上游仓库；**两版都依赖作者的自建加速站 `git.521989.xyz`** 作为镜像。

### 免责声明

* 本项目仅供个人学习与网络管理自用，不对使用后果承担任何直接、间接或连带责任。
* 请遵守当地法律法规，严禁将本项目规则用于任何违法、违规用途。

---

## 📝 更新日志

* **2026-09-25**：结构由 8 段精简为 4 块；规则 71 → 69 条（合并 `101.226.0.0/16` + `101.227.0.0/16`）；修正 5 条被代理列表抢走的死规则；移除 AppleProxy 列表；关闭 MITM。
* **2026-09-25（二改）**：20 条 blackmatrix7 规则集由 `cdn.jsdelivr.net`（`@master` 分支缓存最长约 12 小时）改为自建加速站 `git.521989.xyz`（上游 `max-age=300`），使 23 条规则集统一为 5 分钟级新鲜度。
* **2026-09-25（三改）**：以手机导出配置为基线重新对齐（规则内容无误），**清空配置文件里的全部注释**，说明文字统一移入 [`配置说明.md`](./配置说明.md)。
* **2026-09-25（四改）**：新增 **通用版** [`shadowrocket-白名单.通用版.conf`](./shadowrocket-白名单.通用版.conf)（换公共 DoH、20 条规则集改走公共 CDN jsDelivr、摘掉作者自用清单与 `521989.xyz`）及其扫码二维码，并把「哪些是作者个人定制项」写进 [`配置说明.md`](./配置说明.md)。
* **2026-09-25（五改）**：**清理"伪腾讯信令"IP 段** —— 删除 `100.128.0.0/9`、`172.32.0.0/11`、`172.72.0.0/13`、`172.80.0.0/12`、`172.96.0.0/11`、`172.128.0.0/9`、`204.141.0.0/16`、`30.0.0.0/8`（逐一查证均为境外真实公网段：T-Mobile USA / Microsoft / Akamai / NTT / 美国国防部），并删除冗余的 `push-apple.com.akadns.net`。规则 69 → **60 条**（通用版 65 → 56）；打洞改由端口规则 `DST-PORT,3478` + 两条 UDP 规则承担，境外流量交给 `GEOIP,CN` → `FINAL,PROXY`。
* **2026-09-25（六改）**：**补齐 Apple 规则集** —— ④ 段新增 `Apple.list`（KEYWORD/UA/IP-CIDR 那一半），与原有的 `Apple_Domain.list`（1560 条域名）配对，格式与 `AdvertisingLite` / `Privacy` 的两半用法一致。规则 60 → **61 条**、规则集 23 → **24**（通用版 57 / 21）。
* **2026-09-25（七改）**：**证书链 / 探测调整** —— `letsencrypt.org` → `lencr.org`（Let's Encrypt 已停用 OCSP、CRL 迁至 `x1/x2.c.lencr.org`，旧域名实测 `ENOTFOUND`）；删除 `detectportal.firefox.com`（Firefox 专用）与 `connectivitycheck.gstatic.com`（Android / Chrome 探测）。规则 61 → **59 条**（通用版 57 → 55）。
* **2026-09-26（八改）**：新增**测试通道** [`shadowrocket-白名单.测试版.conf`](./shadowrocket-白名单.测试版.conf) —— 以自用版为基线 + **T1：`dns-direct-system` / `dns-fallback-system` 两项改 `true`**；配套两张二维码（自建加速 / jsDelivr）。**链接与二维码自此固定**，以后测试内容直接更新这一个文件。动机与判读方法见下面「🧪 测试版」。
* **2026-09-29（九改）**：**通用版瘦身**，并统一镜像 ——
  * 去掉两张最大的广告域名表 `AdvertisingLite_Domain`（37,692）+ `Privacy_Domain`（39,916）；保留 `BlockHttpDNS` / `AdvertisingLite.list` / `Privacy.list` 三条轻量拦截。远端加载量 **8.75 万 → 0.94 万**。
  * 规则集镜像由公共 CDN jsDelivr **改回自建加速站** `git.521989.xyz`（作者决定以自用为准），因此**通用版不再是"完全不依赖作者服务"**，见上面 `IMPORTANT` 提示。
  * 补回**系统 / 时间 / 证书直连段**（`DST-PORT,123`、`DST-PORT,5223`、`IP-CIDR,17.0.0.0/8`、`push.apple.com`、OCSP / CRL、digicert 链等）与 `Apple.list`；新增 `AppleProxy` 列表并**排在直连大表之前**（原来排在 `Apple_Domain` 之后，44 条永不生效）。
  * 去掉 `[MITM]` 段（原为 `enable = false`，无功能影响）。文件 93 → **76 行**、规则集 21 → **20 条**。
  * README 里的规则计数口径统一为「`[Rule]` 段内联规则行 / 远端规则集数」，并据此修正了自用版的旧数字（59 → **43**）。

---

## ❤️ 致谢

* [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)
* [Surge 官方文档](https://manual.nssurge.com/)（Shadowrocket 语法与参数的主要参考）

---
<p align="center">Maintained by henrysha1989</p>
