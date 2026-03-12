# OpenClaw 完整集成指南

> 在 Windows 11 + WSL2 环境下，零成本搭建你的 AI 助手系统

[![OpenClaw Version](https://img.shields.io/badge/OpenClaw-2026.3.2-blue)](https://github.com/openclaw)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![WSL2](https://img.shields.io/badge/WSL2-Windows%2011-lightgrey)](https://learn.microsoft.com/wsl/)

---

## 快速开始

只需 4 个阶段，完成从 0 到 1 的 AI 助手搭建：

| 阶段 | 内容 | 时间 |
|------|------|------|
| 1 | Claude Code + 硅基流动 API | 10 分钟 |
| 2 | OpenClaw 安装与配置 | 15 分钟 |
| 3 | 飞书机器人集成 | 20 分钟 |
| 4 | 定时任务创建 | 5 分钟 |

**立即开始：** [完整集成指南](./docs/openclaw-complete-guide.md)

---

## 文档结构

```
openclaw/
├── README.md                        # 本文件 - 快速入口
├── docs/
│   ├── openclaw-complete-guide.md       # 主入口指南 - 从这里开始
│   ├── openclaw-install-config-guide.md # OpenClaw 详细配置指南
│   ├── openclaw-feishu-setup-guide.md   # 飞书集成详细指南
│   └── openclaw-usage-log.md            # 使用记录 - Token 消耗与建议
```

### 应该先看哪个文档？

1. **新手入门** → [`docs/openclaw-complete-guide.md`](./docs/openclaw-complete-guide.md)
2. **遇到安装问题** → [`docs/openclaw-install-config-guide.md`](./docs/openclaw-install-config-guide.md)
3. **配置飞书机器人** → [`docs/openclaw-feishu-setup-guide.md`](./docs/openclaw-feishu-setup-guide.md)
4. **使用记录与 Token 消耗** → [`docs/openclaw-usage-log.md`](./docs/openclaw-usage-log.md)

---

## 技术栈

- **运行环境**: Windows 11 + WSL2
- **核心工具**: Claude Code, OpenClaw v2026.3.2
- **AI 模型**: SiliconFlow (DeepSeek-V3/R1) - 免费额度
- **消息渠道**: 飞书机器人
- **搜索增强**: Tavily API (免费 1000 次/月)

---

## 核心特性

- **零成本体验**: 利用硅基流动免费 API 额度
- **多模型支持**: DeepSeek-V3/R1 无缝切换
- **飞书集成**: 企业级消息推送和交互
- **定时任务**: Cron 表达式支持，自动化 AI 任务
- **Web 搜索**: 实时网络信息查询能力

---

## 快速命令参考

### 服务管理

```bash
openclaw daemon start          # 启动 Gateway
openclaw daemon stop           # 停止服务
openclaw daemon status         # 查看状态
openclaw health                # 健康检查
```

### 定时任务

```bash
# 创建每分钟时间汇报任务
openclaw cron add \
  --name "时间汇报" \
  --every "1m" \
  --message "请告诉我当前的时间" \
  --channel "feishu" \
  --to "ou_xxxxx" \
  --announce
```

### 飞书配对

```bash
openclaw pairing list feishu           # 查看配对请求
openclaw pairing approve feishu XXXX   # 批准配对
```

---

## 配置文件位置

| 文件 | 路径 |
|------|------|
| 主配置 | `~/.openclaw/openclaw.json` |
| 日志 | `/tmp/openclaw/` |
| 会话历史 | `~/.openclaw/agents/<id>/sessions/` |

---

## 常见问题

| 问题 | 解决方案 |
|------|----------|
| API Key 无效 (401) | 检查 API Key 格式，确认无空格 |
| Gateway 无法启动 | 设置 `"gateway.mode": "local"` |
| 浏览器未授权 | 设置 `"auth.mode": "none"` 或使用无痕模式 |
| 飞书长连接失败 | 先启动 Gateway，再保存事件订阅 |
| 搜索不可用 | 配置 Tavily API Key |

详细解决方案见：[常见问题排查](./docs/openclaw-install-config-guide.md#7-常见问题与解决方案)

---

## 贡献

欢迎提交 Issue 和 Pull Request 来改进文档。

---

## 许可证

MIT License

---

## 相关资源

- [OpenClaw 官方仓库](https://github.com/openclaw/openclaw)
- [硅基流动控制台](https://cloud.siliconflow.cn/)
- [飞书开放平台](https://open.feishu.cn/)
- [Tavily API](https://app.tavily.com/)

---

**文档更新时间**: 2026-03-12
