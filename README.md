# workbuddy-checkin

WorkBuddy「Buddy 加油站」**每日积分自动签到** Skill（可分发版）。

让 WorkBuddy 每天自动读取本机已登录的桌面端登录态，向腾讯官方接口发起「查询状态 → 领取」签到请求；脚本**幂等**，当天已签到会直接结束、不会重复领取。

> 原始来源：[cat-xierluo/legal-skills](https://github.com/cat-xierluo/legal-skills)（MIT 许可证）。
> 本仓库在原始代码基础上做了两处**向后兼容**的增强（见文末），其余逻辑（API 路径、鉴权头、基于真实 HTTP 状态码的判定、`code=10001` 幂等兜底）均与原版一致，并已通过安全审计。

## 安全要点（务必先读）

- 令牌（accessToken）只在本机内存中、经 HTTP 请求头使用，**不写日志、不落盘、不回显、不外传**。
- 唯一网络请求目标是腾讯官方域名 `copilot.tencent.com`，**无任何第三方域名、无反向 shell、无混淆代码**。
- 日志 `logs/checkin.log` 只记录「签到结果 / 已领取积分 / 连续天数」，**不含令牌**。
- ⚠️ 令牌等同账号密码：**切勿**把 `logs/checkin.log` 或脚本输出粘贴到公开场合或提交到仓库。

## 安装与使用

完整步骤（安装目录、手动试跑、每日自动签到、排障、安全）见 👉 **[INSTALL.md](INSTALL.md)**。

一句话安装：把本仓库内容放到 `~/.workbuddy/skills/workbuddy-checkin/`（Windows 为 `C:\Users\<你>\.workbuddy\skills\workbuddy-checkin\`），本机已装并登录 WorkBuddy 桌面端 v5.3.8+ 即可。

## 相对原版的增强

1. **Windows + Git Bash 路径兼容补丁**：原生 `node.exe` 无法识别 `/c/...` 这类 posix 路径（会误判成 `D:\c\...` 而找不到 `decrypt-token.js`）。`checkin.sh` 在检测到 Windows/MinGW/MSYS/Cygwin 环境时，用 `cygpath -w` 把脚本目录转成 Windows 原生路径。macOS / Linux 不受影响。
2. **自动探测 WorkBuddy 托管 Node**：`find_node` / `Find-Node` 新增对 `~/.workbuddy/binaries/node/versions/*/node(.exe)` 的探测，使大多数只装了 WorkBuddy 托管运行时的用户**开箱即用**，无需手动设置 `WB_CHECKIN_NODE`。

## 许可证

[MIT](LICENSE)。派生自 [cat-xierluo/legal-skills](https://github.com/cat-xierluo/legal-skills)（MIT），修改部分同样以 MIT 发布。
