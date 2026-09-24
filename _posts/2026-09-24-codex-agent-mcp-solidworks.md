---
title: "Codex / Agent + MCP + SolidWorks：从CAD自动检查到工程图生成的工程化实践"
date: 2026-09-24
categories:
  - AI
  - CAD
  - Engineering
  - MCP
---

# Codex / Agent + MCP + SolidWorks

## 从CAD自动检查到工程图生成的工程化实践

## 1. 项目背景

传统机械设计过程中，CAD建模、工程图制作、尺寸检查和设计评审大量依赖工程师经验。

这些经验通常存在于：

- 工程师脑中的规则
- 项目历史文件
- 分散的检查流程
- 手工评审记录

本项目尝试探索一种新的方式：

将机械工程经验转化为 Agent 可以执行的数字化流程。

目标不是简单自动生成CAD，而是建立完整闭环：

```
需求输入
 ↓
语义理解
 ↓
CAD分析
 ↓
设计检查
 ↓
工程图生成
 ↓
验证
 ↓
交付
```

---

# 2. 整体架构

系统采用：

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

三个核心层负责不同职责：

## Agent

负责：

- 理解用户目标
- 规划任务步骤
- 调用能力
- 汇总结果

## Skill

负责：

- 工程知识
- 流程定义
- 输入输出规范
- 异常处理规则

## MCP

负责：

- Agent与外部工具连接
- 工具调用协议
- 数据传递

---

# 3. Agent、Skill、MCP如何协作

Agent并不会因为连接了MCP就自动知道机械设计流程。

流程来源：

```
用户需求
 ↓
Agent
 ↓ 读取
Skill
 ↓ 执行
Workflow
 ↓ 调用
MCP Tools
 ↓
SolidWorks
```

其中：

- Skill定义“怎么做”
- MCP定义“怎么连接工具”
- Agent负责“理解任务并协调执行”

---

# 4. Skill设计

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

Skill不是普通插件，而是一种工程能力封装。

包含：

- 工作流程
- 工程模板
- Agent提示词
- 工具接口说明
- 示例案例

Skill可以：

- 团队自行创建
- 基于项目经验迭代
- 作为企业工程资产复用

---

# 5. MCP设计

MCP（Model Context Protocol）是一种工具连接协议。

官方协议：

https://modelcontextprotocol.io/

MCP Server可以：

- 使用官方SDK开发
- 使用社区实现
- 企业内部自研

本项目中MCP作为：

```
Agent
 ↓
MCP Client
 ↓
MCP Server
 ↓
SolidWorks Runtime
```

之间的桥梁。

---

# 6. CAD语义层设计

项目没有直接依赖简单几何遍历，而是建立语义层。

核心思想：

让系统理解工程含义，而不是只识别几何实体。

包括：

- SemanticExecutionFacts
- HoleSemantics
- PhysicalOpenings
- FrameEvidence
- DatumEvidence

用于描述：

- 孔特征
- 基准关系
- 几何范围
- 工程意图

---

# 7. 工程图自动生成架构演进

## 第一阶段：Inline实现

快速验证功能。

问题：

- 流程耦合
- 难以复用
- 难以扩展

## 第二阶段：统一路由

引入统一入口：

```
ExecuteAnnotationCreation
```

根据类型分发：

```
Annotation
 |
 ├── Pattern
 ├── Callout
 └── Overall
```

## 第三阶段：Context收敛

建立：

- AnnotationExecutionRequest
- ResolvedAnnotationExecutionContext
- OverallExecutionContext

解决：

- 输入边界
- 状态传递
- 生命周期管理

---

# 8. OVERALL泛化过程

OVERALL功能迁移过程中重点保证：

- 不复制算法
- 不产生第二套实现
- 保留原有验证逻辑

最终目标：

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

---

# 9. 验证过程中的工程问题

## Semantic Frame Evidence问题

现象：

```
axis extrema unavailable
```

原因：

旧解析逻辑未适配新的语义数据结构。

处理：

适配已有 semantic artifact 数据，不重新计算几何结果。

---

## SolidWorks Native Call问题

现象：

```
AddHoleCallout2
CALL_ENTER=YES
CALL_RETURN=NO
```

验证过：

- View激活
- Selection重新选择
- Rebuild

问题定位到同步COM调用生命周期。

记录：

- 不增加fallback算法
- 不修改核心流程
- 保留诊断信息

---

# 10. 最终交付物

项目沉淀：

## 工作流

《机加件数字设计工作流》

包含：

- 输入
- 步骤
- 输出
- 异常处理
- 人工确认点

## Skill包

`machining-part-design-workflow`

## 模板

包括：

- 机加件需求模板
- CAD对比模板
- 工程图模板
- 评审模板

## MCP说明

包括：

- 架构
- Tool Contract
- 编排入口

---

# 11. 总结

本项目探索了一种机械设计自动化模式：

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

最终目标是将工程经验转化为可复用数字资产，使机械设计流程具备更高的自动化、可追踪和可扩展能力。
