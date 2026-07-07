---
title: 什么是 OpenClaw：本地优先的 AI 助手框架入门
tags: [OpenClaw, AI, Tools]
categories: [AI编程与应用]
toc: true

---

OpenClaw 是一个开源的 AI 助手框架，让你能够在本地运行 AI 助手，并通过 Skills（技能）系统扩展其能力。本文将介绍 OpenClaw 的基本概念、安装配置以及使用方法。



<!-- more -->



## **OpenClaw 简介**

### **什么是 OpenClaw**

**OpenClaw 是一个开源的、可扩展的 AI 助手平台**，旨在让用户能够在本地环境中运行和定制自己的 AI 助手。与云端 AI 服务不同，OpenClaw 强调数据隐私和本地控制，所有数据都存储在本地，用户可以完全掌控自己的 AI 助手。

OpenClaw 的核心理念是**模块化**和**可扩展性**。通过 Skills（技能）系统，用户可以为 AI 助手添加各种功能，如天气查询、GitHub 操作、语音合成等。这种设计使得 OpenClaw 不仅是一个聊天工具，更是一个可以深度定制的智能助手平台。

------

### **OpenClaw 的核心特性**

首先，**本地优先**。OpenClaw 完全在本地运行，不需要将数据发送到云端，保证了数据的隐私和安全。所有聊天记录、配置文件都存储在本地目录中。

其次，**Skills（技能）系统**。这是 OpenClaw 最强大的特性之一。Skills 是模块化的功能扩展，用户可以通过编写 SKILL.md 文件来定义新的技能，或者从社区下载现成的技能。

第三，**多平台支持**。OpenClaw 支持多种消息平台，包括 Web 聊天、Discord、Telegram、Slack、WhatsApp 等，用户可以选择自己喜欢的方式与 AI 助手交互。

此外，**记忆系统**。OpenClaw 内置了记忆系统，包括短期记忆（会话历史）和长期记忆（MEMORY.md），使得 AI 助手能够记住用户的偏好和重要信息。


-----------

## **安装与配置**

### **环境要求**

**OpenClaw 基于 Node.js 构建**，因此需要以下环境：

- **Node.js**：版本 18.0 或更高
- **npm**：Node.js 自带的包管理器
- **Git**：用于克隆仓库和更新

对于 Windows 用户，还需要确保 PowerShell 执行策略允许运行脚本。

------

### **安装步骤**

**1. 全局安装 OpenClaw**

```bash
npm install -g openclaw
```

**2. 验证安装**

```bash
openclaw --version
```

**3. 初始化工作空间**

OpenClaw 会在用户目录下创建 `.openclaw/workspace` 作为默认工作空间。

------

### **配置说明**

OpenClaw 的配置文件位于 `~/.openclaw/config.json`，主要配置项包括：

| **配置项**               | **说明**           | **默认值**         |
| ------------------------ | ------------------ | ------------------ |
| `model`                  | 默认使用的 AI 模型 | `kimi-coding/k2p5` |
| `gateway.port`           | 本地网关服务端口   | `8080`             |
| `agents.defaults.skills` | 默认加载的技能列表 | `[]`               |

**修改配置示例**：

```bash
# 设置默认模型
openclaw configure model kimi-coding/k2p5

# 查看当前配置
openclaw status
```

------

### **启动网关**

OpenClaw 使用网关模式运行，需要先启动网关服务：

```bash
openclaw gateway start
```

启动后，可以通过 `http://localhost:8080` 访问 Web 聊天界面。

-----------

## **核心概念**

### **Skills（技能）**

**Skills 是 OpenClaw 的功能扩展单元**，每个 Skill 都是一个独立的模块，包含特定的功能和知识。

**Skill 的结构**：

```
skill-name/
├── SKILL.md          # 技能定义文件（必需）
├── scripts/          # 可执行脚本（可选）
├── references/       # 参考文档（可选）
└── assets/           # 资源文件（可选）
```

**SKILL.md 示例**：

```markdown
---
name: weather
description: Get current weather and forecasts via wttr.in or Open-Meteo.
              Use when user asks about weather, temperature, or forecasts.
---

# Weather Skill

## Usage

Simply ask about weather in any location:
- "What's the weather in Beijing?"
- "Will it rain tomorrow?"
```

------

### **Sessions（会话）**

**Session 是用户与 AI 助手的对话上下文**。每个 Session 都有自己的历史记录，AI 助手可以基于上下文理解用户的意图。

**Session 的类型**：

1. **主会话（Main Session）**：直接与用户的对话，可以访问 MEMORY.md 等长期记忆
2. **子会话（Sub-session）**：用于执行特定任务的隔离会话，通过 `sessions_spawn` 创建

**管理会话**：

```bash
# 列出所有会话
openclaw sessions list

# 查看会话历史
openclaw sessions history <session-key>
```

------

### **Memory（记忆）**

**OpenClaw 的记忆系统分为两层**：

**1. 短期记忆（Session Memory）**

- 存储在当前会话的上下文中
- 随会话结束而清除
- 用于维护对话的连贯性

**2. 长期记忆（Long-term Memory）**

- 存储在 `MEMORY.md` 文件中
- 跨会话持久化
- 包含用户偏好、重要决策、待办事项等

**记忆文件结构**：

```
~/.openclaw/workspace/
├── MEMORY.md              # 长期记忆
├── memory/
│   └── 2026-02-27.md     # 每日记录
└── SOUL.md               # AI 角色定义
```

------

### **Agent（代理）**

**Agent 是 OpenClaw 中执行任务的实体**。每个 Agent 都有自己的配置，包括使用的模型、加载的 Skills 等。

**Agent 的配置项**：

| **配置项** | **说明**                        |
| ---------- | ------------------------------- |
| `model`    | 使用的 AI 模型                  |
| `thinking` | 思考级别（off/low/medium/high） |
| `skills`   | 加载的技能列表                  |
| `soul`     | AI 角色定义文件                 |

-----------

## **常用功能**

### **内置工具**

OpenClaw 提供了丰富的内置工具，可以通过 function calling 使用：

| **工具**         | **用途**     |
| ---------------- | ------------ |
| `read`           | 读取文件内容 |
| `write`          | 写入文件     |
| `edit`           | 编辑文件     |
| `exec`           | 执行命令     |
| `web_search`     | 网页搜索     |
| `web_fetch`      | 获取网页内容 |
| `browser`        | 浏览器自动化 |
| `sessions_spawn` | 创建子会话   |

------

### **Skills 管理**

**查看已安装 Skills**：

```bash
openclaw skills list
```

**安装新 Skill**：

将 Skill 文件夹复制到 OpenClaw 的 skills 目录：

```bash
cp -r my-skill ~/.openclaw/skills/
```

**创建自定义 Skill**：

参考 `skill-creator` Skill 的指南，创建自己的 SKILL.md 文件。

------

### **消息平台集成**

OpenClaw 支持多种消息平台，配置方式：

**Discord**：

```bash
openclaw configure discord.token <your-bot-token>
```

**Telegram**：

```bash
openclaw configure telegram.bot-token <your-bot-token>
```

**Slack**：

```bash
openclaw configure slack.bot-token <your-bot-token>
```

-----------

## **使用示例**

### **示例 1：查询天气**

当安装了 `weather` Skill 后，可以直接询问：

```
用户：北京今天天气怎么样？
AI：北京今天晴天，温度 15-22°C，微风。
```

------

### **示例 2：文件操作**

```
用户：在我的空间里创建一个 hello.txt，内容是 Hello World
AI：[执行 write 工具创建文件]
已完成！文件已保存到 D:\openclaw\hello.txt
```

------

### **示例 3：网页搜索**

```
用户：搜索 OpenClaw 的最新版本
AI：[执行 web_search 工具]
OpenClaw 最新版本是 v2026.2.26，发布于...
```

------

### **示例 4：创建子任务**

```
用户：分析这个项目的代码质量
AI：[创建子会话进行代码分析]
已创建分析任务，稍后将返回结果...
```

-----------

## **最佳实践**

### **编写有效的 Skills**

1. **描述要清晰**：SKILL.md 的 description 要准确描述 Skill 的用途和触发条件
2. **示例要具体**：提供具体的使用示例，帮助 AI 理解何时使用该 Skill
3. **保持简洁**：SKILL.md 不宜过长，参考文档可以放在 references/ 目录
4. **测试要充分**：确保 Skill 在各种场景下都能正常工作

------

### **管理记忆**

1. **定期整理**：定期回顾 memory/ 下的每日记录，将重要信息整理到 MEMORY.md
2. **分类存储**：按主题分类存储记忆，便于查找和更新
3. **及时更新**：当用户偏好或重要决策发生变化时，及时更新 MEMORY.md

------

### **安全注意事项**

1. **谨慎执行命令**：`exec` 工具可以执行任意命令，要确保 AI 不会执行危险操作
2. **保护敏感信息**：不要在 Skill 中硬编码 API Key 等敏感信息
3. **定期备份**：定期备份 workspace 目录，防止数据丢失

-----------

## **总结**

**OpenClaw 是一个强大且灵活的本地 AI 助手平台**，通过 Skills 系统实现了高度的可扩展性。它的本地优先设计保证了数据隐私，多平台支持提供了便捷的使用体验。

对于开发者来说，OpenClaw 不仅是一个工具，更是一个可以深度定制的智能助手框架。通过编写自定义 Skills，可以将 OpenClaw 打造成符合个人需求的专属助手。

随着社区的发展，越来越多的 Skills 将被开发出来，OpenClaw 的能力也将不断扩展。

------

**参考链接**：

- OpenClaw 官方文档：https://docs.openclaw.ai
- OpenClaw GitHub：https://github.com/openclaw/openclaw
- Skills 市场：https://clawhub.com

---
