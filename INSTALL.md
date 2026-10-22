# WorkBuddy 每日签到 Skill（可分发版）

> 让 WorkBuddy 自动领取「Buddy 加油站」每日积分。仅依赖本机已登录的 WorkBuddy 桌面端，
> 不依赖第三方服务、不收集任何个人信息。
>
> 原始来源：GitHub `cat-xierluo/legal-skills`（MIT 许可证）。
> 本分发版在原始代码基础上做了两处**向后兼容**的增强（见文末「相对原版的改动」）。

---

## 一、它是干什么的

每天自动读取本机 WorkBuddy 桌面端的登录态（v5.3.8+ 的明文登录文件，或旧版 `state.vscdb`），
向腾讯官方接口 `https://copilot.tencent.com/...` 发起「查询状态 → 领取」的签到请求。
脚本**幂等**：当天已签到会直接结束、不会重复领取；连续签到中断也不会触发异常。

### 安全要点（务必读）
- 令牌（accessToken）只在本机内存中、经 HTTP 请求头使用，**不写日志、不落盘、不回显、不外传**。
- 唯一网络请求目标是腾讯官方域名 `copilot.tencent.com`，**无任何第三方域名、无反向 shell、无混淆**。
- 日志 `logs/checkin.log` 只记录「签到结果 / 已领取积分 / 连续天数」，**不含令牌**。
- ⚠️ 令牌等同账号密码：**切勿**把 `logs/checkin.log` 或脚本输出粘贴到公开场合或提交到仓库。

---

## 二、安装（三步）

1. 把整个 `workbuddy-checkin` 文件夹放到 WorkBuddy 的**用户级 skills 目录**：

   | 系统 | 目标路径 |
   |------|----------|
   | Windows | `C:\Users\<你的用户名>\.workbuddy\skills\workbuddy-checkin\` |
   | macOS / Linux | `~/.workbuddy/skills/workbuddy-checkin/` |

   > 即最终结构应为：`.../.workbuddy/skills/workbuddy-checkin/SKILL.md`、`scripts/`、`references/`。
   > 复制后无需重启 WorkBuddy，下一轮对话即可被识别为可用 Skill。

2. 确认本机已做以下任一件事（脚本会自动探测）：
   - 已安装并登录 **WorkBuddy 桌面端 v5.3.8 及以上**（推荐，自带明文登录态，无需额外依赖）；
   - 或已安装 **Node.js**（在 PATH 上，或 WorkBuddy 托管的 `~/.workbuddy/binaries/node/...`）。
   - 极少数旧版无明文登录态的账号，需要 Electron 回退分支（见 `setup.sh` / `setup.ps1`，默认**不**自动下载）。

3. 完成。无需其他配置。

---

## 三、手动试跑（建议先跑一次确认链路通）

在终端里（**注意：路径用你上面安装的实际位置**）：

**macOS / Linux / Windows-Git-Bash：**
```bash
bash ~/.workbuddy/skills/workbuddy-checkin/scripts/checkin.sh
```

**Windows PowerShell（5.1 及以上）：**
```powershell
powershell -ExecutionPolicy Bypass -File "$env:USERPROFILE\.workbuddy\skills\workbuddy-checkin\scripts\checkin.ps1"
```

首次运行会输出一行结果，并写入 `logs/checkin.log`：
- `🎉 签到成功！领取 ...` —— 当天首次签到，拿到了积分；
- `✅ 今日已签到，无需重复领取` —— 当天已被签到过（幂等，正常）；
- `❌ 未找到 Node ...` —— 没探测到运行时，见下方「故障排查」；
- `❌ 令牌已过期（HTTP 401/403）` —— 打开 WorkBuddy 桌面端刷新登录态后重试。

> 排查：若脚本报「找不到 Node」，多半是因为 WorkBuddy 的**托管 Node 不在 PATH 上**。
> 本分发版已自动探测 `~/.workbuddy/binaries/node/versions/*/node.exe`；
> 若仍失败，可显式指定：
> ```bash
> WB_CHECKIN_NODE="C:/Users/<你>/.workbuddy/binaries/node/versions/24.14.0/node.exe" \
>   bash ~/.workbuddy/skills/workbuddy-checkin/scripts/checkin.sh
> ```

---

## 四、配置每日自动签到

两种任选其一：

### 方式 A：用 WorkBuddy 自动化（最省事，推荐）
在 WorkBuddy 里新建一个**每日 recurring 自动化**，提示词写：

> 用 Bash 执行 `bash <你的安装路径>/scripts/checkin.sh`，读取其输出与 `logs/checkin.log` 最新一行，
> 向我汇报：签到成功（积分/连续天数）/ 今日已签到 / 令牌失效 / 失败原因。
> 严禁展示 token、cookie、Authorization 头、账号 UID 等任何凭据字段。

排程建议每天多个整点（如 09/12/15/18/21 点），脚本幂等，电脑任一时间点在线即自动补签。
（注：本环境 RRULE 校验器不支持「一天多个整点」的单条规则，需拆成多条任务。）

### 方式 B：系统自带定时器
- **macOS / Linux**：`crontab -e` 加一行（每天 09:00）
  ```cron
  0 9 * * * bash ~/.workbuddy/skills/workbuddy-checkin/scripts/checkin.sh >> ~/.workbuddy/skills/workbuddy-checkin/logs/checkin.log 2>&1
  ```
- **Windows**：管理员或普通用户均可，用任务计划程序或 `schtasks`：
  ```powershell
  schtasks /Create /TN WorkBuddyDailyCheckin `
    /TR "powershell -ExecutionPolicy Bypass -File $env:USERPROFILE\.workbuddy\skills\workbuddy-checkin\scripts\checkin.ps1" `
    /SC DAILY /ST 09:00 /F
  ```

---

## 五、环境变量（可选）

| 变量 | 作用 |
|------|------|
| `WB_CHECKIN_NODE` | 显式指定 node 可执行文件路径（最高优先级） |
| `WB_CHECKIN_ELECTRON` | 显式指定 electron 可执行文件（仅旧版 state.vscdb 回退分支需要） |
| `WB_CHECKIN_JITTER` | 正整数秒，脚本开始前随机等待 0~N 秒，避免整点请求风暴 |

---

## 六、相对原版的改动（本分发版）

1. **Windows + Git Bash 路径兼容补丁**：原生 Windows 版 `node.exe` 无法识别 `/c/...` 这类
   posix 路径（会被误判成 `D:\c\...` 而找不到 `decrypt-token.js`）。`checkin.sh` 在检测到
   Windows/MinGW/MSYS/Cygwin 环境时，用 `cygpath -w` 把脚本目录转成 Windows 原生路径。
   macOS / Linux 不受影响。
2. **自动探测 WorkBuddy 托管 Node**：`find_node` / `Find-Node` 新增对
   `~/.workbuddy/binaries/node/versions/*/node(.exe)` 的探测，使大多数只装了 WorkBuddy
   托管运行时的用户「开箱即用」，无需手动设置 `WB_CHECKIN_NODE`。

> 其余逻辑（API 路径、`X-User-Id` / `X-Domain` 头、基于真实 HTTP 状态码的鉴权判定、
> `code=10001` 幂等兜底）均与原版一致，并已通过安全审计。
