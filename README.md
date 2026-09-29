# 🚀 Shadowrocket 自用分流配置（白名单模式）

个人自用的 Shadowrocket（小火箭）规则分流配置，只包含**规则分流逻辑**，不含任何节点与凭据。

> [!CAUTION]
> 本项目**不提供**任何代理服务器节点、鉴权凭据或订阅服务，仅包含规则分流逻辑。

---

## 📄 配置说明

* **模式**：白名单 —— 兜底走代理（`FINAL, PROXY`）。明确指定中国大陆域名、局域网 IP 与常见国内服务走 `DIRECT`，其余未知流量默认 `PROXY`。
* **文件**：三份，规则逻辑同源 ——
  * [`shadowrocket-白名单.conf`](./shadowrocket-白名单.conf)：**作者自用版（默认）**。27 条内联规则、22 条远端规则集（含 3 条自建清单），含自建 DoH 与自建加速站。
  * [`shadowrocket-白名单.通用版.conf`](./shadowrocket-白名单.通用版.conf)：**通用版**。3 条内联规则、19 条远端规则集；公共 DNS（阿里 / 腾讯 DoT）+ 自建加速站镜像；**Apple 全直连**（无 `AppleProxy`）；**无系统/证书直连段**。
  * [`shadowrocket-白名单.测试版.conf`](./shadowrocket-白名单.测试版.conf)：**测试通道**，作者自用。以自用版为基线，只叠加**当前正在验证的改动**（见下面 **🧪 测试版** 一节）。
  **自用版带 17 行说明头**（含设计要点与 DNS 说明），导入后一眼能看见；**通用版是纯规则、无注释**；**测试版带 20 行说明头**（写清「这一版在测什么」）。
  「N 条内联规则」= `[Rule]` 段里直接写死在文件里的规则行；「N 条远端规则集」= `RULE-SET` / `DOMAIN-SET` 引用的远端列表。
* **规模**：两份都是 4 个段（`[General]` / `[Rule]` / `[URL Rewrite]` / `[MITM]`）。

**结构**：按「从具体到宽泛」分块 —— 精确直连（镜像站 / 局域网设备；自用版另含 iOS 系统底层与证书链）→ 拦截 → 代理 → 直连大表（`Apple` / `China_Domain` / `ChinaMedia` / `Download`）→ `GEOIP,CN` → `FINAL,PROXY`。两份的差别：**自用版把拦截放在第 1 位**、系统与证书链直连段保留 9 行（共 27 条内联）；**通用版把 Apple 两半提到拦截与代理之前**（= Apple 全直连、无 `AppleProxy`），并**删掉了整个系统/证书直连段**（只剩 3 条内联）。每块为什么在那个位置，见 [`配置说明.md`](./配置说明.md)。

规则集全部来自 [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)（`Apple`（域名 + 规则两半）/ `ChinaMedia` / `GlobalMedia` / `AdvertisingLite` / `Privacy` 等）—— **两份各 19 条**；自用版另有 **3 条**自建列表（`*-custom.list`，通用版没有）。**两份配置的公共规则集都经自建加速站 `git.521989.xyz` 拉取**。

> [!IMPORTANT]
> **通用版不再"完全不依赖作者服务"**：自 **2026-09-29** 起，两份配置的公共规则集**都经自建加速站 `git.521989.xyz` 拉取**（此前通用版走公共 CDN jsDelivr）。如果你的网络访问不了该加速站，19 条规则集会全部拉取失败、整份退化成 `FINAL,PROXY`；此时把规则集前缀换成自备镜像即可（形如 `https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/...`）。

> [!NOTE]
> **该选哪份？** 想复刻作者整套（自建 DoH / 自用清单 / 假响应）→ 看 **自用版**；只想要公开上游规则集、不要作者的自用清单（`*-custom.list`）→ 选 **通用版**。两份的逐条差异见 [`配置说明.md` 的「🆚 自用版 vs 通用版」](./配置说明.md#-自用版-vs-通用版)。

---

## 🧩 两个版本怎么选

| 文件 | 适合谁 | 规则集来源 | 远端加载量 |
| :--- | :--- | :--- | :--- |
| `shadowrocket-白名单.conf` | 作者本人 / 想复刻整套的人 | 自建加速站 · 22 条（19 公开 + 3 作者自用） | 约 **0.99 万** 条 |
| `shadowrocket-白名单.通用版.conf` | 只要公开上游规则集的人 | 自建加速站 · 19 条（全部公开上游） | 约 **0.93 万** 条 |

**两份现在都不装那两张最大的广告域名表**（`AdvertisingLite_Domain` 37,692 + `Privacy_Domain` 39,916；通用版 09-29 先删，自用版同日 B 组跟进）。拦截只保留轻量集合：通用版 `BlockHttpDNS` + `AdvertisingLite.list` + `Privacy.list`（469 条），自用版另加自建的 `reject-custom.list`。通用版另有两点不同：**不用 `AppleProxy`**，并把 Apple 两半提到拦截之前 ⇒ **所有 Apple 流量走直连**。换来的是两份的远端加载量都降到 1 万条以内、编译与更新开销大幅下降；代价是拦截覆盖面变小 —— ⚠️ 但**不等于拦截失效**：自用版实测 45% 的 REJECT 事件来自**内联关键字规则**（`reading-ad` 一条就占 266,360 次），不受删表影响。**分流判定完全不受影响** —— 去掉的是 `REJECT` 域名表与一张 Apple 例外表，不是直连/代理判据。

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
> 扫一次码长期有效，随时看到「这一版在测什么」（配置文件开头的说明头）。

**当前实验（2026-09-29）· DNS 走自建 DoH（对照自用版的本地 DoT）**

| | 自用版（基线） | 测试版 |
| :--- | :--- | :--- |
| `dns-server` | `tls://223.5.5.5, tls://1.12.12.12`（本地 DoT，阿里 + 腾讯） | `https://doh.521989.xyz:8443/dns-query/iphone17pm#no-h3`（**自建 DoH**，回源本机 ADH） |

**其余 100% 相同** —— 逐行 diff 只差 `dns-server` 这一行（去掉注释后）。基线 = **2026-09-29 B 组精简版的自用版**（103 行 / 22 规则集 / 27 内联）。

**为什么测**：自用版 2026-09-27 为了「快」把 DNS 从自建 DoH 换成本地 DoT，代价是**手机解析不再经过 AdGuard Home** —— ADH querylog 看不到手机，DNS 层拦截与「漏网之鱼」采集对手机这一端失效。换回 DoH 就是做 A/B 对照。

**判读（跑一天后对比三项）**

| # | 指标 | 预期 |
| :--- | :--- | :--- |
| ① | ADH querylog 里 iPhone 的查询量 | 从「几乎为 0」明显上升 |
| ② | 手机侧 REJECT 事件数 | 下降（ADH 直接答 `0.0.0.0`，App 不再反复重试） |
| ③ | DNS 解析延迟 | DoH 多一次 TLS 往返，注意冷启动与首连 |

**工具**：先 `node ops/adh/fetch-qlog.mjs` 刷 querylog，再 `python3 ops/shadowrocket/ab-dns-visibility.py`。

**回退**：重新导入自用版；或把 `dns-server` 改回 `tls://223.5.5.5, tls://1.12.12.12`。

> [!WARNING]
> 本版依赖作者自建 DoH（`doh.521989.xyz:8443`）——**别人导入会解析失败/证书不认，等于断网**。这是「自用通道」，不要给别人用。

> **上一轮实验（T1–T6）已全部收口**：T2（放行 `dig.bdurl.net` / `dns.weixin.qq.com.cn`）已并入自用版；T3 的 `dns-direct-system` 回退为 `false`；T1 的 `dns-fallback-system` 已按 A 组删除；T5/T6 的**假响应 + MITM 方案未采用**（自用版头部已写明不需要 MITM，`ad-mitm.module` 目前无消费方）。

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

配置里的规则集（自用版 22 条 = blackmatrix7 19 条 + 自用 3 条；通用版 19 条，全部 blackmatrix7）都是**远端引用**，不是快照。Shadowrocket 会把它们缓存起来，需要主动触发才会重新下载。

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
| blackmatrix7 19/19 条 | 上游不定期 | **两版都是自建加速站**（Fastly `max-age=300`） | 约 5 分钟后 |

> 自用版 22 条规则集统一走自建加速站，通用版 19 条同样走自建加速站，两版缓存都是 5 分钟级。代价是每次刷新都要从自建加速站回源（自用版约 0.2 MB、通用版约 0.13 MB；`git.521989.xyz` 由本人维护）。**通用版不再走公共 CDN jsDelivr。**

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
* **2026-09-29（十改）**：**通用版去 AppleProxy + Apple 全直连**，并再瘦一轮 ——
  * 删除 `AppleProxy.list`（PROXY）；`Apple.list` + `Apple_Domain.list` **前移到拦截与代理之前** ⇒ 所有 Apple 流量走直连（此前 `GlobalMedia` 的 `USER-AGENT,AppleTV*` / `com.apple.tv*` 会把 Apple TV 抢去代理）。
  * 删掉 13 条与 `Apple_Domain` / `China_Domain` 重复的内联规则（`ocsp.apple.com`、`updates*.cdn-apple.com`、`digicert.com`、`push.apple.com` 等）；它们原先还在挡 `AppleProxy`，现在不需要了。内联规则 25 → **12 条**、规则集 20 → **19 条**、文件 76 → **63 行**。
  * 保留 6 条无表覆盖的内联域名（`pool.ntp.org` + 5 家第三方 CA 吊销）与 3 条端口/网段（`DST-PORT,123` / `DST-PORT,5223` / `IP-CIDR,17.0.0.0/8`）。
  * ⚠️ 副作用（已实测，仅 1 条）：Apple 表里的 `.crashlytics.com` 同时也在拦截表里，Apple 前移后它由 REJECT 变成 DIRECT。
* **2026-09-29（十一改）**：**通用版删掉整个系统/证书直连段** ——
  * 删掉 `DST-PORT,123`（NTP）、`DST-PORT,5223`（APNs）、`IP-CIDR,17.0.0.0/8`、`pool.ntp.org`，以及 `digicert.cn` / `entrust.net` / `lencr.org` / `identrust.com` / `sectigo.com` 五家第三方 CA 吊销域名。
  * 前 4 条**删了不影响**：iOS 的 NTP / APNs 主机都是 `*.apple.com`，已被前移的 `Apple_Domain` 覆盖，照样直连。
  * **唯一实际损失**：那 5 家第三方 CA 的 OCSP/CRL 查询改走代理（TLS 握手多一次往返；代理异常时可能让证书校验软失败）。要找回只需加回 5 行 `DOMAIN-SUFFIX,<ca>,DIRECT`。
  * 结果：内联规则 12 → **3 条**（只剩 `521989.xyz` 放行 + `GEOIP,CN` + `FINAL,PROXY`），文件 63 → **52 行**，规则集仍 19 条。
* **2026-09-29（十二改）**：**通用版补回 `[MITM]` 段**，`hostname` 补全为 `google.cn, *.google.cn, g.cn, *.g.cn`（覆盖 `[URL Rewrite]` 那两条正则能匹配的全部主机），与自用版**逐字节一致**。⚠️ `enable = false` 未改 —— 要让 HTTPS 的 `www.google.cn` / `google.cn/search` 也被改写，必须自己改成 `true` 并安装信任 CA 证书；只加 host 列表**不会**改变现有行为。
  * 另：`skip-proxy` 14 → **6 条**、`tun-excluded-routes` 14 → **8 条**（去掉规则集已覆盖的域名与永不路由的 TEST-NET / 废弃 6to4 段 / 与 `ipv6 = false` 矛盾的 `ff02::fb/128`）。
* **2026-09-29（十三改）**：**自用版 A 组零行为变化精简**（自建 3 条清单原样保留）——
  * 删掉系统/证书段里 **13 条与 `Apple_Domain` / `China_Domain` 重复**的内联（`time.asia.apple.com`、`time.windows.com`、`push.apple.com`、`mesu.apple.com`、`appldnld.apple.com`、`updates-http.cdn-apple.com`、`updates.cdn-apple.com`、`gdmf.apple.com`、`gg.apple.com`、`gs.apple.com`、`ocsp.apple.com`、`crl.apple.com`、`digicert.com`）⇒ 该系统段 22 → **9 行**。
  * `skip-proxy` 9 → **6 条**；`tun-excluded-routes` 14 → **8 条**。
  * 删 2 个手册查不到的键（`dns-fallback-system`、`udp-policy-fulfilled-by-proxy`）⇒ `[General]` 13 → **11 键**。
  * 去掉 `DOMAIN,dig.bdurl.net,DIRECT,no-resolve` 与 `DOMAIN,dns.weixin.qq.com.cn,DIRECT,no-resolve` 上的 `no-resolve`（**只对 IP 类规则有意义**，写在 `DOMAIN` 上是死参数）。
  * 顺手修正说明头第 4 点：原文"`dns-server` 不管直连还是走代理的域都由它解析"与手册 546-548 相反（**覆写只作用于直连类域名**）。
  * 结果：内联 43 → **30 条**、文件 121 → **107 行**；**远端加载量不变（87,511 条）**，行为一字未变。
* **2026-09-29（十四改）**：**自用版 B 组（逐条实证核对后执行）**——
  * **删两张最大广告域名表**（`AdvertisingLite_Domain` + `Privacy_Domain`）⇒ 远端加载量 **87,511 → 9,900（-88.7%）**。实测损失：全库 593,355 条 REJECT 里，可归因到现行拦截表的 48,039 条中有 **43,288（90%）来自这两张表**、会被放行；**但 45% 的 REJECT（269,127 条）是 `DOMAIN-KEYWORD` 命中，其中 `reading-ad` 一条就 266,360 次**，不受删表影响。
  * 删 3 条国内快路径 `IP-CIDR`（`101.226/15`、`180.111/16`、`183.131/16`）—— **65.6 万条事件里命中 0 次**。
  * 补 `hijack-dns = *:53`（与通用版对齐；实测全库无 `:53` 流量，属防御性设置）。
  * ⚠️ **`AND,((PROTOCOL,UDP),(DST-PORT,8500-8800))` 保留**：手册只收录单端口，但**实测命中 8,138 次** ⇒ 范围写法生效，**不是死规则**（`DST-PORT,3478` 2 次 / `12070` 29 次 / `123` 111 次 / `5223` 13 次同样生效）。
  * **不做 B5（Apple 前移）**：Apple 与拦截段重叠的 5 条域名**全部只被 `AdvertisingLite_Domain` 拦**，该表已删 ⇒ 前移与不前移等价；UA 冲突只影响 Apple TV。
  * 结果：内联 30 → **27 条**、规则集 24 → **22 条**、文件 107 → **103 行**。
* **2026-09-29（十五改）**：**测试版重新定位为「DNS 走自建 DoH」实验通道**——
  * 基线换成 **B 组精简版的自用版**（103 行 / 22 规则集 / 27 内联），去掉注释后逐行 diff **只差 `dns-server` 一行**：自用版 `tls://223.5.5.5, tls://1.12.12.12`，测试版 `https://doh.521989.xyz:8443/dns-query/iphone17pm#no-h3`。
  * 目的：自用版 2026-09-27 为「快」换本地 DoT 后，**手机解析不再经过 ADH** ⇒ DNS 层拦截与「漏网之鱼」采集对手机失效。换回自建 DoH 做 A/B。
  * 判读三项：ADH querylog 里 iPhone 查询量（预期上升）、手机 REJECT 事件数（预期下降）、DNS 解析延迟。工具 `ops/adh/fetch-qlog.mjs` + `ops/shadowrocket/ab-dns-visibility.py`。
  * 说明头 5 行 → **20 行**，把实验目的、对照项、判读方法与回退方式全写进文件；并实测标注该 DoH 可用（RFC 8484 POST 返回正常解析结果，**只收二进制 POST，不收 `?name=` JSON GET**）。
  * ⚠️ 该版依赖自建 DoH，**别人导入会断网** —— 文件头与本小节都写明「不要给别人用」。
  * 旧实验 T1–T6 全部收口：T2 已并入自用版、T3 已回退、T1 的两个键已按 A 组删除、T5/T6 的假响应+MITM 方案未采用。

---

## ❤️ 致谢

* [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)
* [Surge 官方文档](https://manual.nssurge.com/)（Shadowrocket 语法与参数的主要参考）

---
<p align="center">Maintained by henrysha1989</p>
