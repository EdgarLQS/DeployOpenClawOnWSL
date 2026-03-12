# OpenClaw v2026.3.2 安装与硅基流动 DeepSeek 配置完整指南

> **文档版本：** 2026-03-10
> **适用环境：** Windows 11 + WSL2
> **OpenClaw 版本：** 2026.3.2

---

## 目录

1. [环境准备](#1-环境准备)
2. [安装 OpenClaw](#2-安装-openclaw)
3. [配置硅基流动 DeepSeek API](#3-配置硅基流动-deepseek-api)
4. [启动 Gateway](#4-启动-gateway)
5. [浏览器访问配置](#5-浏览器访问配置)
6. [使用示例](#6-使用示例)
7. [常见问题与解决方案](#7-常见问题与解决方案)

---

## 1. 环境准备

### 系统要求

- **操作系统：** Windows 11 + WSL2
- **Node.js：** >= 22.12.0 (推荐 v24+)
- **包管理器：** pnpm (推荐) 或 npm

### 检查环境

```bash
# 检查 Node.js 版本
node --version
# 输出示例：v24.13.0

# 检查 pnpm
pnpm --version
# 如未安装：npm install -g pnpm
```

---

## 2. 安装 OpenClaw

### 方式一：npm 全局安装（推荐）

```bash
# 安装指定版本
npm install -g openclaw@2026.3.2

# 验证安装
openclaw --version
# 输出：2026.3.2
```

### 方式二：从源码安装

```bash
# 进入源码目录
cd /mnt/d/claude/openclaw/openclaw

# 安装依赖
pnpm install

# 构建项目
pnpm build

# 直接运行
pnpm openclaw --version
```

### 方式三：安装脚本

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --version 2026.3.2
```

---

## 3. 配置硅基流动 DeepSeek API

### 3.1 获取 API Key

1. 访问 [硅基流动控制台](https://cloud.siliconflow.cn/)
2. 登录/注册账号
3. 进入 **API Keys** 页面
4. 创建新的 API Key（格式：`sk-xxxxxxxxxxxxxxxx`）

### 3.2 验证 API Key（可选）

```bash
curl -s -X POST "https://api.siliconflow.cn/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-你的 APIKey" \
  -d '{
    "model": "deepseek-ai/DeepSeek-V3",
    "messages": [{"role": "user", "content": "Hello"}],
    "max_tokens": 10
  }'
```

### 3.3 创建配置文件

编辑 `~/.openclaw/openclaw.json`：

```bash
mkdir -p ~/.openclaw
nano ~/.openclaw/openclaw.json
```

**完整配置内容：**

```json5
{
  // 环境变量配置
  "env": {
    "SILICONFLOW_API_KEY": "sk-你的 APIKey"
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

  // 默认模型设置
  "agents": {
    "defaults": {
      "model": {
        "primary": "siliconflow/deepseek-ai/DeepSeek-V3",
        "fallbacks": ["siliconflow/deepseek-ai/DeepSeek-R1"]
      },
      "models": {
        "siliconflow/deepseek-ai/DeepSeek-V3": {
          "alias": "DeepSeek-V3"
        },
        "siliconflow/deepseek-ai/DeepSeek-R1": {
          "alias": "DeepSeek-R1"
        }
      }
    }
  },

  // 自定义 SiliconFlow 提供商配置
  "models": {
    "mode": "merge",
    "providers": {
      "siliconflow": {
        "baseUrl": "https://api.siliconflow.cn/v1",
        "apiKey": "sk-你的 APIKey",
        "api": "openai-completions",
        "models": [
          {
            "id": "deepseek-ai/DeepSeek-V3",
            "name": "DeepSeek V3"
          },
          {
            "id": "deepseek-ai/DeepSeek-R1",
            "name": "DeepSeek R1"
          }
        ]
      }
    }
  }
}
```

### 3.4 验证配置

```bash
# 查看模型状态
openclaw models status

# 预期输出包含：
# Default       : siliconflow/deepseek-ai/DeepSeek-V3
# siliconflow effective=models.json:sk-xxx...
```

---

## 4. 启动 Gateway

### 4.1 启动服务

```bash
# 作为守护进程启动（推荐）
openclaw daemon start

# 或重启服务
openclaw daemon restart
```

### 4.2 检查状态

```bash
# 查看 Gateway 状态
openclaw gateway status

# 查看健康状态
openclaw health

# 查看日志
tail -f /tmp/openclaw/openclaw-2026-03-10.log
```

### 4.3 预期输出

```
Runtime: running (pid xxxxx, state active, sub running)
RPC probe: ok
Listening: 127.0.0.1:18789
Dashboard: http://127.0.0.1:18789/
```

---

## 4.4 启动/停止 Gateway

### 守护进程管理（推荐）

```bash
# 启动
openclaw daemon start

# 停止
openclaw daemon stop

# 重启
openclaw daemon restart

# 查看状态
openclaw daemon status

# 卸载服务
openclaw daemon uninstall
```

### 前台运行（开发调试）

```bash
# 前台运行，查看实时日志
openclaw gateway --port 18789 --verbose

# 按 Ctrl+C 停止
```

### systemd 服务管理（WSL2/Linux）

```bash
# 使用 systemctl 管理
systemctl --user start openclaw-gateway
systemctl --user stop openclaw-gateway
systemctl --user restart openclaw-gateway
systemctl --user status openclaw-gateway

# 查看日志
journalctl --user -u openclaw-gateway -f
```

---

## 4.5 自动任务（Cron Jobs）

OpenClaw 内置 cron 调度器，可以定时执行 AI 任务。

### 基本语法

```bash
openclaw cron add --name "任务名称" --every "10m" --message "你的提示词"
```

### 常用示例

#### 1. 每 10 分钟执行一次

```bash
openclaw cron add \
  --name "天气提醒" \
  --every "10m" \
  --message "请告诉我当前的天气情况" \
  --thinking low
```

#### 2. 每天早上 8 点执行（5 字段 cron）

```bash
openclaw cron add \
  --name "早间新闻" \
  --cron "0 8 * * *" \
  --message "请总结今天的头条新闻" \
  --thinking medium
```

#### 3. 每周一上午 9 点执行

```bash
openclaw cron add \
  --name "周报生成" \
  --cron "0 9 * * 1" \
  --message "请帮我生成本周工作总结" \
  --thinking high
```

#### 4. 每小时执行并发送到指定频道

```bash
openclaw cron add \
  --name "定时检查" \
  --every "1h" \
  --message "检查系统状态" \
  --channel "telegram" \
  --to "+1234567890" \
  --announce
```

#### 5. 一次性任务（20 分钟后执行）

```bash
openclaw cron add \
  --name "提醒会议" \
  --at "+20m" \
  --message "提醒我 20 分钟后有会议" \
  --delete-after-run
```

### Cron 表达式格式

```
分 时 日 月 周
```

**示例：**

| 表达式 | 说明 |
|--------|------|
| `*/10 * * * *` | 每 10 分钟 |
| `0 * * * *` | 每小时整点 |
| `0 8 * * *` | 每天早上 8 点 |
| `0 9 * * 1-5` | 每周一到五上午 9 点 |
| `30 14 * * *` | 每天下午 2 点 30 分 |
| `0 0 1 * *` | 每月 1 号零点 |

### 管理定时任务

```bash
# 列出所有任务
openclaw cron list

# 查看任务状态
openclaw cron status

# 启用任务
openclaw cron enable --name "任务名称"

# 禁用任务
openclaw cron disable --name "任务名称"

# 立即执行任务（测试）
openclaw cron run --name "任务名称"

# 查看执行历史
openclaw cron runs --name "任务名称"

# 编辑任务
openclaw cron edit --name "任务名称" --cron "0 9 * * *"

# 删除任务
openclaw cron rm --name "任务名称"
```

### 高级选项

```bash
# 指定模型
openclaw cron add \
  --name "复杂任务" \
  --every "1h" \
  --message "分析问题" \
  --model "siliconflow/deepseek-ai/DeepSeek-R1" \
  --thinking high

# 指定 Agent
openclaw cron add \
  --name "工作助手" \
  --every "30m" \
  --message "检查工作邮件" \
  --agent "work"

# 设置超时
openclaw cron add \
  --name "快速检查" \
  --every "5m" \
  --message "检查状态" \
  --timeout "30000"
```

### 查看任务执行日志

```bash
# 查看最近的执行记录
openclaw cron runs --limit 10

# 查看特定任务的执行记录
openclaw cron runs --name "任务名称" --limit 5
```

---

## 5. 浏览器访问配置

### 5.1 访问地址

在 **Windows 浏览器**中访问：
- **http://localhost:18789**

WSL2 会自动转发端口，无需额外配置。

### 5.2 认证配置

#### 方式一：无认证模式（开发环境）

```json5
{
  "gateway": {
    "auth": {
      "mode": "none"
    }
  }
}
```

#### 方式二：Token 认证模式（生产环境）

```json5
{
  "gateway": {
    "auth": {
      "mode": "token",
      "token": "自定义你的 token"
    }
  }
}
```

配置后重启 Gateway：
```bash
openclaw daemon restart
```

### 5.3 Control UI 设置

1. 打开 http://localhost:18789
2. 点击 **Reload** 按钮
3. 如使用 Token 认证，在设置中输入 Token
4. 在 **Raw JSON5** 编辑器中可以修改配置

---

## 6. 使用示例

### 6.1 CLI 命令

```bash
# AI 对话
openclaw agent --agent main --message "你好，请介绍一下自己" --thinking low

# 查看模型列表
openclaw models list

# 查看频道状态
openclaw channels status --probe
```

### 6.2 Web 界面

1. 打开浏览器访问 http://localhost:18789
2. 在聊天界面输入消息
3. 在 Raw JSON5 中修改配置
4. 查看会话历史和日志

### 6.3 切换模型

在 Raw JSON5 中修改：

```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "siliconflow/deepseek-ai/DeepSeek-R1"
      }
    }
  }
}
```

保存后点击 Reload 即可。

---

## 6.4 配置 Tavily 搜索 API（用于实时信息查询）

### 获取 Tavily API Key

1. 访问 [https://app.tavily.com/](https://app.tavily.com/)
2. 注册账号（免费 1000 次/月）
3. 复制 API Key（格式：`tvly-xxxxxxxx`）

### 配置 Tavily

```bash
openclaw config set tools.web.search.apiKey "tvly-你的 Key"
openclaw daemon restart
```

或在 Raw JSON5 中添加：

```json5
{
  "tools": {
    "web": {
      "search": {
        "apiKey": "tvly-你的 Key"
      }
    }
  }
}
```

### 验证配置

```bash
openclaw agent --message "搜索今天的科技新闻"
```

---

## 7. 常见问题与解决方案

### 7.1 API Key 无效

**症状：** `401 status code (no body)`

**原因：** API Key 配置错误或已过期

**解决方案：**
1. 检查 API Key 是否正确复制（无多余空格）
2. 在 provider 控制台验证 Key 状态
3. 重新生成 API Key

**验证命令：**
```bash
curl -s -X POST "https://api.siliconflow.cn/v1/chat/completions" \
  -H "Authorization: Bearer sk-你的 Key" \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek-ai/DeepSeek-V3","messages":[{"role":"user","content":"Hi"}]}'
```

### 7.2 Tavily 搜索不可用

**症状：**
```
missing_brave_api_key
web_search needs a Brave Search API key
```

**原因：** 未配置 Web Search API

**解决方案：**
1. 访问 https://app.tavily.com/ 注册获取 API Key（免费 1000 次/月）
2. 配置：
```bash
openclaw config set tools.web.search.apiKey "tvly-你的 Key"
openclaw daemon restart
```

### 7.3 Gateway 无法启动

**症状：** `Gateway start blocked: set gateway.mode=local`

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

### 7.3 浏览器访问显示 401/未授权

**症状：** `unauthorized: gateway token missing` 或 `rate_limited`

**解决方案一：** 清除浏览器缓存
1. 按 `Ctrl + Shift + Delete`
2. 清除"缓存的图片和文件"、"Cookie"
3. 重新访问

**解决方案二：** 使用无痕模式
- Chrome: `Ctrl + Shift + N`
- Edge: `Ctrl + Shift + N`

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

### 7.4 认证限流

**症状：** `too many failed authentication attempts (retry later)`

**解决方案：**
1. 停止并重启 Gateway：
   ```bash
   openclaw daemon stop
   openclaw daemon start
   ```
2. 清除浏览器缓存
3. 或使用无痕模式访问

### 7.5 模型响应失败

**症状：** `All models failed` 或连接超时

**检查步骤：**
```bash
# 1. 检查 Gateway 状态
openclaw gateway status

# 2. 查看日志
cat /tmp/openclaw/openclaw-2026-03-10.log | tail -50

# 3. 测试 API Key
curl -s -X POST "https://api.siliconflow.cn/v1/chat/completions" \
  -H "Authorization: Bearer sk-你的 Key" \
  -d '{"model":"deepseek-ai/DeepSeek-V3","messages":[{"role":"user","content":"Hi"}]}'
```

### 7.6 Windows 无法访问 WSL2 端口

**症状：** 浏览器无法连接 localhost:18789

**解决方案：**
1. 确认 WSL2 版本 >= 2.0
2. 在 PowerShell 中执行：
   ```powershell
   netsh interface portproxy delete v4tov4 listenport=18789 listenaddress=0.0.0.0
   netsh interface portproxy add v4tov4 listenport=18789 listenaddress=0.0.0.0 connectport=18789 connectaddress=$(wsl hostname -I | awk '{print $1}')
   ```
3. 或在 WSL 内修改绑定地址为 `0.0.0.0`

---

## 附录：配置文件位置

| 文件/目录 | 路径 | 说明 |
|-----------|------|------|
| 主配置 | `~/.openclaw/openclaw.json` | 主要配置文件 |
| 配置备份 | `~/.openclaw/openclaw.json.bak` | 自动备份 |
| 工作区 | `~/.openclaw/workspace` | Agent 工作区 |
| 凭证 | `~/.openclaw/credentials/` | 频道认证信息 |
| 会话 | `~/.openclaw/agents/<id>/sessions/` | 会话历史 |
| 日志 | `/tmp/openclaw/` | 运行日志 |

---

## 附录：常用命令速查

### 基础命令

```bash
# 版本检查
openclaw --version

# 帮助
openclaw --help
openclaw <command> --help
```

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

# 前台运行（调试）
openclaw gateway --port 18789 --verbose
```

### 模型与频道

```bash
openclaw models status
openclaw models list
openclaw channels status --probe
```

### 配置操作

```bash
openclaw config get gateway.mode
openclaw config set gateway.auth.mode "none"
openclaw config unset gateway.auth.token
```

### AI 对话

```bash
# 测试对话
openclaw agent --agent main --message "Hello" --thinking low

# 指定模型
openclaw agent --agent main --message "分析问题" --model "siliconflow/deepseek-ai/DeepSeek-R1" --thinking high
```

### 定时任务（Cron）

```bash
# 列出任务
openclaw cron list

# 添加任务
openclaw cron add --name "提醒" --every "10m" --message "检查状态"

# 启用/禁用任务
openclaw cron enable --name "提醒"
openclaw cron disable --name "提醒"

# 立即执行（测试）
openclaw cron run --name "提醒"

# 查看执行历史
openclaw cron runs --name "提醒"

# 删除任务
openclaw cron rm --name "提醒"
```

### 日志查看

```bash
# 实时日志
tail -f /tmp/openclaw/openclaw-*.log

# 最近 100 行
cat /tmp/openclaw/openclaw-*.log | tail -100

# systemd 日志（WSL2）
journalctl --user -u openclaw-gateway -f
```

---

**文档更新时间：** 2026-03-10
**OpenClaw 版本：** 2026.3.2
