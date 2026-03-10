# OpenClaw + Claude Code 完整集成指南

> **文档版本:** 2026-03-10
> **适用环境:** Windows 11 + WSL2
> **特色:** 利用硅基流动免费 API 额度，零成本体验 AI 助手

---

## 快速导航

| 阶段 | 内容 | 预计时间 |
|------|------|----------|
| 1. Claude Code | 安装与配置硅基流动 API | 10 分钟 |
| 2. OpenClaw | 安装与模型配置 | 15 分钟 |
| 3. 飞书集成 | 机器人配置与配对 | 20 分钟 |
| 4. 定时任务 | 自动汇报任务创建 | 5 分钟 |

---

## 阶段 1: 安装 Claude Code 并配置硅基流动 API

### 1.1 WSL2 环境检查

```bash
# 检查 WSL 版本
wsl --version

# 检查 Node.js (需要 >= 22.12.0)
node --version
```

### 1.2 安装 Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

### 1.3 配置硅基流动 API (关键步骤)

**重要:** 硅基流动提供免费的 API 额度，适合学习和测试。
**获取 API Key:**
1. 访问 [硅基流动控制台](https://cloud.siliconflow.cn/i/NCHmBiGa)
2. 注册/登录账号
3. 进入 API Keys 页面创建新 Key
4. 新用户注册赠送免费额度
   
```bash
# 设置环境变量 (添加到 ~/.bashrc 或 ~/.zshrc 永久生效)
export ANTHROPIC_BASE_URL='https://api.siliconflow.cn/'
export ANTHROPIC_API_KEY='sk-你的 API-KEY'
export ANTHROPIC_MODEL='Qwen/Qwen3-Coder-480B-A35B-Instruct'

# 验证配置 输出 2.1.59 (Claude Code)
claude --version

#进入查看模型 输出有 ·Qwen/Qwen3-Coder-480B-A35B-Instruct·即表示配置成功
/model

```
---


**<span style="color:red">确保本地 Claude Code 是可用的，后面直接利用它帮你自动安装</span>**

## 阶段 2: 使用 Claude Code 安装 OpenClaw

### 2.1 安装 OpenClaw

首先到文件夹下`DeployOpenClawOnWSL`进入 Claude Code 提问：
```
参考文档 @openclaw-complete-guide.md @openclaw-feishu-setup-guide.md @openclaw-install-config-guide.md   请帮我安装 OpenClaw v2026.3.2，安装好之后优先保证本地可用，然后再指导我链接飞书机器人，最后指导我怎么设定定时任务
```

或者手动执行[**<span style="color:red">不建议手动执行，利用 claude code 帮你安装</span>**]:

```bash
npm install -g openclaw@2026.3.2
openclaw --version
```

### 2.2 配置 OpenClaw

详细配置步骤参考：[OpenClaw 安装与配置指南](./openclaw-install-config-guide.md)

**快速配置:**
```bash
mkdir -p ~/.openclaw
nano ~/.openclaw/openclaw.json
```

---

## 阶段 3: 飞书机器人集成

### 3.1 安装飞书插件

```bash
openclaw plugins install @openclaw/feishu
```

### 3.2 飞书开放平台配置

详细步骤参考：[飞书配置指南](./openclaw-feishu-setup-guide.md)

**关键步骤:**
1. 创建企业应用
2. 获取 App ID 和 App Secret
3. 配置权限 (批量导入 JSON)
4. 启用机器人
5. 发布应用

### 3.3 长连接建立

**正确顺序:**
1. 配置 App ID/Secret 到 openclaw.json
2. 启动 Gateway: `openclaw daemon start`
3. 确认 WebSocket 已连接：`openclaw logs --follow | grep feishu`
4. 在飞书后台保存事件订阅

### 3.4 配对流程

1. 飞书中给机器人发消息
2. 获取配对码
3. 执行：`openclaw pairing approve feishu <配对码>`

---

## 阶段 4: 创建定时任务

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

### 更多示例参考

查看 [飞书配置指南](./openclaw-feishu-setup-guide.md#7-创建定时任务)

---

## 常见问题速查

| 问题 | 解决方案 |
|------|----------|
| 401 API Key 无效 | 检查 API Key 是否正确，无空格 |
| Gateway 无法启动 | 设置 `"gateway.mode": "local"` |
| 浏览器 401 未授权 | 设置 `"auth.mode": "none"` 或使用无痕模式 |
| 搜索功能不可用 | 配置 Tavily API Key (免费 1000 次/月) |
| 飞书长连接失败 | 确认 App Secret 正确，先启动 Gateway 再保存 |
| 机器人不回复 | 检查配对状态和事件订阅 |

详细解决方案参考：[问题排查指南](./openclaw-install-config-guide.md#7-常见问题与解决方案)

---

## 完整配置文件示例（带详细注释）

```json5
{
  // ============================================
  // 环境变量配置
  // ============================================
  "env": {
    // 硅基流动 API Key - 用于访问 DeepSeek 等 AI 模型
    // 获取地址：https://cloud.siliconflow.cn/
    // 格式：sk-xxxxxxxxxxxxxxxx
    "SILICONFLOW_API_KEY": "sk-你的 APIKey"
  },

  // ============================================
  // 网关配置 - 控制 Web UI 访问
  // ============================================
  "gateway": {
    // 端口号：浏览器访问的端口 (默认 18789)
    "port": 18789,
    // 运行模式：local=本地模式 (必须设置，否则无法启动)
    "mode": "local",
    // 绑定地址：loopback=仅本地访问 (127.0.0.1)
    // 如需局域网访问可改为 "0.0.0.0"
    "bind": "loopback",
    // 认证配置
    "auth": {
      // 认证模式：none=无认证 (开发环境) | token=Token 认证 (生产环境)
      "mode": "none"
      // 如使用 token 认证，需添加："token": "自定义令牌"
    }
  },

  // ============================================
  // Agent 默认配置 - 控制 AI 助手行为
  // ============================================
  "agents": {
    "defaults": {
      // 模型配置
      "model": {
        // 主模型：优先使用的模型
        "primary": "siliconflow/deepseek-ai/DeepSeek-V3",
        // 备用模型：主模型失败时自动切换
        "fallbacks": ["siliconflow/deepseek-ai/DeepSeek-R1"]
      },
      // 模型别名配置 - 便于识别
      "models": {
        "siliconflow/deepseek-ai/DeepSeek-V3": {
          "alias": "DeepSeek-V3"
        },
        "siliconflow/deepseek-ai/DeepSeek-R1": {
          "alias": "DeepSeek-R1"
        }
      },
      // 上下文管理 - 控制会话历史长度
      "contextPruning": {
        // 模式：cache-ttl=基于 TTL 缓存 | aggressive=激进裁剪
        "mode": "cache-ttl",
        // TTL: 会话缓存有效期 (30m=30 分钟，1h=1 小时)
        "ttl": "1h"
      },
      // 会话压缩配置
      "compaction": {
        // 模式：safeguard=保护模式 (保留更多上下文)
        "mode": "safeguard"
      },
      // 心跳检测 - 保持会话活跃
      "heartbeat": {
        // 检测间隔：每 30 分钟检查一次
        "every": "30m"
      }
    }
  },

  // ============================================
  // 模型提供商配置 - 自定义 AI 模型来源
  // ============================================
  "models": {
    // 模式：merge=合并内置配置 | override=完全覆盖
    "mode": "merge",
    // 自定义提供商列表
    "providers": {
      // 硅基流动提供商配置
      "siliconflow": {
        // API 基础地址 - OpenAI 兼容接口
        "baseUrl": "https://api.siliconflow.cn/v1",
        // API Key (也可不填，使用 env.SILICONFLOW_API_KEY)
        "apiKey": "sk-你的 APIKey",
        // API 类型：openai-completions=OpenAI 兼容格式
        "api": "openai-completions",
        // 可用模型列表
        "models": [
          {
            // 模型 ID - 必须与提供商一致
            "id": "deepseek-ai/DeepSeek-V3",
            // 显示名称
            "name": "DeepSeek V3"
          },
          {
            "id": "deepseek-ai/DeepSeek-R1",
            "name": "DeepSeek R1"
          }
        ]
      }
    }
  },

  // ============================================
  // 工具配置 - 扩展 AI 能力
  // ============================================
  "tools": {
    // Web 搜索工具配置
    "web": {
      "search": {
        // Tavily API Key - 用于实时网络搜索
        // 获取地址：https://app.tavily.com/ (免费 1000 次/月)
        // 格式：tvly-xxxxxxxx 或 tvly-dev-xxxxxxxx
        "apiKey": "tvly-你的 APIKey"
      }
    }
  },

  // ============================================
  // 频道配置 - 消息渠道集成
  // ============================================
  "channels": {
    // 飞书频道配置
    "feishu": {
      // 是否启用飞书
      "enabled": true,
      // DM 策略：pairing=需要配对 | open=开放模式
      "dmPolicy": "pairing",
      // 账号配置 (支持多账号)
      "accounts": {
        "default": {
          // 飞书应用 App ID
          "appId": "cli_xxxxxxxxxxxx",
          // 飞书应用 App Secret (仅在初始化时需要)
          "appSecret": "你的 AppSecret"
        }
      }
    }
  },

  // ============================================
  // 插件配置
  // ============================================
  "plugins": {
    // 已启用的插件
    "entries": {
      "feishu": {
        "enabled": true
      }
    },
    // 已安装的插件信息 (自动生成，无需手动编辑)
    "installs": {
      "feishu": {
        "source": "npm",
        "spec": "@openclaw/feishu",
        "version": "2026.3.7"
      }
    }
  }
}
```

---

## 相关文档

- [OpenClaw 安装与配置完整指南](./openclaw-install-config-guide.md) - 详细安装步骤和故障排除
- [飞书频道配置指南](./openclaw-feishu-setup-guide.md) - 飞书集成详细步骤

---

## 附录：常用命令速查

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
# 列出任务
openclaw cron list

# 创建任务
openclaw cron add --name "任务名" --every "1h" --message "消息内容"

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
