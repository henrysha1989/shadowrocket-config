# 🚀 Shadowrocket 自用分流配置（白名单模式）

个人自用的 Shadowrocket（小火箭）规则分流配置，只包含**规则分流逻辑**，不含任何节点与凭据。

> [!CAUTION]
> 本项目**不提供**任何代理服务器节点、鉴权凭据或订阅服务，仅包含规则分流逻辑。

---

## 📄 配置说明

* **模式**：白名单 —— 兜底走代理（`FINAL, PROXY`）。明确指定中国大陆域名、局域网 IP 与常见国内服务走 `DIRECT`，其余未知流量默认 `PROXY`。
* **文件**：[`shadowrocket-白名单.conf`](./shadowrocket-白名单.conf)

### 🧱 规则结构（从具体到宽泛）

| 块 | 内容 | 为什么在这个位置 |
| :--- | :--- | :--- |
| ① 拦截 | 广告 / 追踪 / 隐私域名（远端 `DOMAIN-SET` + `RULE-SET`） | 最先匹配，被拒的请求不再进入后续任何匹配 |
| ② 精确直连 | Apple OTA / OCSP / 系统更新、系统连通性检查 | 必须早于块 ③，否则会被 ③ 的服务列表抢走 |
| ③ 代理 | 需走代理的海外服务（Google / YouTube / Telegram / GitHub 等） | 必须早于块 ④，否则会被 ④ 的大表泛域名误截 |
| ④ 直连大表 + 兜底 | 国内直连大表、微信 / 腾讯 IP 段、`GEOIP,CN,DIRECT`、`FINAL,PROXY` | 兜底，放最后 |

### ⚙️ 关键参数

| 参数 | 值 | 说明 |
| :--- | :--- | :--- |
| `block-quic` | `all-proxy` | 只对走代理的连接封 QUIC（`443/UDP`），直连流量不受影响；比手写 `AND` 规则更省 |
| `udp-policy-not-supported-behaviour` | `REJECT` | UDP 兜底策略，替代"全端口 UDP 丢弃"的手写规则 |
| `dns-server` | 带 `#no-h3` | 关闭 DoH 自动升级 HTTP/3，省掉一次协商 |
| `[MITM]` | `enable = false` | 不解密任何流量；`hostname` 已预置，将来要用只改 `enable` |

规则集来自 [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)（`Apple` / `ChinaMax` / `GlobalMedia` / `AdvertisingLite` / `Privacy` 等）。

---

## 🔗 订阅链接

> [!TIP]
> **国内网络环境**推荐优先用 **jsDelivr CDN** 或 **自建加速** 链接，连接更稳定。

* [原始 GitHub Raw 链接](https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/refs/heads/main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.conf)
* [⚡ jsDelivr CDN 加速链接](https://fastly.jsdelivr.net/gh/henrysha1989/shadowrocket-config@main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.conf)
* [🚀 自建加速链接（国内可用）](https://git.521989.xyz/https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.conf)

---

## 📖 使用方法

1. 打开 **Shadowrocket** 客户端。
2. 进入底部导航的 **配置** 页面。
3. 点击右上角 **`+`**。
4. 在弹出框的 **URL** 栏粘贴上方任一订阅链接，点击 **下载**。
5. 下载完成后，点击该配置并选择 **使用配置**。

### 🔄 更新远程规则集

配置里的 24 条规则集（blackmatrix7 21 条 + 自用 3 条）都是**远端引用**，不是快照。Shadowrocket 会把它们缓存起来，需要主动触发才会重新下载。

**手动**

| 方式 | 路径 | 效果 |
| :--- | :--- | :--- |
| 单条重拉 | 配置 → 点配置文件 → 编辑配置 → **规则集 URL** → 点某条地址 | 只重新拉取这一条（会弹状态提示），**不会**重新编译配置 |
| 全量重拉 | 配置 → 点配置文件 → **使用配置** / **编译配置** | 重新拉取全部 24 条并重新编译 |

「规则集 URL」页每条地址后面显示「已成功下载数 / 总数」，✅ 成功、❎ 失败。自用那 3 条是：

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
> **间隔建议 7 天**：每次更新要重下约 1.7 MB 规则集、重编译 8 万余条规则，这是本配置唯一确定的周期性网络突发，拉到 7 天可直接砍到 1/7。需要立刻刷新时按上表手动做一次即可。

#### ⏱ 各条列表的源头节奏

| 规则集 | 上游更新 | 中间缓存 | 手机上最快可见 |
| :--- | :--- | :--- | :--- |
| 自用 3 条（`*-custom.list`） | 每 4 小时自动（本机管线 → 仓库 Action） | 自建加速站 / Fastly `max-age=300` | 约 5 分钟后 |
| blackmatrix7 21 条 | 上游不定期 | jsDelivr `@master` 分支缓存（最长约 12 小时） | 最长约 12 小时后 |

> 想让第二行也贴近实时：把这 21 条的 `cdn.jsdelivr.net/gh/blackmatrix7/...` 换成自建加速站前缀（`git.521989.xyz/https://raw.githubusercontent.com/...`，实测 5 分钟缓存），代价是每次刷新都从自建加速站回源约 1.7 MB。

---

## ⚠️ 注意事项

> [!WARNING]
> * **CDN 缓存延迟**：jsDelivr 存在数小时缓存。若刚更新完没有生效，请手动更新配置或临时改用 Raw / 自建加速链接。
> * **隐私与安全**：本仓库不含任何服务器地址、密码或订阅凭据；若你在自己的 fork 里添加节点，请勿提交明文凭据。
> * **规则来源**：规则集是远端引用的第三方列表，其可用性取决于上游仓库与 CDN。

### 免责声明

* 本项目仅供个人学习与网络管理自用，不对使用后果承担任何直接、间接或连带责任。
* 请遵守当地法律法规，严禁将本项目规则用于任何违法、违规用途。

---

## 📝 更新日志

* **2026-09-25**：结构由 8 段精简为 4 块；规则 71 → 69 条（合并 `101.226.0.0/16` + `101.227.0.0/16`）；修正 5 条被代理列表抢走的死规则；移除 AppleProxy 列表；关闭 MITM。

---

## ❤️ 致谢

* [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)
* [Surge 官方文档](https://manual.nssurge.com/)（Shadowrocket 语法与参数的主要参考）

---
<p align="center">Maintained by henrysha1989</p>
