# JDPro 国内防封配置指南 (Docker/青龙面板)

> [!NOTE]
> 本指南旨在帮助国内用户配置 `6dylan6/jdpro`，实现**脚本运行走本地网络**（避免被判定为数据中心IP），同时确保**代码更新**和**通知推送**畅通无阻。

## ⚠️ 核心警告

> [!CAUTION]
> **绝对不要设置全局代理！**
> 如果你在配置文件中设置了 `http_proxy` / `https_proxy`，所有的脚本流量（包括京东签到）都会走代理，极易导致黑号或CK失效。请务必清理这些变量。

## 🛠️ 配置步骤 (config.sh)

在青龙面板的「配置文件」中进行如下设置：

### 1. 清理危险变量
检查并**注释**或**删除**以下变量：

```bash
# ⚠️ 必须清空或注释，严禁开启
# export http_proxy="http://127.0.0.1:7890"
# export https_proxy="http://127.0.0.1:7890"

# ⚠️ JDPro 专用代理池变量，必须为空，强制直连
export DP_POOL=""
export DY_PROXY=""
```

### 2. 配置仓库加速 (Git)
确保能正常拉取 GitHub 代码，而不影响脚本运行 IP。

```bash
# ✅ 设置 GitHub 镜像加速 (推荐)
export GithubProxyUrl="https://ghproxy.net/"
# 备选: "https://mirror.ghproxy.com/"
```

### 3. 配置通知代理 (Telegram)
如果你国内机器无法直连 Telegram API，请单独设置 TG 代理。

> [!TIP]
> 此设置仅影响通知发送，不会改变脚本签到的网络环境。

```bash
# ✅ Telegram 代理专用变量
export TG_PROXY_HOST="192.168.x.x"  # 你的代理机内网 IP
export TG_PROXY_PORT="7890"         # 代理端口
export TG_PROXY_AUTH=""             # 账号密码(无则留空)
```

### 4. 降低风控策略

> [!IMPORTANT]
> 建议调整以下参数以模拟真人行为，减少并发请求导致的封禁风险。

```bash
# ✅ 调整任务超时时间 (防止网络波动导致任务意外终止)
export CommandTimeoutTime="3h"

# ✅ 随机延时 (保持默认或适当增加，切勿设为0)
export RandomDelay="300"
```

## 🔍 验证检查清单

- [ ] `http_proxy` 环境变量已移除。
- [ ] `DP_POOL` 变量为空。
- [ ] `GithubProxyUrl` 已设置为加速镜像。
- [ ] 运行脚本日志显示正常签到，无连接代理错误。
- [ ] 手动测试通知能成功收到。
