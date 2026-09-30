# 🚀 Shadowrocket 自用分流配置（白名单模式）

个人自用的 Shadowrocket（小火箭）**规则分流配置**，只含规则，不含任何节点与凭据。

> [!CAUTION]
> 本项目**不提供**任何代理服务器节点、鉴权凭据或订阅服务，仅包含规则分流逻辑。

模式：**白名单** —— 兜底走代理（`FINAL,PROXY`）；明确指定的大陆域名、局域网 IP 与 Apple 服务走 `DIRECT`。

---

## 📄 三份配置

| 文件 | 面向 | 内联 / 规则集 | 订阅链接（自建加速，国内可用） |
| :--- | :--- | :--- | :--- |
| [`shadowrocket-白名单.conf`](./shadowrocket-白名单.conf) | **作者自用（默认）** | 27 / 22 | [订阅](https://git.521989.xyz/https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.conf) |
| [`shadowrocket-白名单.通用版.conf`](./shadowrocket-白名单.通用版.conf) | 想少几条个人内联规则、但同样要自建三表的人 | 3 / 22 | [订阅](https://git.521989.xyz/https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.%E9%80%9A%E7%94%A8%E7%89%88.conf) |
| [`shadowrocket-白名单.测试版.conf`](./shadowrocket-白名单.测试版.conf) | 作者自用（测试通道） | 27 / 22 | [订阅](https://git.521989.xyz/https://raw.githubusercontent.com/henrysha1989/shadowrocket-config/main/shadowrocket-%E7%99%BD%E5%90%8D%E5%8D%95.%E6%B5%8B%E8%AF%95%E7%89%88.conf) |

* **自用版**含 3 条自建清单（`reject-custom` / `proxy-custom` / `direct-custom`，由本机 AdGuard Home 管线维护）。
* **通用版**挂**三张自建表**：[`reject-custom.list`](https://github.com/henrysha1989/shadowrocket-adr-rules/blob/main/reject-custom.list)（拦截，含原 `hongguo-ad.list` 的红果/番茄专表 27 条全 `REJECT-DROP`）、[`direct-custom.list`](https://github.com/henrysha1989/shadowrocket-adr-rules/blob/main/direct-custom.list)、[`proxy-custom.list`](https://github.com/henrysha1989/shadowrocket-adr-rules/blob/main/proxy-custom.list)（这两张由手机 db 分析生成，2026-09-30 起补齐 —— 自用版是 `FINAL,PROXY`，缺了直连表会让本该直连的域掉进代理）；Apple 两半排在拦截之前（Apple 全直连）。
* **测试版** = 自用版 + 1 处改动（`dns-server` 换成自建 DoH），在测「ADH 对手机 DNS 的可见性」。⚠️ 依赖自建 DoH，**别人导入会断网**。

每段为什么这么写、每个实验的来龙去脉与变更历史：见 [`配置说明.md`](./配置说明.md)。

---

## 📱 扫码导入（自建加速）

**自用版（默认）**

<img src="./qr/accelerator.png" width="300" alt="自用版 · 自建加速链接二维码">

**通用版**

<img src="./qr/generic-accelerator.png" width="300" alt="通用版 · 自建加速链接二维码">

**测试版**

<img src="./qr/test-accelerator.png" width="300" alt="测试版 · 自建加速链接二维码">

手动添加：小火箭 → **配置** → 右上角 `+` → 粘贴上面的订阅链接 → **下载** → 点该配置 → **使用配置**。

---

## 🧩 用到的规则集

全部来自 **[blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)**，经作者自建加速站 `git.521989.xyz` 拉取：

| 用途 | 规则集 |
| :--- | :--- |
| **拦截** | `BlockHttpDNS` · `AdvertisingLite.list` · `Privacy.list`（自用版另有自建 `reject-custom.list`） |
| **代理** | `Gemini` · `Telegram` · `YouTube` · `Google` · `Facebook` · `Twitter` · `GitHub` · `GlobalMedia`（自用版另有自建 `proxy-custom.list`） |
| **直连** | `Apple`（`Apple.list` + `Apple_Domain.list` 两半）· `China_Domain` · `ChinaMedia` · `Download` · `Lan` · `STUN` · `Tesla`（自用版另有自建 `direct-custom.list`） |

相关链接：

* 上游规则集：<https://github.com/blackmatrix7/ios_rule_script>
* 自建清单（由本机 ADH 管线自动维护）：<https://github.com/henrysha1989/shadowrocket-adr-rules>
* 加速站用法：把 GitHub 链接直接接在 `https://git.521989.xyz/` 后面

---

## 🔄 更新规则集

配置里的规则集都是**远端引用**（非快照）。手动全量重拉：**配置 → 点该配置 → 使用配置 / 编译配置**。
自动：**设置 → 自动更新 → 配置 → 自动后台更新**（间隔 1–7 天；每次会重下约 0.2 MB 规则集并重新编译）。

> `设置 → 订阅` 里的自动更新只管**节点订阅**，与规则集无关。

---

## ⚠️ 注意事项

* **依赖**：三份配置的规则集都经作者自建加速站 `git.521989.xyz` 拉取（上游是 raw 的 Fastly 缓存，5 分钟级）。加速站不可达时规则集会全部拉取失败、整份退化成 `FINAL,PROXY`；此时把前缀换成自备镜像即可。
* **隐私**：本仓库不含任何服务器地址、密码或订阅凭据。
* **来源**：规则集是远端引用的第三方列表，可用性取决于上游仓库。

### 免责声明

* 本项目仅供个人学习与网络管理自用，不对使用后果承担任何直接、间接或连带责任。
* 请遵守当地法律法规，严禁将本项目规则用于任何违法、违规用途。

---
<p align="center">Maintained by henrysha1989</p>
