# OpenClaw 学习教程

> 🦞 一周掌握 OpenClaw，打造跨设备 AI 助手工作流

[![OpenClaw](https://img.shields.io/badge/OpenClaw-AI%20Gateway-orange)](https://openclaw.ai)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## 📖 简介

本教程帮助你在 **1 周内** 掌握 OpenClaw，从安装配置到自动化工作流，实现随时随地和 AI 助手对话。

## 🎯 学习目标

完成本教程后，你将能够：

- ✅ 在多个聊天渠道（Telegram/飞书/Discord）连接 AI 助手
- ✅ 配置 Agent 工作空间和人格
- ✅ 安装和管理 Skills/Plugins
- ✅ 配置 Cron 定时任务
- ✅ 使用 Heartbeat 实现主动工作

## 📚 课程大纲

| Day | 主题 | 内容 |
|-----|------|------|
| 1 | [环境搭建](docs/day1-setup.md) | 安装 OpenClaw，连接第一个渠道 |
| 2 | [多渠道配置](docs/day2-channels.md) | Telegram/飞书/Discord 配置 |
| 3 | [Agent 工作空间](docs/day3-workspace.md) | SOUL.md/USER.md/记忆系统 |
| 4 | [Skills 系统](docs/day4-skills.md) | 安装和创建 Skills |
| 5 | [Cron 定时任务](docs/day5-cron.md) | 定时提醒和自动化任务 |
| 6 | [Heartbeat](docs/day6-heartbeat.md) | 主动检查和批量任务 |
| 7 | [综合项目](docs/day7-project.md) | 打造个人 AI 助手 |

## 🚀 快速开始

### 1. 安装

```bash
# 确保 Node 22+
node -v

# 安装 OpenClaw
npm i -g openclaw

# 首次运行
openclaw
```

### 2. 配置

```bash
# 配置飞书
openclaw configure --section feishu
```

### 3. 启动

```bash
openclaw gateway start
```

### 4. 发消息

在 Telegram 给你的 bot 发消息，收到 AI 回复！

## 📁 项目结构

```
openclaw-tutorial/
├── README.md           # 本文件
├── docs/
│   ├── day1-setup.md
│   ├── day2-channels.md
│   └── ...
├── examples/           # 配置示例
│   ├── openclaw.json
│   ├── workspace/
│   └── skills/
└── projects/           # 项目模板
```

## 🔗 资源链接

- [OpenClaw 官方文档](https://docs.openclaw.ai)
- [ClawHub (Skills 市场)](https://clawhub.com)
- [OpenClaw Discord](https://discord.com/invite/clawd)
- [GitHub](https://github.com/openclaw/openclaw)

## 📅 学习时间线 (2.17 - 3.2)

本教程与 [claude-code-tutorial](https://github.com/linjh1118/claude-code-tutorial) **并行学习**。

### Week 1：基础篇 (2.17 - 2.23)

| 日期 | OpenClaw 任务 | 交付物 |
|-----|---------------|--------|
| 2/17 (一) | Day1: 环境搭建 | 安装成功 + 飞书连接截图 |
| 2/18 (二) | Day2: 多渠道配置 | 第二个渠道连接截图 |
| 2/19 (三) | Day3: Agent 工作空间 | SOUL.md + USER.md |
| 2/20 (四) | Day4: Skills 系统 | 创建 Skill 文件 |
| 2/21 (五) | Day5: Cron 定时任务 | Cron 执行截图 |
| 2/22 (六) | Day6: Heartbeat | HEARTBEAT.md 配置 |
| 2/23 (日) | **Day7: 综合项目** | **🎯 Week1 项目提交** |

### Week 2：进阶篇 (2.24 - 3.2)

| 日期 | OpenClaw 任务 | 交付物 |
|-----|---------------|--------|
| 2/24 (一) | [深入 Skills 定制](docs/day8-advanced-skills.md) | 高级 Skill |
| 2/25 (二) | [多 Agent 配置](docs/day9-multi-agent.md) | Agent 配置文件 |
| 2/26 (三) | Webhook/自动化 | 自动化配置 |
| 2/27 (四) | 高级 Cron | 复杂定时任务 |
| 2/28 (五) | 工作流优化 | 学习笔记 |
| 3/1 (六) | 跨设备协作 | 多设备截图 |
| 3/2 (日) | **最终项目** | **🎯 最终项目 + 总结报告** |

### 🎯 关键里程碑

```
2/17          2/23              3/2
 │             │                 │
 ▼             ▼                 ▼
开始学习     Week1 项目        最终项目
(并行)        提交             + 总结报告
```

## 📝 前置要求

- Node 22+
- API Key（推荐 Anthropic）
- 飞书账号（需要创建企业自建应用）

## 🤝 贡献

欢迎提交 Issue 和 PR！

## 📄 License

MIT License
