---
title: "Codex / Agent + MCP + SolidWorks：从 CAD 自动检查到工程图生成的工程化实践"
description: "记录一次将机械设计流程、Agent、Skill、MCP 与 SolidWorks 自动化结合的工程实践过程。"
date: 2026-09-24
category: "AI & Engineering"
tags: ["Agent", "MCP", "SolidWorks", "CAD", "Skill", "Automation"]
draft: false
---

# Codex / Agent + MCP + SolidWorks

## 从 CAD 自动检查到工程图生成的工程化实践

## 1. 项目背景

传统机械设计流程中，CAD 检查、工程图制作、尺寸验证和设计评审大量依赖工程师经验。这些经验通常存在于个人习惯、历史文件和项目流程中，难以形成可复用能力。

本项目探索将机械工程知识转化为 Agent 可执行的数字化工作流。

整体目标：

```
需求输入
 ↓
语义理解
 ↓
CAD 分析
 ↓
设计检查
 ↓
工程图生成
 ↓
验证
 ↓
交付
```

## 2. 系统架构

整体架构：

```
User
 ↓
Agent
 ↓
Skill Workflow
 ↓
MCP
 ↓
SolidWorks API
 ↓
CAD / Drawing Output
```

### Agent

负责理解任务、规划步骤、调用能力并汇总结果。

### Skill

负责封装工程知识：

- 工作流程
- 输入输出规范
- 提示词模板
- 异常处理规则

### MCP

负责 Agent 与外部工具之间的连接。

MCP 不是工程算法，而是一种工具通信协议。

官方协议：

https://modelcontextprotocol.io/

MCP Server 可以由官方 SDK、社区实现或企业内部开发。

## 3. Agent 如何知道流程

Agent 不通过 MCP 学习流程。

执行关系：

```
用户需求
 ↓
Agent
 ↓
Skill
 ↓
Workflow
 ↓
MCP Tools
 ↓
SolidWorks
```

其中：

- Skill 定义怎么做
- MCP 定义怎么连接工具
- Agent 负责理解任务并协调执行

## 4. Skill 设计

项目沉淀：

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

Skill 不是普通插件，而是工程能力资产。

其他团队复用时，只需要：

1. 导入 Skill
2. 配置工具连接
3. 提供工程输入
4. 执行 Workflow
5. 获取标准化输出

## 5. CAD 语义层

项目没有直接依赖简单几何遍历，而建立语义层：

- SemanticExecutionFacts
- HoleSemantics
- PhysicalOpenings
- FrameEvidence
- DatumEvidence

目标是让系统理解工程含义，而不是只识别几何实体。

## 6. 工程图自动化架构演进

### 第一阶段：Inline 实现

快速验证功能，但存在耦合问题。

### 第二阶段：统一 Router

建立统一入口：

```
ExecuteAnnotationCreation
```

根据 Annotation 类型分发：

```
Pattern
Callout
Overall
```

### 第三阶段：Context 收敛

建立：

- AnnotationExecutionRequest
- ResolvedAnnotationExecutionContext
- OverallExecutionContext

用于解决输入边界和生命周期问题。

## 7. OVERALL 泛化过程

迁移过程中保持：

- 不复制算法
- 不产生第二套实现
- 保留原验证逻辑

目标流程：

```
Request
 ↓
OverallExecutionContext
 ↓
Probe
 ↓
Commit
 ↓
Registration
```

## 8. 验证过程中的问题

### Semantic Frame Evidence

问题：

```
axis extrema unavailable
```

解决方向：适配已有语义数据结构，不重新计算几何结果。

### SolidWorks Native Call

问题：

```
AddHoleCallout2
CALL_ENTER=YES
CALL_RETURN=NO
```

验证过视图、选择状态和 Rebuild 流程后，问题定位到同步 COM 调用生命周期。

## 9. 最终交付物

沉淀内容：

- 《机加件数字设计工作流》
- machining-part-design-workflow Skill
- 工程输入模板
- Prompt 模板
- MCP 架构说明
- 项目 Wiki
- 交付结构

## 10. 总结

本项目探索：

```
工程知识
+
Agent规划
+
Skill流程
+
MCP工具连接
+
CAD自动执行
```

最终目标是将机械工程经验转化为可复用数字资产。
