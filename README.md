# 🚀 Cf-Edgetunnel (edgetunnel Fork) — 多国代理节点增强版

基于 [cmliu/edgetunnel](https://github.com/cmliu/edgetunnel) 的 fork，支持 **多国代理节点自动分组**，订阅输出自带 `[JP]` `[AU]` `[SG]` 标识。

---

## ✨ 增强特性

| 特性 | 说明 |
|:---|:---|
| 🌍 **多国代理节点** | 通过 `COUNTRY_PROXY` 环境变量配置多国反代 IP，订阅自动分国家输出 |
| 🏷️ **自动国家前缀** | 节点名称自动添加 `[JP]` `[AU]` `[SG]` 前缀，Clash 前端转 emoji 🇯🇵🇦🇺🇸🇬 |
| 🔗 **proxyip 透传** | 订阅转换路径自动透传 `proxyip` 参数 |
| ⚡ **Wrangler CLI 部署** | 完整支持 `wrangler` 命令行部署和管理 |
| 🔧 **仅 Workers** | 专为 CF Workers 优化，不依赖 Pages |

---

## ⚡ Workers 部署（wrangler CLI）

### 前置条件

```bash
# 安装 wrangler
npm install -g wrangler

# 登录 Cloudflare
wrangler login
```

### 克隆与部署

```bash
# 克隆本仓库
git clone https://github.com/weekeao-boop/Cf-Edgetunnel.git
cd Cf-Edgetunnel

# 修改 wrangler.toml 中的 KV 命名空间 ID
# KV 绑定名必须为 "KV"
# 先创建 KV 命名空间（如未创建）：
wrangler kv namespace create EDTUNNEL_NS
# 将返回的 id 填入 wrangler.toml 的 [[kv_namespaces]] 中

# 设置环境变量（务必先设置再部署）
wrangler secret put ADMIN    # 管理员密码
wrangler secret put COUNTRY_PROXY  # 多国代理，格式: JP:ip,AU:ip,SG:ip

# 使用明文环境变量部署（替代 secret，避免 secret 环境变量取不到的问题）
npx wrangler deploy --var ADMIN="你的密码" --var COUNTRY_PROXY="JP:38.47.104.110,AU:206.245.167.71,SG:178.128.86.3"

# 或使用 wrangler.toml 中的 [vars] 配置明文变量（推荐）：
# [vars]
# ADMIN = "你的密码"
# COUNTRY_PROXY = "JP:38.47.104.110,AU:206.245.167.71,SG:178.128.86.3"
```

### 绑定自定义域名

```bash
# 在 Cloudflare Dashboard Workers 页面配置自定义域
# 或通过 API 绑定
wrangler domains add your-domain.com
```

### 更新代码

```bash
git pull upstream main
npx wrangler deploy
```

---

## 🔑 环境变量说明

| 变量名 | 必填 | 示例 | 说明 |
|:---|:---:|:---|:---|
| **ADMIN** | ✅ | `你的密码` | 后台管理面板登录密码 |
| **COUNTRY_PROXY** | ❌ | `JP:38.47.104.110,AU:206.245.167.71,SG:178.128.86.3` | **多国代理IP**，格式 `国家码:IP,国家码:IP,...`，自动生成 `[JP]` `[AU]` 节点 |
| **KEY** | ❌ | `CMLiussss` | 快速订阅路径密钥 |
| **UUID** | ❌ | `90cd4a77-...` | 强制固定 UUIDv4 |
| **PROXYIP** | ❌ | `proxyip.cmliussss.net:443` | 全局反代 IP（与 COUNTRY_PROXY 互斥逻辑不同） |
| **URL** | ❌ | `https://example.com` | 伪装页面地址 |
| **KV** | ❌ | — | KV 命名空间绑定（必填 KV 绑定） |
| **BEST_SUB** | ❌ | `true` | 优选订阅生成器模式 |

### COUNTRY_PROXY 格式详解

```
JP:38.47.104.110,AU:206.245.167.71,SG:178.128.86.3
```

- `国家码`：任意 2 字母 ISO 国家码（`JP` `AU` `SG` `US` `KR` `HK` `GB` 等）
- `IP`：该国家的反代 IP 地址
- 订阅输出节点自动加 `[JP]` `[AU]` `[SG]` 前缀
- Clash 客户端自动转换为 🇯🇵 🇦🇺 🇸🇬 emoji

### 也支持 URL 参数传多国 proxyip

```
https://your.domain/sub?token=xxx&proxyip=JP:1.2.3.4&proxyip=AU:5.6.7.8
```

URL 参数优先级高于环境变量。

---

## 📡 订阅配置

### 订阅地址

```
https://你的域名/sub?token=你的订阅TOKEN
```

订阅 Token 由 `ADMIN` + `KEY` 自动 MD5 生成。也可在后台 `/admin` 查看。

### Clash-Party / subconverter 配置

确保你的转换器（如 `api.v1.mk`）在 `profile.yaml` 中指向：

```yaml
订阅:
  CF-FREE-RAY:
    type: http
    URL: https://你的域名/sub?token=你的TOKEN
    interval: 3600
```

> **注意**：edgetunnel 直接输出完整 Clash 配置，不需要额外经过 subconverter 转换。`target=mixed` 模式已包含所有格式。

---

## 🔧 wrangler 常用命令

```bash
# 部署
npx wrangler deploy

# 查看日志
npx wrangler tail

# 列出 secret
npx wrangler secret list

# 删除 secret
npx wrangler secret delete 变量名

# 设置明文环境变量
npx wrangler deploy --var KEY="value"

# KV 操作
npx wrangler kv:key put --binding=KV "key" "value"
npx wrangler kv:key get --binding=KV "key"

# 列出 Workers
npx wrangler deployments list
```

---

## ⚠️ 常见踩坑点

1. **Secret 环境变量取不到值** → 部分 Worker 版本中 secret 环境变量读取异常。**解决方案**：改用 `--var` 明文变量部署，或写入 `wrangler.toml` 的 `[vars]` 段。
2. **Pages 项目的域名冲突** → 域名绑定 Pages 后无法同时绑定 Worker。**解决方案**：删除 Pages 项目（先解绑域名再删项目），域名绑定 Worker。
3. **Apex 域名冲突** → `example.com` 根域名已有 DNS 记录时无法直接绑定。**解决方案**：用子域名如 `ray.example.com` 绑定，或清空 DNS 后绑定。
4. **订阅返回空** → 检查 Token 是否正确、KV 命名空间是否绑定（变量名必须为 `KV`）。
5. **节点延迟高** → 更换 `COUNTRY_PROXY` 中的 IP 为低延迟反代 IP。

---

## 📖 上游项目

原项目 [cmliu/edgetunnel](https://github.com/cmliu/edgetunnel) — 感谢 cmliu 的杰出工作。

---

## ⚠️ 免责声明

1. 本项目仅供**教育、科学研究及个人安全测试**之目的。
2. 使用者在下载或使用本项目代码时，必须严格遵守所在地区的法律法规。
3. 作者对任何滥用本项目代码导致的行为或后果均不承担任何责任。
4. 建议在测试完成后 24 小时内删除本项目相关部署。