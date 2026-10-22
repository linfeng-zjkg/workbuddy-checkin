# WorkBuddy 每日签到 Skill — 接收方使用说明

> 让 WorkBuddy 自动领取「Buddy 加油站」每日积分。仅依赖本机已登录的 WorkBuddy 桌面端，
> 不依赖第三方服务、不收集任何个人信息。
>
> 原始来源：GitHub `cat-xierluo/legal-skills`（MIT 许可证）。
> 本分发版在原始代码基础上做了两处**向后兼容**的增强（见文末「相对原版的改动」）。

本说明面向**拿到本仓库的人**，按「下载 → 安装 → 试跑 → 自动化 → 排障」五步走，照做即可。

---

## 第 0 步：先确认你满足前置条件

| 条件 | 说明 | 是否满足 |
|------|------|----------|
| 电脑装了 **WorkBuddy 桌面端 v5.3.8 及以上**并已登录 | 脚本读取的就是这个桌面端的本地登录态 | 必须 ✅ |
| 操作系统 | Windows 10/11、macOS、Linux 均可 | 任一 ✅ |
| 需要自己装 Node.js 吗？ | **不需要**——WorkBuddy 自带托管 Node，脚本已能自动探测。你本机有 Node 也行 | 可选 |
| 需要账号密码 / 第三方 App 吗？ | **都不需要** | — |

> ⚠️ 只装了 WorkBuddy 网页版、没装桌面端 → 没有本地登录态文件，脚本读不到令牌，**无法签到**。
> 必须用已登录的 **桌面端**（v5.3.8+ 会在本地存一份明文登录态文件）。

---

## 第 1 步：拿到文件（两种途径，选一个）

### 途径 A：从 GitHub 下载 ZIP（不会用 git 的人首选）
1. 浏览器打开 👉 **https://github.com/linfeng-zjkg/workbuddy-checkin**
2. 点页面右上角的绿色 **「Code」** 按钮 → 选 **「Download ZIP」**
3. 下载完成后解压，你会得到一个名为 `workbuddy-checkin` 的文件夹

### 途径 B：用 git 克隆（会命令行的人）
```bash
git clone https://github.com/linfeng-zjkg/workbuddy-checkin.git
```
克隆出来的 `workbuddy-checkin/` 文件夹即为所需。

---

## 第 2 步：把文件夹放进正确位置（最关键一步）

目标：让 WorkBuddy 能在 `用户级 skills 目录` 下找到这个 skill。

### Windows（最简单，用资源管理器）
1. 按 `Win + R`，输入 `%USERPROFILE%\.workbuddy\skills` 回车
   - 如果弹出「找不到」说明 `skills` 文件夹还不存在，就在 `.workbuddy` 目录下**新建**一个 `skills` 文件夹
2. 把第 1 步拿到的 **整个 `workbuddy-checkin` 文件夹** 复制 / 粘贴进去
3. 最终路径应形如：
   ```
   C:\Users\你的用户名\.workbuddy\skills\workbuddy-checkin\
   ```

### macOS / Linux（终端）
```bash
mkdir -p ~/.workbuddy/skills
cp -r /你放压缩包的位置/workbuddy-checkin ~/.workbuddy/skills/
```

### ✅ 安装正确性自检
进入该目录，确认能看到这些文件/文件夹（缺一不可）：
```
workbuddy-checkin/
├── SKILL.md          ✅ 必须
├── INSTALL.md        ✅ 必须
├── README.md         ✅ 必须
├── LICENSE           ✅ 必须
├── scripts/          ✅ 必须（含 checkin.sh / checkin.ps1 / decrypt-token.js）
├── references/       ✅ 必须
└── logs/             ✅ 必须有（可为空，签到记录写在这里）
```
> 复制后**无需重启** WorkBuddy，下一轮对话它就会被识别为可用 Skill。

---

## 第 3 步：手动试跑一次（确认链路真的通）

> 强烈建议先手动跑一次，确认「读取登录态 → 打通官方接口」这条链路没问题，再上自动化。

### Windows 用户（任选其一）

**方式 1：PowerShell**
```powershell
powershell -ExecutionPolicy Bypass -File "$env:USERPROFILE\.workbuddy\skills\workbuddy-checkin\scripts\checkin.ps1"
```

**方式 2：Git Bash / WSL**
```bash
bash ~/.workbuddy/skills/workbuddy-checkin/scripts/checkin.sh
```

### macOS / Linux 用户
```bash
bash ~/.workbuddy/skills/workbuddy-checkin/scripts/checkin.sh
```

### 你会看到三种结果之一（都是正常的，对号入座）

| 输出 | 含义 | 你该怎么做 |
|------|------|-----------|
| `🎉 签到成功！领取 X 积分（连续 N 天）` | 当天首次签到，拿到了积分 | 完事，去第 4 步设自动化 |
| `✅ 今日已签到，无需重复领取` | 今天已经被签到过（比如你刚在客户端手动签了，或今天早些时候脚本跑过） | **正常**，脚本是幂等的，不会重复领；去第 4 步 |
| `❌ 未找到 Node 或令牌` / `❌ 令牌已过期（HTTP 401/403）` | 运行时或登录态有问题 | 看下方「第 5 步排障」 |

> 无论哪种结果，详情都会同时写入 `logs/checkin.log`（看一眼：里面**只有**积分/连续天数，**没有**你的令牌）。

---

## 第 4 步：设置「每天自动签到」（二选一）

签到本身是幂等的，所以**一天跑多次也没事**——这也是为什么推荐一天排几个时间点（比如 09/12/15/18/21 点），电脑只要在这中间任一时刻开机在线，就会自动补签。

### 方式 A：让 WorkBuddy 自己帮你建（最省事，强烈推荐）

直接开一个新对话，把下面这段**原样**发给 WorkBuddy，让它帮你建定时任务：

> 请帮我创建一个每日自动化任务来领 WorkBuddy「Buddy 加油站」积分。用 Bash 执行这条命令：
> `bash ~/.workbuddy/skills/workbuddy-checkin/scripts/checkin.sh`
> 执行后读取命令输出与 `~/.workbuddy/skills/workbuddy-checkin/logs/checkin.log` 最新一行，
> 向我汇报结果：签到成功（多少积分/连续几天）/ 今日已签到 / 令牌失效 / 失败原因。
> 重要：回复中严禁展示 token、cookie、Authorization 头、账号 UID 等任何凭据字段。
> 请设置为每天 09:00、12:00、15:00、18:00、21:00 各运行一次（若一次只能设一个时间，就建 5 个任务）。

如果你更想自己手动在界面里建（不用上面这段），通用路径是：
**WorkBuddy 左侧「自动化 / 计划任务」入口 → 新建任务 → 触发方式选「每天 / 重复」→ 设置时间 → 把上面那段提示词粘进任务内容 → 保存并启用。**

> 注：某些 WorkBuddy 版本的定时规则不支持「一天多个整点」写在一条里，拆成 5 条单点任务即可，脚本幂等互不影响。

### 方式 B：用操作系统自带定时器（不依赖 WorkBuddy 在线）

**macOS / Linux —— `crontab`**
```bash
crontab -e
# 在打开的文件末尾加下面一行（每天 09:00 跑，结果追加进日志）：
0 9 * * * bash ~/.workbuddy/skills/workbuddy-checkin/scripts/checkin.sh >> ~/.workbuddy/skills/workbuddy-checkin/logs/checkin.log 2>&1
```

**Windows —— 任务计划程序（命令行版）**
```powershell
schtasks /Create /TN WorkBuddyDailyCheckin `
  /TR "powershell -ExecutionPolicy Bypass -File $env:USERPROFILE\.workbuddy\skills\workbuddy-checkin\scripts\checkin.ps1" `
  /SC DAILY /ST 09:00 /F
```
（想多加几个时间点就多跑几次 `/Create`，把 `/TN` 和 `/ST` 改一下。）

---

## 第 5 步：故障排查对照表

| 现象 | 可能原因 | 解决办法 |
|------|----------|----------|
| `❌ 未找到 Node` | WorkBuddy 托管 Node 没被探测到（极少见） | 显式指定：`WB_CHECKIN_NODE="C:/Users/你/.workbuddy/binaries/node/versions/24.14.0/node.exe" bash ~/.workbuddy/skills/workbuddy-checkin/scripts/checkin.sh`（路径里的版本号以你机器实际为准，去 `.workbuddy/binaries/node/versions/` 看一眼） |
| `❌ 令牌已过期（HTTP 401/403）` | WorkBuddy 桌面端登录态过期 / 被踢下线 | 打开 WorkBuddy 桌面端，重新登录一次；次日自动化会自动恢复 |
| 脚本完全没输出 / 闪退 | Windows 上直接双击 `.ps1` 被策略拦截 | 务必用上面「第 3 步」给的命令行方式运行，不要双击 |
| `✅ 今日已签到` 但积分没涨 | 今天确实已被签到过（客户端手动签过 / 早些时候跑过） | 正常，明天再看；签到按天计，不叠加 |
| 想确认到底签到没 | 看 `logs/checkin.log` 最新一行 | 里面只有结果摘要，无敏感信息，可放心看 |
| 电脑长期关机错过签到 | 离线期间不会补签历史日期 | 开机后下一个时间点会自动跑当天的；历史漏签不影响当天 |

---

## 第 6 步：安全须知（务必读，也务必转告他人）

- 令牌（accessToken）只在本机内存中、经 HTTP 请求头使用，**不写日志、不落盘、不回显、不外传**。
- 唯一网络请求目标是腾讯官方域名 `copilot.tencent.com`，**无任何第三方域名、无反向 shell、无混淆代码**。
- 日志 `logs/checkin.log` 只记录「签到结果 / 已领取积分 / 连续天数」，**不含令牌**。
- ⚠️ 令牌等同账号密码：**切勿**把 `logs/checkin.log` 或脚本输出粘贴到公开场合或提交到仓库。
- 本仓库代码已在来源方做逐文件安全审计（无外泄、无混淆、无越权请求），可放心使用；若仍有顾虑，可自行 `cat scripts/*.js` 与 `scripts/*.sh` 复看。

---

## 可选环境变量（一般不需要设）

| 变量 | 作用 | 何时用 |
|------|------|--------|
| `WB_CHECKIN_NODE` | 显式指定 node 可执行文件路径（最高优先级） | 自动探测失败时 |
| `WB_CHECKIN_ELECTRON` | 显式指定 electron 路径 | 仅旧版 `state.vscdb` 回退分支需要（极少） |
| `WB_CHECKIN_JITTER` | 正整数秒，脚本开始前随机等 0~N 秒 | 想避开整点请求高峰时 |

---

## 相对原版的改动（本分发版）

1. **Windows + Git Bash 路径兼容补丁**：原生 Windows 版 `node.exe` 无法识别 `/c/...` 这类
   posix 路径（会被误判成 `D:\c\...` 而找不到 `decrypt-token.js`）。`checkin.sh` 在检测到
   Windows/MinGW/MSYS/Cygwin 环境时，用 `cygpath -w` 把脚本目录转成 Windows 原生路径。
   macOS / Linux 不受影响。
2. **自动探测 WorkBuddy 托管 Node**：`find_node` / `Find-Node` 新增对
   `~/.workbuddy/binaries/node/versions/*/node(.exe)` 的探测，使大多数只装了 WorkBuddy
   托管运行时的用户「开箱即用」，无需手动设置 `WB_CHECKIN_NODE`。

> 其余逻辑（API 路径、`X-User-Id` / `X-Domain` 头、基于真实 HTTP 状态码的鉴权判定、
> `code=10001` 幂等兜底）均与原版一致，并已通过安全审计。
