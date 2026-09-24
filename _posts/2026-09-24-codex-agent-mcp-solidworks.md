---
title: "Codex / Agent + MCP + SolidWorks：AI驱动的机加工模型检查与工程图生成实践"
date: 2026-09-24
categories:
  - AI
  - CAD
  - Engineering
  - MCP
---

# Codex / Agent + MCP + SolidWorks

## AI驱动的机加工模型检查与辅助工程图生成系统

## 1. 项目背景

传统机械设计流程中，CAD建模、工程图制作、尺寸检查和设计评审依赖大量人工操作。随着 Agent、Skill 和 MCP 技术的发展，可以将机械工程流程进行结构化，让 AI Agent 参与工程任务执行。

本项目探索了一套：

```
Agent
  ↓
Skill Workflow
  ↓
MCP
  ↓
SolidWorks API
  ↓
Semantic Analysis
  ↓
Drawing Generation
```

的数字化设计流程。

## 2. Agent、Skill 与 MCP 的关系

### Agent

Agent负责理解用户任务、规划步骤、调用能力并汇总结果。

Agent本身不保存所有工程流程，而是通过 Skill 获得领域执行规则，通过 MCP 调用外部工具。

### Skill

Skill是工程领域知识和流程定义，不是普通插件。

本项目中的：

`machining-part-design-workflow`

定义：

- 输入要求
- 工作阶段
- 输出格式
- 异常处理
- 人工确认点

Skill可以由团队自行创建，也可以基于项目经验持续迭代。

Skill通常包含：

```
SKILL.md
templates/
prompts/
workflow/
examples/
docs/
```

其中：

- SKILL.md 描述能力边界和执行规则
- templates 提供标准工程输入
- prompts 提供 Agent 行为模板
- examples 提供参考案例
- docs 提供部署和交接说明

### MCP

MCP（Model Context Protocol）用于连接 Agent 和外部工具。

MCP本身不是工程算法，而是工具通信协议。

官方协议地址：

https://modelcontextprotocol.io/

实际项目中 MCP Server 可以：

- 使用官方 SDK 开发
- 使用社区实现
- 企业内部自行开发

本项目采用 MCP 作为 Agent 与 CAD 自动化能力之间的桥梁。

## 3. Agent 如何知道执行流程？

Agent并不是通过 MCP 获得流程。

三者关系如下：

```
用户需求
  ↓
Agent
  ↓ 读取
Skill
  ↓ 定义
Workflow
  ↓ 调用
MCP Tools
  ↓
工程软件
```

例如机加工件设计任务：

1. Agent读取任务目标
2. Skill确定需要执行的工程阶段
3. Workflow决定先需求分析，再CAD检查，再生成图纸
4. MCP负责连接SolidWorks等工具
5. Agent汇总结果并生成报告

因此：

- Skill负责“怎么做”
- MCP负责“怎么连接工具”
- Agent负责“理解任务并协调执行”

## 4. 机加工数字设计 Workflow

完整流程：

1. 需求理解
2. CAD分析
3. 语义提取
4. 参数化设计/修改
5. 工程图生成
6. 验证
7. 交付

每个阶段均定义：

- 输入
- 执行动作
- 输出
- 异常处理
- 人工确认点

## 5. SolidWorks 自动化探索

项目围绕以下能力展开：

- CAD语义解析
- 几何与工程特征识别
- 螺纹和孔特征检查
- 工程图自动生成
- 尺寸验证
- PDF交付流程

## 6. MCP架构

系统关系：

```
User
 ↓
Agent
 ↓
Skill
 ↓
MCP Client
 ↓
MCP Server
 ↓
SolidWorks Runtime
```

这样可以将：

- AI推理
- 工程流程
- CAD执行

进行解耦。

## 7. 可复用 Skill 包设计

项目交付的 Skill 包：

```
machining-part-design-workflow

├── SKILL.md
├── workflow
├── templates
├── prompts
├── examples
├── docs
└── mcp
```

其他工程团队复用时，只需要：

1. 导入 Skill
2. 配置 MCP 工具连接
3. 提供工程输入模板
4. 执行 Workflow
5. 获取标准化输出

## 8. 工程化特点

相比简单自动化脚本，本项目强调：

- 流程可复用
- 状态可追踪
- 输入输出标准化
- 人工确认节点明确
- 算法与工具解耦
- 工程知识资产化

## 9. 交付物

最终沉淀：

- 机加件数字设计 Workflow
- machining-part-design-workflow Skill
- 工程输入模板
- Prompt模板
- MCP编排说明
- 项目Wiki
- 设计交付结构

## 10. 后续方向

未来可继续扩展：

- 更多CAD平台支持
- 更多工程检查规则
- 更丰富MCP Tool Contract
- 自动化验证体系
- 企业级工程知识库
