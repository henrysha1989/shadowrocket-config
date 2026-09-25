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

### 🔄 开启自动更新

**设置 → 订阅**，打开 **打开应用时更新** 与 **自动后台更新**。

> [!NOTE]
> **更新间隔建议设为 7 天**（客户端里可选 1–7 天）。每次自动更新都要重新下载约 1.7 MB 规则集并重新编译 8 万余条规则，是这份配置唯一确定的周期性网络突发；拉到 7 天可直接砍到 1/7。

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
