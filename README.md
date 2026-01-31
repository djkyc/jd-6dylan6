# jd-6dylan6

6dylan6/jdpro 仓库分析与配置指南
目标
在通过 Docker 部署的 Qinglong 面板中运行 6dylan6/jdpro 脚本时，确保：

避免违规：京东相关业务（如签到）完全走国内本地宽带网络。
正常更新：仓库代码更新（git pull）和通知（如 Telegram）可以使用代理（如果需要）。
规避封号：利用仓库内置的防封策略。
核心分析
经过对 6dylan6/jdpro 仓库文档的分析，该仓库支持两种级别的代理配置：

系统/仓库级下载代理：用于拉取代码。
脚本运行级代理：通过 DP_POOL (代理池) 或 DY_PROXY (API 代理) 变量控制。
关键发现： 如果设置了 DP_POOL 且没有设置 PERMIT_JS，默认行为是所有脚本都会走代理。这会导致签到任务使用代理 IP，从而增加被京东判定为“数据中心/黑产 IP”的风险。

配置建议 (Configuration Guide)
请在青龙面板的「配置文件」(config.sh) 中按以下策略进行配置。

1. 确保脚本运行走“国内本地宽带” (核心需求)
为了确保签到等所有业务脚本使用本机直连网络，请必须清空或注释掉以下变量（如果之前设置过）：

# [错误示范] 不要设置全局代理，这会导致脚本流量走代理
# export http_proxy="http://127.0.0.1:7890"
# export https_proxy="http://127.0.0.1:7890"
# [jdpro 专用配置]
# 确保 DP_POOL (代理池) 为空，除非你明确知道自己在做什么
export DP_POOL=""
# 确保 DY_PROXY (API代理) 为空
export DY_PROXY=""
例外情况：如果你必须为某些非签到任务（例如浏览国外网站的任务，虽然 JD 此类任务很少）使用代理，你可以利用白名单功能：

设置：DP_POOL="http://代理地址"
必须同时设置：export PERMIT_JS="activity_name&other_task" (仅允许这些脚本走代理)
注意：PERMIT_JS 中绝对不要包含 sign、checkin 等签到脚本的关键词。
2. 单独配置代码仓库更新代理
由于国内网络访问 GitHub 可能受阻，你需要为 ql repo 命令配置单独的加速代理，而不影响脚本运行。

在 config.sh 中找到 GithubProxyUrl 并修改：

# 使用加速镜像（推荐）
export GithubProxyUrl="https://ghproxy.net/"
# 或者其他可用的加速前缀
这样，青龙在拉取脚本时会使用加速链接，但脚本运行时仍使用你的本地网络。

3. 单独配置通知代理 (Telegram 等)
如果你需要使用 Telegram 通知，但脚本又要直连，请使用专门的 TG 代理变量，而不是全局代理：

# Telegram 代理专用变量 (仅影响通知发送，不影响 JD 脚本)
export TG_PROXY_HOST="192.168.x.x"  # 你的代理机 IP
export TG_PROXY_PORT="7890"         # 你的代理端口
export TG_PROXY_AUTH=""             # 账号密码(如果有)
对于 Server酱、企业微信等国内服务，通常不需要代理。

防封与优化策略
除了“不走代理”外，根据仓库文档，建议应用以下配置以降低风险：

随机延时 (Random Delay)

青龙默认有随机延时。
在任务中利用 RandomDelay 机制，不要让所有账户在同一秒签到。
并发与分组

不要让所有账号同时并发跑同一个脚本（容易触发风控）。
利用 jdpro 的分组功能：
# 示例：config.sh 或 任务后缀
#desi JD_COOKIE 1-10  (前10个号一组)
调整超时时间

防止任务因网络卡顿被强制杀掉（导致异常退出记录）。
在 config.sh 中设置：
export CommandTimeoutTime="3h"  # 建议由默认1h改为3h
总结操作清单
打开青龙面板 -> 配置文件。
检查并删除/注释所有 http_proxy, https_proxy, all_proxy 全局变量。
检查并删除/注释 DP_POOL, DY_PROXY (或者确保你理解 PERMIT_JS 白名单机制)。
设置 GithubProxyUrl="https://ghproxy.net/" 以保障脚本更新。
(可选) 设置 TG_PROXY_HOST 以保障 TG 通知。
保存。
遵循以上配置，即可实现“脚本运行走本地宽带，代码更新走代理”的隔离环境
