# QuartexNode 自动续期脚本

通过 GitHub Actions 每 12 小时自动为 QuartexNode 服务器执行续期操作，并通过 Telegram 推送运行结果。

---

## 功能特性

- 🕐 每 12 小时自动运行（GitHub Actions 托管，无需服务器）
- 🔐 Token 失效后自动重新登录
- 📲 Telegram 通知（成功 / 暂无需续期 / 失败 / 异常）
- 📝 每次运行时间记录到 `time.txt`
- 🔒 所有敏感信息通过 GitHub Secrets 管理，代码中无明文

---

## 文件结构

```
.
├── rew.py                        # 主脚本
├── time.txt                      # 运行时间记录（自动生成）
├── README.md
└── .github/
    └── workflows/
        └── renew.yml             # GitHub Actions 工作流
```

---

## 部署步骤

### 1. Fork 或克隆本仓库

### 2. 配置 GitHub Secrets

进入仓库 → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**，依次添加：

| Secret 名称 | 说明 | 示例 |
|---|---|---|
| `QUARTEX_EMAIL` | 登录邮箱 | `your@email.com` |
| `QUARTEX_PASSWORD` | 登录密码 | `your_password` |
| `QUARTEX_SERVER_ID` | 目标服务器 ID | `5070` |
| `TG_CONFIG` | Telegram 配置 | `123456789 AABBccDDee...` |

> **TG_CONFIG 格式**：`chat_id` 和 `bot_token` 之间用**空格**分隔。

### 3. 获取 Telegram 参数

1. 在 Telegram 搜索 `@BotFather`，发送 `/newbot` 创建机器人，获得 `bot_token`
2. 给机器人发一条任意消息，然后访问：
   ```
   https://api.telegram.org/bot<bot_token>/getUpdates
   ```
   在返回的 JSON 中找到 `result[0].message.chat.id`，即为 `chat_id`

### 4. 启用 Actions

仓库 → **Actions** → 若提示需要启用，点击 **Enable** 即可。

---

## 运行时机

| 触发方式 | 时间 |
|---|---|
| 定时任务 | 每天 UTC 00:00 和 12:00（北京时间 08:00 和 20:00） |
| 手动触发 | Actions → `QuartexNode 自动续期` → `Run workflow` |

---

## Telegram 通知说明

| 状态 | 触发条件 |
|---|---|
| ✅ 续期成功 | 服务器剩余时间 < 24 小时，续期请求被接受 |
| 💬 暂无需续期 | 服务器剩余时间 > 24 小时，服务器返回 400 |
| 🔑 Token 失效 | 请求返回 401 / 403，下次执行将自动重新登录 |
| ❌ 登录 / 续期失败 | 账号密码错误或服务器异常 |
| 🔌 网络异常 | 请求超时或连接失败 |

---

## 运行记录

每次执行后会在仓库根目录的 `time.txt` 中追加一行时间戳，例如：

```
2026-05-31 00:00:01 UTC
2026-05-31 12:00:02 UTC
2026-06-01 00:00:01 UTC
```

---

## 本地运行

```bash
pip install requests

export QUARTEX_EMAIL="your@email.com"
export QUARTEX_PASSWORD="your_password"
export QUARTEX_SERVER_ID="5070"
export TG_CONFIG="123456789 AABBccDDee..."

python rew.py
```

本地运行会进入每小时轮询循环，使用 `Ctrl+C` 退出。
