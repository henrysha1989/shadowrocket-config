# 🚀 Shadowrocket 自用分流配置（白名单模式）

个人自用的 Shadowrocket（小火箭）规则分流配置，只包含**规则分流逻辑**，不含任何节点与凭据。

> [!CAUTION]
> 本项目**不提供**任何代理服务器节点、鉴权凭据或订阅服务，仅包含规则分流逻辑。

---

## 📄 配置说明

* **模式**：白名单 —— 兜底走代理（`FINAL, PROXY`）。明确指定中国大陆域名、局域网 IP 与常见国内服务走 `DIRECT`，其余未知流量默认 `PROXY`。
* **文件**：两份，规则逻辑同源 ——
  * [`shadowrocket-白名单.conf`](./shadowrocket-白名单.conf)：**作者自用版（默认）**。61 条规则、24 条远端规则集，含自建 DoH 与自建加速站。
  * [`shadowrocket-白名单.通用版.conf`](./shadowrocket-白名单.通用版.conf)：**通用版，任何人可直接用**。57 条规则、21 条远端规则集，换公共 DoH + 公共 CDN jsDelivr。
  两份都是**纯规则、无注释**（说明都收在 [`配置说明.md`](./配置说明.md)）。
* **规模**：4 个段（`[General]` / `[Rule]` / `[URL Rewrite]` / `[MITM]`）。

**结构**：按「从具体到宽泛」分四块 —— ① 拦截 → ② 精确直连（Apple 系统底层 / 私网设备）→ ③ 代理（海外服务）→ ④ 直连大表 + 微信打洞端口 + `GEOIP,CN` 兜底。每块为什么在那个位置、每条特殊规则的来龙去脉，见 [`配置说明.md`](./配置说明.md)。

规则集：**21 条**来自 [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)（`Apple`（域名 + 规则两半）/ `ChinaMedia` / `GlobalMedia` / `AdvertisingLite` / `Privacy` 等）；自用版另有 **3 条**自用列表（通用版没有）。自用版经**自建加速站** `git.521989.xyz` 拉取，通用版经**公共 CDN jsDelivr** 拉取（不依赖作者任何服务）。

> [!NOTE]
> **该选哪份？** 只想拿来自己用、不想碰作者的基础设施 → 选 **通用版**；想复刻作者这套（含自建 DoH / 加速站 / 自用清单）→ 看 **自用版**。两份的逐条差异见 [`配置说明.md` 的「🆚 自用版 vs 通用版」](./配置说明.md#-自用版-vs-通用版)。

---

## 🧩 两个版本怎么选

| 文件 | 适合谁 | 依赖 | 规则集来源 |
| :--- | :--- | :--- | :--- |
| `shadowrocket-白名单.conf` | 作者本人 / 想复刻整套的人 | 作者的自建 DoH + 自建加速站 | 23 条（20 公开 + 3 作者自用） |
| `shadowrocket-白名单.通用版.conf` | **任何人** | 无 | 20 条（全部公开上游） |

通用版唯一的取舍：规则集走 jsDelivr，`@master` 缓存最长约 12 小时（自用版的加速站是 5 分钟级）。要更快的新鲜度，把规则集前缀换成自备加速站即可。

---

## 🔗 订阅链接

> [!TIP]
> **国内网络环境**推荐优先用 **自建加速** 或 **jsDelivr CDN** 链接，连接更稳定。

### 自用版（默认）

* [原始 GitHub Raw 链接](https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/refs/heads/main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.conf)
* [⚡ jsDelivr CDN 加速链接](https://fastly.jsdelivr.net/gh/henrysha1989/shadowrocket-config@main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.conf)
* [🚀 自建加速链接（国内可用）](https://git.521989.xyz/https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.conf)

### 通用版（推荐给别人用）

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

#### 通用版（推荐给别人用）

**🚀 自建加速（推荐，国内可用）**

<img src="./qr/generic-accelerator.png" width="380" alt="通用版 · 自建加速链接二维码">

**⚡ jsDelivr CDN**

<img src="./qr/generic-jsdelivr.png" width="380" alt="通用版 · jsDelivr 链接二维码">

> 原始 Raw 链接不单独出码 —— 国内直连不稳定，需要用的时候复制上面的文字链接即可。

---

## 📖 使用方法

> 手机和屏幕在同一视线内时，直接用 Shadowrocket 的「扫一扫」扫上面的二维码最快；不在手边就按下面手动添加。

1. 打开 **Shadowrocket** 客户端。
2. 进入底部导航的 **配置** 页面。
3. 点击右上角 **`+`**。
4. 在弹出框的 **URL** 栏粘贴上方任一订阅链接，点击 **下载**。
5. 下载完成后，点击该配置并选择 **使用配置**。

### 🔄 更新远程规则集

配置里的规则集（自用版 24 条 = blackmatrix7 21 条 + 自用 3 条；通用版 21 条）都是**远端引用**，不是快照。Shadowrocket 会把它们缓存起来，需要主动触发才会重新下载。

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
| blackmatrix7 21 条 | 上游不定期 | 自用版：自建加速站 / 通用版：raw 官方 | 约 5 分钟后 |

> 自用版 24 条规则集统一走自建加速站，缓存都是 5 分钟级。代价是每次刷新都从自建加速站回源约 1.6 MB（`git.521989.xyz` 由本人维护）；通用版走公共 CDN jsDelivr（`@master` 缓存最长约 12 小时）。

---

## ⚠️ 注意事项

> [!WARNING]
> * **CDN 缓存延迟**：配置本身的 jsDelivr 订阅链接存在数小时缓存（自用版的规则集已全部改走自建加速站，不受影响）。若刚推送完没用上，请手动更新配置或临时改用 Raw / 自建加速链接。
> * **隐私与安全**：本仓库不含任何服务器地址、密码或订阅凭据；若你在自己的 fork 里添加节点，请勿提交明文凭据。
> * **规则来源**：规则集是远端引用的第三方列表，其可用性取决于上游仓库；自用版还额外依赖作者的自建加速站。

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

---

## ❤️ 致谢

* [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)
* [Surge 官方文档](https://manual.nssurge.com/)（Shadowrocket 语法与参数的主要参考）

---
<p align="center">Maintained by henrysha1989</p>
