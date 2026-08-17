# linuxdo-checkin 项目说明与每日保活操作指南

> 项目地址：https://github.com/doveppp/linuxdo-checkin

## 一、这个项目是干什么的

**linuxdo-checkin** 是一个 [LinuxDo 论坛（linux.do）](https://linux.do/) 的**每日自动签到/保活脚本**。

LinuxDo 论坛的信任等级（Trust Level）与"访问天数"等活跃指标挂钩，账号需要每天登录并浏览帖子才能维持/提升等级。这个项目就是替你自动完成这些动作：

1. **自动登录** LinuxDo（支持 Cookie 登录或账号密码登录）；
2. **自动浏览帖子**：随机打开 10 个主题帖，模拟真人向下滚动阅读（每次滚动 550~650 像素、随机等待 2~4 秒），约 30% 概率顺手点个赞；
3. **打印 Connect Info**：从 connect.linux.do 抓取你的信任等级进度表（当前值 / 要求值）；
4. **（可选）推送通知**：签到结果可推送到 Telegram / Gotify / Server酱³ / wxpush。

技术上使用 Python + DrissionPage（驱动 Chromium 无头浏览器）+ curl_cffi 实现，能绕过常规反爬检测。

> ⚠️ 项目公告：请勿魔改成多线程刷论坛，本项目仅用于个人账号保活。

## 二、如何操作实现每日保活

有两种运行方式，**推荐 GitHub Actions（免费、无需自己的服务器）**。

### 方式 A：GitHub Actions（推荐）

#### 1. Fork 仓库

打开 https://github.com/doveppp/linuxdo-checkin ，点右上角 **Fork**，把仓库 Fork 到你自己的 GitHub 账号下。

#### 2. 配置登录凭据（Secrets）

进入你 Fork 后的仓库：`Settings` → `Secrets and variables` → `Actions` → `New repository secret`，添加以下 Secret（**二选一**）：

> 📍 Secrets 在哪：打开你 Fork 后的仓库页面，顶部导航栏（Code / Pull requests / Actions / ... / Settings）最右边的 **Settings**；进入后看**左侧边栏**，往下找到 **Secrets and variables**，点开选 **Actions**，再点绿色按钮 **New repository secret**。注意是**仓库的 Settings**，不是你 GitHub 账号头像里的个人 Settings。

**方式一：Cookie 登录（推荐，优先使用，可绕过验证码/2FA）**

| Secret 名称 | 值 |
|---|---|
| `LINUXDO_COOKIES` | 浏览器复制的 Cookie 字符串，如 `_t=xxx; _forum_session=yyy` |

> 获取方法：浏览器登录 [linux.do](https://linux.do/) → 按 `F12` → `Application`（应用）→ `Cookies` → `https://linux.do` → 把所有 Cookie 拼成 `名=值; 名=值` 字符串复制粘贴。
>
> **怎么拼**：把每行的 Name 和 Value 用 `=` 连接，多个 cookie 之间用「分号+空格」隔开，拼成一整行（中间不能换行），如 `_t=xxx; _forum_session=yyy; cf_clearance=zzz`。表格里 Value 显示是截断的（带 ...），需双击进单元格后 Ctrl+A / Ctrl+C 复制完整值。最关键的是 `_t`（长期登录凭证）和 `_forum_session`，其它 linux.do 域下的也可一并带上。嫌手动拼麻烦可装「Cookie-Editor」浏览器扩展，在 linux.do 页面点扩展 → Export → Header String 一键复制。

**方式二：账号密码登录**

| Secret 名称 | 值 |
|---|---|
| `LINUXDO_USERNAME` | 你的 LinuxDo 用户名或邮箱 |
| `LINUXDO_PASSWORD` | 你的 LinuxDo 密码 |

> 两者都配置时 Cookie 优先，Cookie 失效会自动回退到账号密码登录。

**可选 Secret：**

| Secret 名称 | 说明 |
|---|---|
| `BROWSE_ENABLED` | 是否浏览帖子，`true`/`false`，默认 `true` |
| `TELEGRAM_BOT_TOKEN` + `TELEGRAM_CHAT_ID` | Telegram 通知 |
| `GOTIFY_URL` + `GOTIFY_TOKEN` | Gotify 通知 |
| `SC3_PUSH_KEY` | Server酱³ 通知 |
| `WXPUSH_URL` + `WXPUSH_TOKEN` | wxpush 通知 |
| `WECOM_WEBHOOK_URL` | 企业微信群机器人通知（需改造版代码） |

> 以上全部**可选**，不配置不影响签到，只是收不到推送通知。添加方式与前面相同：仓库 `Settings` → `Secrets and variables` → `Actions` → `New repository secret`。

**可选项详细配置步骤：**

- **`BROWSE_ENABLED`**：一般**不用配**，默认 `true`（自动浏览帖子，保活需要）。只有想临时关闭浏览、只登录时才添加这个 Secret 并把值设为 `false`。

- **Telegram 通知**（推荐，免费）：
  1. 在 Telegram 搜索 `@BotFather`，发送 `/newbot`，按提示给机器人起名，创建成功后会得到 **Bot Token**（形如 `123456789:ABCdefGhIJKlmNoPQRsTUVwxyZ`）→ 添加 Secret：Name 填 `TELEGRAM_BOT_TOKEN`，Value 填这个 Token；
  2. 再搜索 `@userinfobot`，随便发一句话，它会回复你的数字 **用户 ID** → 添加 Secret：Name 填 `TELEGRAM_CHAT_ID`，Value 填这个数字；
  3. **重要**：找到你刚创建的机器人，给它发一句 `/start`（否则机器人无法主动给你发消息）。

- **Server酱³ 通知**（国内用户推荐，微信接收）：
  1. 打开 https://sc3.ft07.com/sendkey ，微信扫码登录，获取你的 **SendKey**（形如 `sctpxxxxt...`）；
  2. 添加 Secret：Name 填 `SC3_PUSH_KEY`，Value 填 SendKey。只需这一个即可。

- **Gotify 通知**（需要你有自建的 Gotify 服务器）：
  1. 登录你的 Gotify 后台 → `Apps` → `Create Application`，创建后复制该应用的 **Token**；
  2. 添加两个 Secret：`GOTIFY_URL` 填服务器地址（如 `https://your.gotify.server:8080`），`GOTIFY_TOKEN` 填应用 Token。

- **wxpush 通知**（需要你有自建的 wxpush 服务）：
  添加两个 Secret：`WXPUSH_URL` 填服务地址，`WXPUSH_TOKEN` 填对应 token（脚本会以 POST 方式请求 `{WXPUSH_URL}/wxsend`）。

- **企业微信群机器人通知**（需使用改造版代码，原版不支持）：
  1. 在企微群聊中：群设置 → 群机器人 → 添加机器人，复制 Webhook 地址（形如 `https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxx`）；
  2. 添加 Secret：Name 填 `WECOM_WEBHOOK_URL`，Value 填完整 Webhook 地址。

> 通知渠道可以同时配置多个，配了哪个就推哪个；一个都不配则自动跳过通知环节。

#### 2.5 多账号保活（需使用改造版代码）

原版只支持单账号；改造版（见 d:\CursorWork\linuxdo-checkin\ 目录下的 main.py / notify.py / README.md / .github/workflows/daily-check-in.yml）支持多账号：

- `LINUXDO_COOKIES`：**每行一个账号**的 Cookie 字符串（Cookie 内部含分号，只能按行分隔）：
  ```
  _t=账号1的cookie; _forum_session=xxx
  _t=账号2的cookie; _forum_session=yyy
  ```
- `LINUXDO_USERNAME` / `LINUXDO_PASSWORD`：多账号按行或 `&` 分隔，顺序一一对应：`user1&user2` / `pass1&pass2`；
- 第 i 个账号优先用第 i 行 Cookie，失效时自动回退到第 i 组账号密码；单账号用法不变；
- 所有账号处理完会合并发一条通知，包含每个账号的结果。

应用方法：把改造版的 4 个文件覆盖到你 fork 的仓库（GitHub 网页版逐个文件点 Edit 粘贴保存，或本地 git push）。注意：覆盖后建议关闭/删除 `sync.yml` 自动同步上游的 workflow，否则每天同步上游会把改动冲掉。

#### 3. 启用并手动触发一次工作流

1. 进入仓库的 `Actions` 选项卡（Fork 后首次进入需点击按钮启用 workflows）；
2. 左侧选择 **Daily Check-in**；
3. 点 `Run workflow` → 选 `main` 分支 → `Run workflow` 手动跑一次验证。

此后工作流会按 `daily-check-in.yml` 里的定时规则 **每 12 小时自动运行一次**（cron: `0 */12 * * *`），无需再手动操作。

#### 4. 查看运行结果

`Actions` → 点最新的 `Daily Check-in` 运行记录 → `run_script` → `Execute script`，可看到登录日志和 **Connect Info** 信任等级进度表（新号可能为空，多挂几天就有了）。

#### 5. 保活相关注意事项

- **防止定时任务被 GitHub 停用**：GitHub 会在仓库 60 天无活动后自动停用定时工作流。仓库自带 `immortality.yml`（GitHub Workflow Immortality）每月自动"续命"，但需要你配置一个名为 `ACTIONS_TRIGGER_PAT` 的 Secret（值为你的 GitHub Personal Access Token，需 `workflow` 权限）；不配置的话，收到 GitHub 的停用提醒邮件时手动点一次 "Enable" 即可。
- **自动同步上游更新**：仓库自带 `sync.yml`，每天 0 点（UTC）自动同步原作者仓库的更新；若上游改了 workflow 文件导致同步失败，需要手动在仓库页面点一次 `Sync fork`。
- **登录偶尔失败是正常的**：重试一下即可；嫌 GitHub 失败邮件烦可以在 GitHub 通知设置里关掉 Actions 邮件通知。
- **Cookie 会过期**：Cookie 登录失效后（日志报 "Cookie 可能已过期"），重新从浏览器复制新 Cookie 更新 `LINUXDO_COOKIES` 即可；或同时配好账号密码作为兜底。

### 方式 B：青龙面板（有自己服务器/NAS 的用户）

> 注意：docker 版青龙请使用 `whyour/qinglong:debian` 镜像，alpine 版可能装不上依赖。

1. **安装依赖**
   - 依赖管理 → 安装依赖 → 类型选 `python3`、自动拆分选 `是`，名称填入：
     ```
     DrissionPage==4.1.0.18
     wcwidth==0.2.13
     tabulate==0.9.0
     loguru==0.7.2
     curl-cffi
     bs4
     ```
   - 依赖管理 → 安装 **Linux 依赖** → 名称填 `chromium`（失败先执行 `apt update`）。
2. **添加订阅**：订阅管理 → 创建订阅
   - 名称：`Linux.DO 签到`；类型：公开仓库
   - 链接：`https://github.com/doveppp/linuxdo-checkin.git`；分支：`main`
   - 定时类型：`crontab`；定时规则：`0 0 * * *`（这是拉取代码更新的时间）
3. **配置环境变量**：环境变量 → 创建变量，填 `LINUXDO_COOKIES`（或 `LINUXDO_USERNAME` + `LINUXDO_PASSWORD`），及上文可选通知变量。
4. **运行**：脚本自带 cron `0 */6 * * *`（每 6 小时一次）；首次可在订阅处手动点"运行"拉取脚本，之后在 定时任务 → `Linux.Do 签到` → `日志` 查看结果。

## 三、总结（最快上手路径）

1. Fork https://github.com/doveppp/linuxdo-checkin
2. 仓库 Settings → Secrets → Actions 里添加 `LINUXDO_COOKIES`（浏览器 F12 复制）
3. Actions 选项卡启用工作流，手动 Run 一次 **Daily Check-in** 验证成功
4. 之后每 12 小时自动签到保活；（可选）配 `ACTIONS_TRIGGER_PAT` 防止 60 天后定时任务被停用，配 Telegram/Server酱 接收每日签到结果通知
