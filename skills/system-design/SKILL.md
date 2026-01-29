---
name: system-design
description: 系统架构设计技能。使用场景：(1) 架构模式选择 (2) 模块划分 (3) 技术架构设计 (4) 接口设计 (5) ADR编写。输出完整的架构设计文档。
---

# 系统架构设计技能

系统化地进行软件系统架构设计，为开发团队提供清晰的技术蓝图。

## 适用角色

- 系统架构师（主要）
- 技术负责人

## 输入/输出

**输入:** PRD、非功能需求、技术约束、资源约束

**输出:**
- 架构设计文档：`docs/ccteam/02-architecture/architecture-design.md`
- ADR：`docs/ccteam/02-architecture/adr/`

## 设计流程

### 阶段1：需求分析

- 功能需求分析
- 非功能需求分析（性能、可用性、安全）
- 约束条件识别

### 阶段2：架构模式选择

| 模式 | 适用场景 |
|------|----------|
| 单体架构 | 小型项目、MVP |
| 分层架构 | 企业应用 |
| 微服务 | 大型系统、多团队 |
| 事件驱动 | 实时处理、解耦 |
| Serverless | 低流量、事件触发 |

### 阶段3：模块划分

使用DDD方法识别限界上下文，遵循单一职责、高内聚低耦合原则。

### 阶段4：技术架构

分层设计、数据架构、安全架构

### 阶段5：接口设计

RESTful API、gRPC、消息格式规范

详细模板见 [references/architecture-template.md](references/architecture-template.md)
