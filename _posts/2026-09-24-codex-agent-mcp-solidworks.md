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

Skill可以由团队自行创建，也可以基于已有能力扩展。

### MCP

MCP（Model Context Protocol）用于连接 Agent 和外部工具。

MCP本身不是工程算法，而是工具通信协议。

实际项目中 MCP Server 可以：

- 使用官方SDK开发
- 使用社区实现
- 企业内部自行开发

本项目采用 MCP 作为 Agent 与 CAD 自动化能力之间的桥梁。

## 3. 机加工数字设计 Workflow

完整流程：

1. 需求理解
2. CAD分析
3. 语义提取
4. 参数化设计/修改
5. 工程图生成
6. 验证
7. 交付

## 4. SolidWorks 自动化探索

项目围绕以下能力展开：

- CAD语义解析
- 几何与工程特征识别
- 螺纹和孔特征检查
- 工程图自动生成
- 尺寸验证
- PDF交付流程

## 5. Skill包设计

可复用 Skill 包结构：

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

其中：

- SKILL.md 定义能力边界
- templates 定义工程输入
- prompts 定义 Agent 行为
- workflow 定义执行流程
- mcp 定义工具连接

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

这样可以将 AI 推理、工程流程和 CAD 执行进行解耦。

## 7. 工程化特点

相比简单自动化脚本，本项目强调：

- 流程可复用
- 状态可追踪
- 输入输出标准化
- 人工确认节点明确
- 算法与工具解耦

## 8. 交付物

最终沉淀：

- 机加件数字设计 Workflow
- machining-part-design-workflow Skill
- 工程输入模板
- Prompt模板
- MCP编排说明
- 项目Wiki
- 设计交付结构

## 9. 后续方向

未来可继续扩展：

- 更多CAD平台支持
- 更多工程检查规则
- 更丰富MCP Tool Contract
- 自动化验证体系
