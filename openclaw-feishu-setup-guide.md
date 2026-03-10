# OpenClaw 飞书频道配置完整指南

> **文档版本：** 2026-03-10
> **适用环境：** Windows 11 + WSL2
> **OpenClaw 版本：** 2026.3.2
> **飞书插件版本：** @openclaw/feishu@2026.3.7

---

## 目录

1. [前置准备](#1-前置准备)
2. [安装飞书插件](#2-安装飞书插件)
3. [飞书开放平台配置](#3-飞书开放平台配置)
4. [OpenClaw 配置](#4-openclaw-配置)
5. [长连接建立](#5-长连接建立)
6. [飞书配对](#6-飞书配对)
7. [创建定时任务](#7-创建定时任务)
8. [遇到的问题与解决方案](#8-遇到的问题与解决方案)
9. [常用命令速查](#9-常用命令速查)

---

## 1. 前置准备

### 环境要求

- OpenClaw v2026.3.2 已安装并运行
- Gateway 已启动 (`openclaw daemon start`)
- 有效的飞书企业账号（可创建应用）

### 检查 Gateway 状态

```bash
openclaw gateway status
openclaw health
```

---

## 2. 安装飞书插件

```bash
openclaw plugins install @openclaw/feishu
```

**预期输出：**
```
Installed @openclaw/feishu@2026.3.7
```

---

## 3. 飞书开放平台配置

### 步骤 3.1：创建应用

1. 访问 [飞书开放平台](https://open.feishu.cn/app)
2. 点击 **创建企业应用**
3. 填写应用信息：
   - 应用名称：如 "OpenClaw 助手"
   - 应用描述：可选
   - 应用图标：可选

### 步骤 3.2：获取凭证

1. 进入 **凭证与基础信息** 页面
2. 复制以下信息：
   - **App ID**（格式：`cli_xxxxxxxxxxxx`）
   - **App Secret**（点击"查看"后复制）

> ⚠️ **重要：** App Secret 只显示一次，请妥善保存！

### 步骤 3.3：配置权限

在 **权限管理** 页面，点击 **批量导入**，粘贴以下 JSON：

```json
{
  "scopes": {
    "tenant": [
      "im:message",
      "im:message:send_as_bot",
      "im:message.p2p_msg:readonly",
      "im:chat.access_event.bot_p2p_chat:read",
      "im:chat.members:bot_access",
      "cardkit:card:read",
      "cardkit:card:write"
    ],
    "user": ["im:chat.access_event.bot_p2p_chat:read"]
  }
}
```

### 步骤 3.4：启用机器人

1. 进入 **应用功能** → **机器人**
2. 点击 **启用机器人**
3. 设置机器人名称和头像

### 步骤 3.5：发布应用

1. 进入 **版本管理与发布**
2. 点击 **创建版本**
3. 提交审核并发布（企业应用通常自动通过）

---

## 4. OpenClaw 配置

### 方式一：使用交互式命令（推荐）

```bash
openclaw channels add
```

1. 选择 **Feishu**
2. 输入 **App ID**
3. 输入 **App Secret**

### 方式二：手动编辑配置文件

```bash
nano ~/.openclaw/openclaw.json
```

添加以下配置：

```json5
{
  "channels": {
    "feishu": {
      "enabled": true,
      "dmPolicy": "pairing",
      "accounts": {
        "default": {
          "appId": "cli_a925099aa7f89ccb",
          "appSecret": "你的 App Secret"
        }
      }
    }
  }
}
```

### 重启 Gateway

```bash
openclaw daemon restart
```

---

## 5. 长连接建立

### 步骤 5.1：确认 Gateway 已启动

```bash
openclaw daemon status
# 应显示：Runtime: running
```

### 步骤 5.2：查看连接日志

```bash
openclaw logs --follow | grep -i feishu
```

**预期日志：**
```
feishu[default]: starting WebSocket connection...
feishu[default]: WebSocket client started
```

### 步骤 5.3：在飞书后台保存事件订阅

1. 访问 [飞书开放平台 - 事件订阅](https://open.feishu.cn/app)
2. 选择你的应用
3. **事件订阅** → **使用长连接接收事件**
4. 添加事件：`im.message.receive_v1`
5. 点击 **保存**

> ✅ **成功标志：** 不再提示"未检测到应用连接信息"

---

## 6. 飞书配对

### 步骤 6.1：添加机器人为好友

1. 在飞书客户端搜索机器人名称
2. 添加好友或直接发送消息

### 步骤 6.2：获取配对码

机器人会回复类似消息：
```
OpenClaw: access not configured.
Your Feishu user id: ou_xxxxxxxxxxxxx
Pairing code: XXXXXXXX
Ask the bot owner to approve with:
openclaw pairing approve feishu XXXXXXXX
```

### 步骤 6.3：批准配对

```bash
openclaw pairing approve feishu <配对码>
```

**预期输出：**
```
Approved feishu sender ou_xxxxxxxxxxxxx
```

### 步骤 6.4：测试聊天

在飞书中给机器人发消息，机器人应该正常回复。

---

## 7. 创建定时任务

### 示例 1：每分钟时间汇报

```bash
openclaw cron add \
  --name "每分钟时间汇报" \
  --every "1m" \
  --message "请告诉我当前的时间" \
  --thinking low \
  --channel "feishu" \
  --to "ou_xxxxxxxxxxxxx" \
  --announce
```

### 示例 2：每小时港股波动汇报

```bash
openclaw cron add \
  --name "港股波动汇报" \
  --every "1h" \
  --message "请搜索并汇报当前港股市场的波动情况" \
  --thinking low \
  --channel "feishu" \
  --to "ou_xxxxxxxxxxxxx" \
  --announce
```

### 示例 3：每天早间新闻

```bash
openclaw cron add \
  --name "早间新闻" \
  --cron "0 9 * * *" \
  --message "请总结昨天的科技新闻" \
  --thinking medium \
  --channel "feishu" \
  --to "ou_xxxxxxxxxxxxx" \
  --announce
```

---

## 8. 遇到的问题与解决方案

### 问题 1：API Key 无效（401 错误）

**症状：**
```
401 status code (no body)
```

**原因：** API Key 配置错误或已过期

**解决方案：**
1. 检查 API Key 是否正确复制（无空格）
2. 在provider 控制台验证 Key 状态
3. 重新生成 API Key

---

### 问题 2：Gateway 无法启动

**症状：**
```
Gateway start blocked: set gateway.mode=local
```

**原因：** 缺少 `gateway.mode` 配置

**解决方案：**
```bash
openclaw config set gateway.mode "local"
openclaw daemon restart
```

或在配置文件中添加：
```json5
{
  "gateway": {
    "mode": "local"
  }
}
```

---

### 问题 3：浏览器访问显示 401 未授权

**症状：**
```
unauthorized: gateway token missing
rate_limited
```

**原因：** 认证 Token 未配置或多次失败触发限流

**解决方案一：** 清除浏览器缓存
- 按 `Ctrl + Shift + Delete`
- 清除"缓存的图片和文件"、"Cookie"
- 重新访问

**解决方案二：** 使用无痕模式
- Chrome/Edge: `Ctrl + Shift + N`

**解决方案三：** 临时关闭认证
```json5
{
  "gateway": {
    "auth": {
      "mode": "none"
    }
  }
}
```

---

### 问题 4：搜索功能不可用（缺少搜索 API）

**症状：**
```
missing_brave_api_key
web_search needs a Brave Search API key
```

**原因：** 未配置 Web Search API

**解决方案：配置 Tavily（免费 1000 次/月）**

1. 访问 https://app.tavily.com/ 注册获取 API Key
2. 配置：
```bash
openclaw config set tools.web.search.apiKey "tvly-xxxxx"
openclaw daemon restart
```

**配置文件方式：**
```json5
{
  "tools": {
    "web": {
      "search": {
        "apiKey": "tvly-xxxxx"
      }
    }
  }
}
```

---

### 问题 5：飞书插件已安装但未配置

**症状：**
```
Feishu: enabled, not configured, stopped, error:not configured
```

**原因：** 飞书凭证（App ID/Secret）未配置

**解决方案：**
```bash
openclaw channels add
# 选择 Feishu，输入 App ID 和 App Secret
openclaw daemon restart
```

---

### 问题 6：飞书长连接建立失败（400 错误）

**症状：**
```
status: 400 Bad Request
code: 10014, msg: app secret invalid
unable to connect to the server after trying 3 times
```

**原因：** App Secret 无效或复制错误

**解决方案：**
1. 在飞书开放平台重新复制 App Secret
2. 或重置 App Secret 后重新配置
3. 验证凭证：
```bash
curl -s -X POST "https://open.feishu.cn/open-apis/auth/v3/app_access_token/internal" \
  -H "Content-Type: application/json" \
  -d '{
    "app_id": "你的 App ID",
    "app_secret": "你的 App Secret"
  }'
```

**预期输出：**
```json
{"app_access_token":"xxx","code":0,"msg":"ok"}
```

---

### 问题 7：飞书后台提示"未检测到应用连接信息"

**症状：**
```
未检测到应用连接信息，请确保长连接建立成功后再保存配置
```

**原因：** 长连接尚未建立或建立失败

**解决方案：**
1. 确认 Gateway 已启动
2. 确认飞书凭证已正确配置
3. 查看日志确认长连接状态：
```bash
openclaw logs --follow | grep -i feishu
```
4. 看到 `WebSocket client started` 后，再回飞书后台保存

---

### 问题 8：飞书机器人不回复消息

**症状：** 发送消息后机器人无响应

**可能原因及解决方案：**

| 原因 | 检查方法 | 解决方案 |
|------|----------|----------|
| 未配对 | `openclaw pairing list feishu` | `openclaw pairing approve feishu <码>` |
| 事件未订阅 | 飞书后台检查事件订阅 | 添加 `im.message.receive_v1` 事件 |
| 长连接断开 | `openclaw logs --follow` | `openclaw daemon restart` |
| 权限不足 | 飞书后台检查权限 | 批量导入所需权限 |

---

### 问题 9：定时任务不执行

**症状：** 定时任务创建后无执行记录

**检查步骤：**
```bash
# 1. 查看任务列表
openclaw cron list

# 2. 查看任务状态
openclaw cron status

# 3. 查看执行历史
openclaw cron runs --name "任务名称"

# 4. 查看日志
openclaw logs --follow | grep cron
```

**常见原因：**
- 任务被禁用：`openclaw cron enable --name "任务名称"`
- 频道未连接：检查频道状态
- 用户 ID 错误：确认飞书用户 open_id 正确

---

### 问题 10：插件重复 ID 警告

**症状：**
```
plugins.entries.feishu: plugin feishu: duplicate plugin id detected
```

**原因：** 插件在多个位置被加载

**影响：** 通常不影响功能，可忽略

**解决方案（可选）：**
```bash
# 清理重复安装
openclaw plugins uninstall feishu
openclaw plugins install @openclaw/feishu
```

---

## 9. 常用命令速查

### Gateway 管理

```bash
# 启动/停止/重启
openclaw daemon start
openclaw daemon stop
openclaw daemon restart

# 状态检查
openclaw daemon status
openclaw gateway status
openclaw health
```

### 飞书频道

```bash
# 添加频道
openclaw channels add

# 查看状态
openclaw channels status --probe

# 查看日志
openclaw logs --follow | grep -i feishu
```

### 飞书配对

```bash
# 查看配对请求
openclaw pairing list feishu

# 批准配对
openclaw pairing approve feishu <配对码>

# 查看已配对用户
openclaw pairing list feishu --approved
```

### 定时任务

```bash
# 创建任务
openclaw cron add --name "任务名" --every "1h" --message "消息内容"

# 列出任务
openclaw cron list

# 查看状态
openclaw cron status

# 查看执行历史
openclaw cron runs --name "任务名"

# 启用/禁用
openclaw cron enable --name "任务名"
openclaw cron disable --name "任务名"

# 立即执行（测试）
openclaw cron run --name "任务名"

# 删除任务
openclaw cron rm --name "任务名"
```

### 配置管理

```bash
# 获取配置
openclaw config get gateway.mode

# 设置配置
openclaw config set tools.web.search.apiKey "xxx"

# 删除配置
openclaw config unset gateway.auth.token

# 验证配置
openclaw config validate
```

---

## 附录：配置示例

### 完整配置文件示例

```json5
{
  // 环境变量
  "env": {
    "SILICONFLOW_API_KEY": "sk-xxx",
    "TAVILY_API_KEY": "tvly-xxx"
  },

  // 模型配置
  "agents": {
    "defaults": {
      "model": {
        "primary": "siliconflow/deepseek-ai/DeepSeek-V3",
        "fallbacks": ["siliconflow/deepseek-ai/DeepSeek-R1"]
      }
    }
  },

  // 搜索配置
  "tools": {
    "web": {
      "search": {
        "apiKey": "tvly-xxx"
      }
    }
  },

  // 网关配置
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "loopback",
    "auth": {
      "mode": "none"
    }
  },

  // 飞书频道配置
  "channels": {
    "feishu": {
      "enabled": true,
      "dmPolicy": "pairing",
      "accounts": {
        "default": {
          "appId": "cli_xxx",
          "appSecret": "xxx"
        }
      }
    }
  }
}
```

---

## 附录：飞书 ID 获取方法

### 获取用户 open_id

**方法 1：通过配对日志**
```bash
openclaw logs --follow | grep open_id
```

**方法 2：通过配对命令**
```bash
openclaw pairing list feishu
```

### 获取群聊 chat_id

**方法 1：通过日志**
1. 将机器人拉入群聊
2. 在群里 @机器人
3. 查看日志获取 `chat_id`

**方法 2：通过飞书 API**
访问 [飞书 API 调试工具](https://open.feishu.cn/tool)

---

**文档更新时间：** 2026-03-10
**OpenClaw 版本：** 2026.3.2
