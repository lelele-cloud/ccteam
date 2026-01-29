# CCTeam 全面优化实施计划

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**目标：** 将 CCTeam 从"设计完成"状态升级为"可执行"状态，实现真正可运行的互联网大厂开发团队模拟。

**架构：** 分四个阶段实施：(1) 执行引擎 - 实现状态管理和流程控制；(2) 智能体增强 - 优化协作机制和行为指导；(3) 技能体系 - 创建实际可用的技能定义；(4) 文档完善 - 补充缺失的模板和规范。

**优先级：** P0 > P1 > P2 > P3

---

## 第一阶段：执行引擎实现 (P0)

### Task 1: 创建项目状态Schema定义

**目标：** 定义完整的 project-status.json 结构，支持状态追踪。

**Files:**
- Create: `schemas/project-status.schema.json`

**Step 1: 创建 schemas 目录并编写状态Schema**

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "CCTeam Project Status",
  "type": "object",
  "required": ["projectInfo", "currentStage", "stages", "team"],
  "properties": {
    "projectInfo": {
      "type": "object",
      "required": ["name", "description", "type", "techStack", "createdAt"],
      "properties": {
        "name": { "type": "string" },
        "description": { "type": "string" },
        "type": { "enum": ["web-fullstack", "web-frontend", "web-backend", "mobile-rn", "mobile-flutter", "miniprogram", "cross-platform", "api-service", "cli-tool"] },
        "techStack": {
          "type": "object",
          "properties": {
            "frontend": { "type": "string" },
            "backend": { "type": "string" },
            "database": { "type": "string" },
            "deployment": { "type": "string" }
          }
        },
        "createdAt": { "type": "string", "format": "date-time" },
        "updatedAt": { "type": "string", "format": "date-time" }
      }
    },
    "currentStage": {
      "type": "object",
      "required": ["id", "name", "status"],
      "properties": {
        "id": { "type": "integer", "minimum": 1, "maximum": 10 },
        "name": { "type": "string" },
        "status": { "enum": ["pending", "in-progress", "review", "completed", "rejected", "skipped"] },
        "startedAt": { "type": "string", "format": "date-time" },
        "assignedAgent": { "type": "string" }
      }
    },
    "stages": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["id", "name", "status"],
        "properties": {
          "id": { "type": "integer" },
          "name": { "type": "string" },
          "displayName": { "type": "string" },
          "status": { "enum": ["pending", "in-progress", "review", "completed", "rejected", "skipped"] },
          "assignedAgents": { "type": "array", "items": { "type": "string" } },
          "reviewers": { "type": "array", "items": { "type": "string" } },
          "startedAt": { "type": "string", "format": "date-time" },
          "completedAt": { "type": "string", "format": "date-time" },
          "outputs": { "type": "array", "items": { "type": "string" } },
          "reviewResult": {
            "type": "object",
            "properties": {
              "passed": { "type": "boolean" },
              "reviewedBy": { "type": "array", "items": { "type": "string" } },
              "comments": { "type": "array", "items": { "type": "string" } },
              "reviewedAt": { "type": "string", "format": "date-time" }
            }
          }
        }
      }
    },
    "team": {
      "type": "object",
      "properties": {
        "activeAgents": { "type": "array", "items": { "type": "string" } },
        "permissions": {
          "type": "object",
          "additionalProperties": {
            "type": "object",
            "properties": {
              "canCreate": { "type": "array", "items": { "type": "string" } },
              "canReview": { "type": "array", "items": { "type": "string" } },
              "canApprove": { "type": "array", "items": { "type": "string" } }
            }
          }
        }
      }
    },
    "issues": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["id", "title", "status", "stage"],
        "properties": {
          "id": { "type": "string" },
          "title": { "type": "string" },
          "description": { "type": "string" },
          "status": { "enum": ["open", "in-progress", "resolved", "closed"] },
          "stage": { "type": "integer" },
          "createdBy": { "type": "string" },
          "assignedTo": { "type": "string" },
          "createdAt": { "type": "string", "format": "date-time" },
          "resolvedAt": { "type": "string", "format": "date-time" }
        }
      }
    },
    "metrics": {
      "type": "object",
      "properties": {
        "totalStages": { "type": "integer" },
        "completedStages": { "type": "integer" },
        "totalIssues": { "type": "integer" },
        "resolvedIssues": { "type": "integer" },
        "reviewPassRate": { "type": "number" }
      }
    }
  }
}
```

**Step 2: Commit**

```bash
git add schemas/project-status.schema.json
git commit -m "feat: add project status JSON schema definition"
```

---

### Task 2: 创建流程引擎核心技能

**目标：** 实现流程控制的核心 skill，支持阶段管理和状态流转。

**Files:**
- Create: `skills/workflow/engine.md`
- Create: `skills/workflow/stage-control.md`
- Create: `skills/workflow/review-gate.md`

**Step 1: 创建 skills/workflow 目录结构**

**Step 2: 创建流程引擎主技能 `skills/workflow/engine.md`**

```markdown
# 流程引擎 (Workflow Engine)

## 概述

流程引擎是 CCTeam 的核心控制中心，负责管理项目状态、阶段流转和评审关卡。

## 触发条件

当以下情况发生时自动激活：
- `/ccteam-new` 创建新项目
- `/ccteam-resume` 恢复项目
- `/ccteam-stage` 手动控制阶段
- 评审完成需要流转

## 核心操作

### 1. 初始化项目状态

当创建新项目时，生成完整的 `project-status.json`：

```json
{
  "projectInfo": {
    "name": "{{project_name}}",
    "description": "{{description}}",
    "type": "{{project_type}}",
    "techStack": {
      "frontend": "{{frontend_stack}}",
      "backend": "{{backend_stack}}",
      "database": "{{database}}",
      "deployment": "{{deployment}}"
    },
    "createdAt": "{{timestamp}}",
    "updatedAt": "{{timestamp}}"
  },
  "currentStage": {
    "id": 1,
    "name": "requirements",
    "status": "pending",
    "assignedAgent": "product-manager"
  },
  "stages": [
    {"id": 1, "name": "requirements", "displayName": "需求分析", "status": "pending", "assignedAgents": ["product-manager"], "reviewers": ["tech-lead"]},
    {"id": 2, "name": "tech-review", "displayName": "技术评审", "status": "pending", "assignedAgents": ["tech-lead", "architect"], "reviewers": ["all"]},
    {"id": 3, "name": "ui-ux-design", "displayName": "UI/UX设计", "status": "pending", "assignedAgents": ["ui-designer", "ux-designer"], "reviewers": ["product-manager", "frontend-engineer"]},
    {"id": 4, "name": "database-design", "displayName": "数据库设计", "status": "pending", "assignedAgents": ["dba", "backend-engineer"], "reviewers": ["architect"]},
    {"id": 5, "name": "api-design", "displayName": "API设计", "status": "pending", "assignedAgents": ["api-engineer"], "reviewers": ["frontend-engineer", "backend-engineer"]},
    {"id": 6, "name": "frontend-dev", "displayName": "前端开发", "status": "pending", "assignedAgents": ["frontend-engineer"], "reviewers": ["code-reviewer", "tech-lead"], "parallelWith": 7},
    {"id": 7, "name": "backend-dev", "displayName": "后端开发", "status": "pending", "assignedAgents": ["backend-engineer"], "reviewers": ["code-reviewer", "tech-lead", "security-engineer"], "parallelWith": 6},
    {"id": 8, "name": "testing", "displayName": "测试", "status": "pending", "assignedAgents": ["qa-engineer"], "reviewers": ["tech-lead", "product-manager"], "waitFor": [6, 7]},
    {"id": 9, "name": "deployment", "displayName": "部署", "status": "pending", "assignedAgents": ["devops-engineer"], "reviewers": ["tech-lead", "security-engineer"]},
    {"id": 10, "name": "documentation", "displayName": "文档归档", "status": "pending", "assignedAgents": ["doc-engineer"], "reviewers": ["product-manager", "tech-lead"]}
  ],
  "team": {
    "activeAgents": [],
    "permissions": {}
  },
  "issues": [],
  "metrics": {
    "totalStages": 10,
    "completedStages": 0,
    "totalIssues": 0,
    "resolvedIssues": 0,
    "reviewPassRate": 0
  }
}
```

### 2. 启动阶段

```
触发: 当前阶段状态为 pending 且前置阶段已完成
操作:
1. 将阶段状态更新为 in-progress
2. 记录 startedAt 时间戳
3. 加载对应的智能体（根据 assignedAgents）
4. 通知智能体开始工作
5. 更新 project-status.json
```

### 3. 完成阶段

```
触发: 智能体宣告工作完成
操作:
1. 验证产出物是否存在
2. 将阶段状态更新为 review
3. 通知评审方开始评审
4. 更新 project-status.json
```

### 4. 评审通过

```
触发: 评审方通过评审
操作:
1. 记录评审结果（reviewResult.passed = true）
2. 将阶段状态更新为 completed
3. 记录 completedAt 时间戳
4. 检查下一阶段的前置条件
5. 如果满足，自动启动下一阶段
6. 更新 metrics
7. 更新 project-status.json
```

### 5. 评审不通过

```
触发: 评审方拒绝评审
操作:
1. 记录评审结果（reviewResult.passed = false）
2. 将阶段状态更新为 rejected
3. 创建 Issue 记录问题
4. 通知原负责智能体
5. 将阶段状态改回 in-progress
6. 更新 project-status.json
```

### 6. 并行阶段处理

```
触发: 阶段5（API设计）完成
操作:
1. 同时启动阶段6（前端开发）和阶段7（后端开发）
2. 分别跟踪两个阶段的状态
3. 当两个阶段都完成后，才能启动阶段8（测试）
```

## 状态文件位置

`docs/ccteam/project-status.json`

## 错误处理

- 如果状态文件损坏，尝试从最近的 git 提交恢复
- 如果找不到状态文件，提示用户运行 `/ccteam-new` 或 `/ccteam-resume`
- 如果阶段流转出错，记录错误并暂停流程

## 日志记录

所有状态变更记录到 `docs/ccteam/workflow.log`：

```
[2024-01-29 10:00:00] STAGE_START: requirements (product-manager)
[2024-01-29 11:30:00] STAGE_COMPLETE: requirements -> review
[2024-01-29 12:00:00] REVIEW_PASS: requirements (tech-lead)
[2024-01-29 12:00:01] STAGE_START: tech-review (tech-lead, architect)
```
```

**Step 3: 创建阶段控制技能 `skills/workflow/stage-control.md`**

```markdown
# 阶段控制 (Stage Control)

## 概述

提供手动控制阶段的能力，用于特殊情况的干预。

## 可用命令

### 启动阶段

```
/ccteam-stage start <stage-name>
```

手动启动指定阶段，忽略前置条件检查。

**参数：**
- `stage-name`: 阶段名称（requirements, tech-review, ui-ux-design, database-design, api-design, frontend-dev, backend-dev, testing, deployment, documentation）

**示例：**
```
/ccteam-stage start api-design
```

### 完成阶段

```
/ccteam-stage complete [--skip-review]
```

标记当前阶段为完成，进入评审流程。

**选项：**
- `--skip-review`: 跳过评审，直接标记为完成（需要技术负责人确认）

### 跳过阶段

```
/ccteam-stage skip <stage-name> --reason "<reason>"
```

跳过指定阶段（仅限可跳过的阶段）。

**可跳过的阶段：**
- ui-ux-design（纯后端项目）
- database-design（无数据库项目）
- deployment（本地开发）

**示例：**
```
/ccteam-stage skip ui-ux-design --reason "纯API服务，无前端界面"
```

### 回退阶段

```
/ccteam-stage rollback [--to <stage-name>]
```

回退到上一个阶段或指定阶段。

**示例：**
```
/ccteam-stage rollback --to tech-review
```

### 查看阶段状态

```
/ccteam-stage status
```

显示所有阶段的当前状态。

## 权限控制

| 命令 | 允许的角色 |
|------|-----------|
| start | tech-lead, project-manager |
| complete | 当前阶段负责人 |
| skip | tech-lead（需说明原因） |
| rollback | tech-lead, architect |
| status | 所有角色 |

## 状态变更通知

每次状态变更后，通知以下角色：
- 项目经理（project-manager）
- 技术负责人（tech-lead）
- 受影响的阶段负责人
```

**Step 4: 创建评审关卡技能 `skills/workflow/review-gate.md`**

```markdown
# 评审关卡 (Review Gate)

## 概述

评审关卡确保每个阶段的产出物质量，是流程控制的关键节点。

## 评审规则

### 评审启动条件

1. 阶段负责人宣告完成
2. 所有必需的产出物已生成
3. 产出物通过基本格式检查

### 评审通过条件

| 评审类型 | 通过条件 |
|----------|----------|
| 单人评审 | 评审人通过即可 |
| 多人评审 | 所有评审人中 ≥2/3 通过 |
| 全员评审 | 所有核心角色中 ≥2/3 通过 |

**核心角色定义：**
- product-manager
- tech-lead
- architect
- frontend-engineer
- backend-engineer

### 评审超时处理

- 评审开始后 24 小时（模拟时间）无响应
- 自动提醒评审人
- 48 小时后自动升级给技术负责人

## 评审流程

### 1. 发起评审

```
触发: 阶段负责人调用完成
操作:
1. 生成评审请求
2. 列出所有产出物
3. 加载评审检查清单
4. 通知评审人
```

### 2. 执行评审

```
评审人操作:
1. 阅读产出物
2. 对照检查清单逐项检查
3. 记录发现的问题
4. 给出评审意见：
   - APPROVE: 通过
   - REQUEST_CHANGES: 需要修改
   - COMMENT: 仅评论
```

### 3. 评审反馈格式

```markdown
## 评审结果

**评审人：** {{reviewer}}
**评审时间：** {{timestamp}}
**结论：** APPROVE / REQUEST_CHANGES

### 检查清单

- [x] 检查项1
- [x] 检查项2
- [ ] 检查项3（问题：xxx）

### 问题列表

| 严重程度 | 问题描述 | 建议 |
|----------|----------|------|
| 🔴 必须修改 | 问题1 | 建议1 |
| 🟡 建议修改 | 问题2 | 建议2 |

### 总体评价

{{评审意见}}
```

### 4. 处理评审结果

**通过：**
```
1. 记录评审通过
2. 更新阶段状态为 completed
3. 触发下一阶段
```

**不通过：**
```
1. 创建 Issue 记录所有问题
2. 将阶段状态改为 rejected
3. 通知负责人修改
4. 负责人修改后重新提交评审
```

## 各阶段评审检查清单

### 需求分析阶段
- [ ] PRD文档完整（背景、目标、功能、验收标准）
- [ ] 用户故事覆盖核心场景
- [ ] 功能优先级已标注
- [ ] 非功能性需求已定义
- [ ] 技术可行性已初步确认

### 技术评审阶段
- [ ] 架构设计合理
- [ ] 技术选型有依据
- [ ] 性能指标已定义
- [ ] 安全需求已考虑
- [ ] 可扩展性已规划
- [ ] 技术风险已识别

### UI/UX设计阶段
- [ ] 交互流程完整
- [ ] 视觉风格统一
- [ ] 响应式设计考虑
- [ ] 无障碍设计考虑
- [ ] 设计规范文档完整

### 数据库设计阶段
- [ ] 数据模型符合业务需求
- [ ] 表结构设计合理
- [ ] 索引策略已规划
- [ ] 数据安全已考虑
- [ ] 扩展性已考虑

### API设计阶段
- [ ] 接口设计符合RESTful规范
- [ ] 数据格式定义清晰
- [ ] 错误码定义完整
- [ ] 认证授权已考虑
- [ ] API文档完整

### 前端/后端开发阶段
- [ ] 代码符合编码规范
- [ ] 单元测试覆盖核心逻辑
- [ ] 无明显安全漏洞
- [ ] 性能指标达标
- [ ] 代码可读性良好

### 测试阶段
- [ ] 测试计划完整
- [ ] 测试用例覆盖率达标
- [ ] 功能测试通过
- [ ] 性能测试达标
- [ ] 安全测试通过

### 部署阶段
- [ ] CI/CD配置正确
- [ ] 部署脚本可执行
- [ ] 回滚方案已准备
- [ ] 监控告警已配置
- [ ] 安全配置已检查

### 文档归档阶段
- [ ] 用户文档完整
- [ ] 开发文档完整
- [ ] API文档与代码一致
- [ ] 部署文档可执行
```

**Step 5: Commit**

```bash
git add skills/workflow/
git commit -m "feat: add workflow engine, stage control, and review gate skills"
```

---

### Task 3: 更新入口技能，集成流程引擎

**目标：** 更新 `/ccteam-new` 等入口技能，使其调用流程引擎。

**Files:**
- Modify: `skills/entry/ccteam-new.md`
- Modify: `skills/entry/ccteam-status.md`
- Modify: `skills/entry/ccteam-resume.md`

**Step 1: 更新 `skills/entry/ccteam-new.md`**

在原有内容基础上，添加流程引擎集成：

```markdown
（在"### 3. 初始化项目"部分后添加）

### 3.5 创建项目状态文件

调用流程引擎初始化状态：

```
使用 skills/workflow/engine.md 中定义的初始化模板
生成 docs/ccteam/project-status.json
记录项目信息、技术栈选择
初始化所有阶段状态为 pending
```

### 3.6 启动第一阶段

```
自动将阶段1（需求分析）状态设为 in-progress
加载产品经理智能体
提示产品经理开始需求分析工作
```

（在最后添加）

## 依赖技能

- `skills/workflow/engine.md` - 流程引擎
- `skills/workflow/stage-control.md` - 阶段控制
- `agents/product-manager/agent.md` - 产品经理智能体
```

**Step 2: 更新 `skills/entry/ccteam-status.md`**

```markdown
# CCTeam - 项目状态

## 概述

查看当前项目的开发状态和进度。

## 使用方法

当用户调用 `/ccteam-status` 时：

### 1. 读取项目状态

读取 `docs/ccteam/project-status.json`

### 2. 生成状态报告

```
📊 项目状态报告
================

项目：{{projectInfo.name}}
描述：{{projectInfo.description}}
类型：{{projectInfo.type}}
创建时间：{{projectInfo.createdAt}}

## 当前阶段

🔵 阶段 {{currentStage.id}}: {{currentStage.name}}
   状态：{{currentStage.status}}
   负责人：{{currentStage.assignedAgent}}
   开始时间：{{currentStage.startedAt}}

## 阶段进度

{{#each stages}}
{{#if (eq status "completed")}}✅{{/if}}
{{#if (eq status "in-progress")}}🔵{{/if}}
{{#if (eq status "review")}}🔍{{/if}}
{{#if (eq status "pending")}}⬜{{/if}}
{{#if (eq status "rejected")}}🔴{{/if}}
{{#if (eq status "skipped")}}⏭️{{/if}}
 阶段{{id}}: {{displayName}} - {{status}}
{{/each}}

## 进度统计

已完成：{{metrics.completedStages}}/{{metrics.totalStages}} ({{percentage}}%)
未解决Issue：{{openIssueCount}}
评审通过率：{{metrics.reviewPassRate}}%

## 待处理事项

{{#each openIssues}}
- [Issue-{{id}}] {{title}} ({{stage}})
{{/each}}
```

### 3. 进度可视化

```
[████████░░░░░░░░░░░░] 40%

需求 ✅ → 技术 ✅ → 设计 ✅ → 数据库 ✅ → API 🔵 → 前端 ⬜ → 后端 ⬜ → 测试 ⬜ → 部署 ⬜ → 文档 ⬜
```

## 快捷操作提示

根据当前状态，提示可用操作：

- 如果当前阶段是 in-progress：
  ```
  💡 当前 {{agent}} 正在处理 {{stage}}
  输入 /ccteam-{{agent}} 与当前负责人对话
  输入 /ccteam-stage complete 标记阶段完成
  ```

- 如果当前阶段是 review：
  ```
  💡 当前 {{stage}} 正在等待评审
  评审人：{{reviewers}}
  输入 /ccteam-review 查看评审详情
  ```

- 如果有未解决的 Issue：
  ```
  ⚠️ 有 {{count}} 个未解决的 Issue 需要处理
  输入 /ccteam-issues 查看详情
  ```
```

**Step 3: 更新 `skills/entry/ccteam-resume.md`**

```markdown
# CCTeam - 恢复项目

## 概述

恢复一个已存在的项目，继续开发流程。

## 使用方法

当用户调用 `/ccteam-resume` 时：

### 1. 检测项目状态

检查 `docs/ccteam/project-status.json` 是否存在：

**如果存在：**
```
✅ 找到项目状态文件

项目：{{projectInfo.name}}
上次更新：{{projectInfo.updatedAt}}
当前阶段：{{currentStage.name}} ({{currentStage.status}})

是否继续此项目？[Y/n]
```

**如果不存在：**
```
❌ 未找到项目状态文件

可能的原因：
1. 这是一个新项目，请使用 /ccteam-new 开始
2. 项目状态文件被删除，需要重新初始化

输入 /ccteam-new 开始新项目
```

### 2. 恢复项目上下文

```
1. 读取 project-status.json
2. 加载当前阶段对应的智能体
3. 读取该阶段已生成的产出物
4. 恢复工作状态
```

### 3. 状态恢复策略

根据 currentStage.status 决定操作：

| 状态 | 操作 |
|------|------|
| pending | 启动阶段，加载负责人智能体 |
| in-progress | 直接加载负责人智能体，继续工作 |
| review | 加载评审人智能体，继续评审 |
| rejected | 加载负责人智能体，处理评审反馈 |
| completed | 检查下一阶段，自动启动 |

### 4. 智能恢复提示

```
📂 项目已恢复

当前状态：阶段 {{stage.id}} - {{stage.displayName}}

{{#if (eq status "in-progress")}}
🔵 正在进行中
{{agent}} 正在处理此阶段

待完成事项：
{{#each pendingTasks}}
- [ ] {{this}}
{{/each}}

输入 /ccteam-{{agent}} 与负责人对话
{{/if}}

{{#if (eq status "rejected")}}
🔴 需要修改
评审反馈：
{{#each reviewComments}}
- {{this}}
{{/each}}

输入 /ccteam-{{agent}} 与负责人讨论修改方案
{{/if}}
```

## 错误恢复

如果状态文件损坏：

```
⚠️ 项目状态文件可能已损坏

尝试恢复选项：
1. 从 git 历史恢复（推荐）
2. 根据已有文档重建状态
3. 重新开始项目

请选择 [1/2/3]:
```
```

**Step 4: Commit**

```bash
git add skills/entry/ccteam-new.md skills/entry/ccteam-status.md skills/entry/ccteam-resume.md
git commit -m "feat: integrate workflow engine into entry skills"
```

---

## 第二阶段：智能体协作优化 (P0)

### Task 4: 定义智能体权限矩阵

**目标：** 创建清晰的权限定义，解决职责边界模糊问题。

**Files:**
- Create: `agents/permissions.md`
- Create: `agents/collaboration-rules.md`

**Step 1: 创建权限矩阵 `agents/permissions.md`**

```markdown
# 智能体权限矩阵

## 概述

定义每个智能体在项目中的权限，包括创建、修改、评审、审批等操作。

## 权限级别

| 级别 | 代码 | 说明 |
|------|------|------|
| 创建 | C | 可以创建新内容 |
| 读取 | R | 可以查看内容 |
| 修改 | U | 可以修改内容 |
| 删除 | D | 可以删除内容 |
| 评审 | V | 可以评审内容 |
| 审批 | A | 可以最终审批 |

## 文档权限矩阵

| 文档类型 | PM | TL | Arch | FE | BE | API | QA | DevOps | DBA | Sec | Doc | CR |
|----------|----|----|------|----|----|-----|----|----|-----|-----|-----|-----|
| PRD | CRUD | RV | RV | R | R | R | R | R | R | R | R | R |
| 用户故事 | CRUD | RV | R | R | R | R | R | R | R | R | R | R |
| 技术方案 | R | CRUDA | CRUD | RV | RV | R | R | R | RV | RV | R | RV |
| 架构文档 | R | RV | CRUDA | RV | RV | R | R | R | RV | RV | R | R |
| UI设计稿 | RV | R | R | RV | R | R | R | R | R | R | R | R |
| 数据库设计 | R | RV | RVA | R | CRUD | R | R | R | CRUDA | RV | R | R |
| API文档 | R | RV | RV | RV | RV | CRUDA | R | R | R | RV | R | R |
| 前端代码 | R | RV | R | CRUD | R | R | RV | R | R | RV | R | RVA |
| 后端代码 | R | RV | R | R | CRUD | R | RV | R | R | RV | R | RVA |
| 测试文档 | R | RV | R | R | R | R | CRUDA | R | R | R | R | R |
| 部署配置 | R | RVA | RV | R | R | R | R | CRUD | R | RVA | R | R |
| 用户文档 | RV | R | R | R | R | R | R | R | R | R | CRUDA | R |

**角色缩写：**
- PM: product-manager
- TL: tech-lead
- Arch: architect
- FE: frontend-engineer
- BE: backend-engineer
- API: api-engineer
- QA: qa-engineer
- DevOps: devops-engineer
- DBA: dba
- Sec: security-engineer
- Doc: doc-engineer
- CR: code-reviewer

## 阶段操作权限

| 操作 | 可执行角色 |
|------|-----------|
| 启动阶段 | tech-lead, project-manager |
| 完成阶段 | 当前阶段负责人 |
| 跳过阶段 | tech-lead（需说明原因） |
| 回退阶段 | tech-lead, architect |
| 强制通过评审 | tech-lead（需说明原因） |

## 评审审批权限

| 阶段 | 评审人 | 最终审批人 |
|------|--------|-----------|
| 需求分析 | tech-lead | tech-lead |
| 技术评审 | 全员 | tech-lead + architect |
| UI/UX设计 | product-manager, frontend-engineer | product-manager |
| 数据库设计 | architect, backend-engineer | architect |
| API设计 | frontend-engineer, backend-engineer | tech-lead |
| 前端开发 | code-reviewer, tech-lead | code-reviewer |
| 后端开发 | code-reviewer, tech-lead, security-engineer | code-reviewer |
| 测试 | tech-lead, product-manager | tech-lead |
| 部署 | tech-lead, security-engineer | tech-lead |
| 文档归档 | product-manager, tech-lead | product-manager |

## 冲突解决机制

当角色职责重叠时的决策优先级：

### 技术决策
```
architect > tech-lead > backend-engineer/frontend-engineer
```

### 产品决策
```
product-manager > tech-lead > 其他角色
```

### 代码质量决策
```
code-reviewer > tech-lead > 开发工程师
```

### 安全决策
```
security-engineer > tech-lead > 其他角色
```

### 进度决策
```
project-manager > tech-lead > 其他角色
```
```

**Step 2: 创建协作规则 `agents/collaboration-rules.md`**

```markdown
# 智能体协作规则

## 概述

定义智能体之间的协作方式、沟通规范和冲突解决机制。

## 协作原则

1. **职责分明** - 每个智能体只负责自己职责范围内的工作
2. **主动沟通** - 发现问题及时通知相关角色
3. **尊重专业** - 尊重其他角色的专业判断
4. **整体优先** - 个人意见服从整体利益

## 角色职责边界

### 技术负责人 vs 架构师

| 维度 | 技术负责人 | 架构师 |
|------|-----------|--------|
| 关注点 | 项目整体技术方向 | 系统架构设计 |
| 决策范围 | 技术选型最终决策 | 架构方案设计 |
| 时间视角 | 当前迭代 | 长期演进 |
| 协作方式 | 架构师提出方案，技术负责人审批 |

**协作流程：**
```
架构师设计方案 → 技术负责人评审 → 通过/打回修改 → 技术负责人最终决策
```

### 代码审查员 vs 技术负责人

| 维度 | 代码审查员 | 技术负责人 |
|------|-----------|-----------|
| 关注点 | 代码质量和规范 | 技术实现和业务逻辑 |
| 决策范围 | 代码是否合格 | 方案是否合理 |
| 审查深度 | 详细审查每行代码 | 关注关键逻辑和设计 |

**协作流程：**
```
代码审查员审查 → 如有重大问题升级给技术负责人 → 技术负责人最终决策
```

### API工程师 vs 后端工程师

| 维度 | API工程师 | 后端工程师 |
|------|----------|-----------|
| 关注点 | API设计和文档 | API实现和业务逻辑 |
| 输出物 | API规范文档 | 实现代码 |
| 时间顺序 | 先设计 | 后实现 |

**协作流程：**
```
API工程师设计接口 → 前后端评审 → 后端工程师实现 → API工程师验证
```

### 前端工程师 vs UI设计师

| 维度 | 前端工程师 | UI设计师 |
|------|-----------|---------|
| 关注点 | 技术实现可行性 | 视觉效果和体验 |
| 决策范围 | 实现方式 | 设计方案 |
| 反馈方向 | 技术限制反馈给设计 | 设计规范交付给开发 |

**交付标准：**
- UI设计师必须提供完整的设计规范（颜色、字体、间距）
- 前端工程师在开发前确认设计的技术可行性
- 如有技术限制，双方协商替代方案

## 冲突处理流程

### 一级冲突：角色间意见不一致

```
1. 双方陈述各自观点和理由
2. 寻找共同点和折中方案
3. 如无法达成一致，升级到上级角色
```

### 二级冲突：技术方案分歧

```
升级路径：开发工程师 → 技术负责人 → 架构师

处理方式：
1. 列出各方案的优缺点
2. 评估对项目的影响
3. 由架构师或技术负责人裁决
```

### 三级冲突：业务需求分歧

```
升级路径：任意角色 → 产品经理 → 技术负责人

处理方式：
1. 回溯原始需求
2. 评估用户价值
3. 由产品经理裁决
```

## 沟通规范

### 信息同步

| 事件 | 通知对象 | 方式 |
|------|----------|------|
| 阶段开始 | 负责人、评审人、PM | 主动通知 |
| 阶段完成 | 评审人、下游角色 | 主动通知 |
| 发现问题 | 相关角色、TL | 立即通知 |
| 需求变更 | 所有角色 | 全员通知 |
| 技术风险 | TL、Arch、PM | 立即升级 |

### 反馈格式

```markdown
## 反馈通知

**发起人：** {{role}}
**时间：** {{timestamp}}
**类型：** 问题/建议/阻塞/风险

### 内容

{{详细描述}}

### 影响范围

{{受影响的角色/阶段}}

### 期望响应

{{期望的处理方式和时间}}
```

## 紧急情况处理

### 定义紧急情况

- 安全漏洞发现
- 核心功能阻塞
- 重大技术风险
- 需求重大变更

### 紧急处理流程

```
1. 发现人立即通知技术负责人
2. 技术负责人评估影响
3. 召集相关角色紧急讨论
4. 制定应对方案
5. 调整项目计划
6. 通知所有受影响角色
```

### 紧急情况的权限提升

在紧急情况下，技术负责人可以：
- 跳过常规评审流程
- 直接回退任何阶段
- 临时调整角色职责
- 暂停项目进度

所有紧急操作必须记录并在事后复盘。
```

**Step 3: Commit**

```bash
git add agents/permissions.md agents/collaboration-rules.md
git commit -m "feat: add agent permissions matrix and collaboration rules"
```

---

### Task 5: 增强智能体行为指导

**目标：** 为每个智能体添加场景指南和决策支持。

**Files:**
- Modify: `agents/product-manager/agent.md`
- Modify: `agents/tech-lead/agent.md`
- Modify: `agents/architect/agent.md`
- Modify: `agents/frontend-engineer/agent.md`
- Modify: `agents/backend-engineer/agent.md`
- Create: `agents/scenario-guides/common-scenarios.md`

**Step 1: 创建通用场景指南 `agents/scenario-guides/common-scenarios.md`**

```markdown
# 智能体通用场景指南

## 1. 如何拒绝不合理需求

### 场景识别

不合理需求的特征：
- 超出技术能力范围
- 违背用户价值原则
- 与现有功能冲突
- 资源投入与价值不匹配
- 违反安全或合规要求

### 处理步骤

```
1. 确认理解需求
   "我理解您希望实现 XXX 功能，目的是 YYY，对吗？"

2. 说明问题
   "这个需求存在以下问题：
   - 问题1：...
   - 问题2：..."

3. 提供替代方案
   "我建议可以考虑以下替代方案：
   - 方案A：...（推荐）
   - 方案B：..."

4. 请求决策
   "请您评估以上方案，或者如果有其他考虑请告诉我。"
```

### 示例

```
用户："我想在登录页面加一个实时股票行情显示"

产品经理回应：
"我理解您希望在登录页面添加股票行情功能。让我分析一下：

❌ 问题：
1. 与登录场景无关，可能分散用户注意力
2. 需要额外接入第三方数据源，增加系统复杂度
3. 可能涉及金融数据合规问题

✅ 建议：
如果用户确实需要股票信息，建议：
1. 在登录后的首页添加股票模块
2. 或者将股票功能作为独立模块规划

您觉得这样的方案如何？"
```

## 2. 如何处理技术债务

### 发现时机

- 代码评审发现历史问题
- 性能测试发现瓶颈
- 新功能开发受阻
- 安全扫描发现漏洞

### 评估标准

| 影响级别 | 定义 | 处理优先级 |
|----------|------|-----------|
| 致命 | 影响核心功能或安全 | 立即处理 |
| 严重 | 影响性能或维护性 | 本迭代处理 |
| 一般 | 代码质量问题 | 下次迭代处理 |
| 轻微 | 代码风格问题 | 择机处理 |

### 记录格式

```markdown
## 技术债务记录

**ID：** TD-{{序号}}
**发现人：** {{角色}}
**发现时间：** {{时间}}
**影响级别：** 致命/严重/一般/轻微

### 问题描述
{{详细描述}}

### 影响范围
{{受影响的模块/功能}}

### 风险评估
{{如不处理可能导致的问题}}

### 建议方案
{{修复方案和预估工作量}}

### 处理状态
- [ ] 已记录
- [ ] 已评估
- [ ] 已排期
- [ ] 已修复
```

## 3. 如何处理安全漏洞

### 紧急响应流程

```
1. 发现漏洞 → 立即通知安全工程师
2. 安全工程师评估严重程度
3. 如为高危漏洞：
   - 暂停相关功能
   - 通知技术负责人
   - 启动紧急修复
4. 修复完成后：
   - 安全测试验证
   - 复盘并记录
```

### 严重程度评估

| 级别 | 定义 | 响应时间 |
|------|------|----------|
| 紧急 | 已被利用或极易利用 | 立即 |
| 高危 | 可导致数据泄露或系统入侵 | 24小时内 |
| 中危 | 需要特定条件才能利用 | 72小时内 |
| 低危 | 理论上存在风险 | 下个版本 |

## 4. 如何处理需求变更

### 变更评估流程

```
1. 产品经理提出变更
2. 技术负责人评估影响
3. 架构师评估技术可行性
4. 项目经理评估进度影响
5. 全员评审变更方案
6. 决策：接受/拒绝/延期
```

### 评估维度

| 维度 | 评估问题 |
|------|----------|
| 范围 | 变更涉及多少功能？ |
| 技术 | 是否需要架构调整？ |
| 进度 | 是否影响上线时间？ |
| 资源 | 是否需要额外资源？ |
| 风险 | 有哪些潜在风险？ |

### 变更记录格式

```markdown
## 需求变更申请

**变更ID：** CR-{{序号}}
**申请人：** {{角色}}
**申请时间：** {{时间}}

### 变更内容
{{原需求}} → {{新需求}}

### 变更原因
{{为什么需要变更}}

### 影响评估
| 维度 | 影响程度 | 说明 |
|------|----------|------|
| 进度 | 高/中/低 | {{说明}} |
| 成本 | 高/中/低 | {{说明}} |
| 风险 | 高/中/低 | {{说明}} |

### 评审结论
- [ ] 接受
- [ ] 拒绝（原因：）
- [ ] 延期（至：）
```

## 5. 如何处理紧急Bug

### 优先级判定

| 优先级 | 判定条件 | 响应方式 |
|--------|----------|----------|
| P0 | 系统崩溃/数据丢失/安全漏洞 | 立即修复，全员支持 |
| P1 | 核心功能不可用 | 当天修复 |
| P2 | 功能异常但有绑定方案 | 48小时内修复 |
| P3 | 体验问题 | 下个版本修复 |

### 紧急修复流程

```
1. 复现确认 → 确认是真实bug
2. 影响评估 → 确定优先级
3. 指派处理 → 对应开发工程师
4. 修复验证 → QA验证
5. 紧急上线 → DevOps执行
6. 复盘记录 → 文档工程师
```

## 6. 如何进行技术选型

### 选型原则

1. **成熟稳定** - 优先选择成熟的技术
2. **团队熟悉** - 考虑团队的技术储备
3. **社区活跃** - 选择有活跃社区的技术
4. **长期支持** - 避免即将淘汰的技术
5. **性能适配** - 满足性能要求

### 选型流程

```
1. 明确需求 → 技术需要解决什么问题？
2. 候选方案 → 列出2-3个候选技术
3. 对比评估 → 从多维度对比
4. POC验证 → 必要时做技术验证
5. 团队评审 → 技术评审会讨论
6. 最终决策 → 技术负责人决定
```

### 评估模板

```markdown
## 技术选型评估

**选型项目：** {{要解决的问题}}

### 候选方案

| 维度 | 方案A | 方案B | 方案C |
|------|-------|-------|-------|
| 成熟度 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| 性能 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| 学习成本 | 低 | 中 | 高 |
| 社区支持 | 活跃 | 活跃 | 一般 |
| 团队熟悉度 | 高 | 中 | 低 |

### 推荐方案

推荐 **方案A**，原因：
1. ...
2. ...
```
```

**Step 2: 为产品经理添加场景指南**

在 `agents/product-manager/agent.md` 末尾添加：

```markdown

## 场景处理指南

### 场景1：用户需求模糊不清

**识别特征：**
- "我想要一个XX功能"但没有具体描述
- 只描述了表象问题，没有说明根本需求

**处理方式：**
```
1. 使用"5个为什么"追问法挖掘真实需求
2. 请用户描述具体使用场景
3. 提供相似功能的参考案例
4. 整理理解后请用户确认
```

### 场景2：多个需求优先级冲突

**处理方式：**
```
评估维度（各占权重）：
- 用户价值（40%）：解决了多少用户的多大痛点
- 业务价值（30%）：对业务指标的影响
- 实现成本（20%）：开发资源投入
- 风险（10%）：延迟上线的风险

综合评分后排序
```

### 场景3：技术说需求不可行

**处理方式：**
```
1. 确认理解技术的限制原因
2. 评估需求的核心价值
3. 与技术讨论替代方案
4. 如实在无法实现，调整需求范围
5. 记录技术限制，作为未来迭代参考
```

### 场景4：需求评审被打回

**处理方式：**
```
1. 仔细阅读评审反馈
2. 逐条分析问题原因
3. 修改文档解决问题
4. 对有争议的点与评审人沟通
5. 重新提交评审
```

## 质量自检清单

在提交评审前，确认以下内容：

- [ ] 目标用户已明确定义
- [ ] 核心功能已识别并排序
- [ ] 每个功能都有用户故事支撑
- [ ] 验收标准可量化
- [ ] 异常场景已考虑
- [ ] 非功能性需求已定义
- [ ] 与技术负责人确认过可行性
```

**Step 3: Commit**

```bash
git add agents/scenario-guides/ agents/product-manager/agent.md
git commit -m "feat: add scenario guides and enhance agent behavior instructions"
```

---

## 第三阶段：技能体系完善 (P1)

### Task 6: 创建核心技能定义

**目标：** 为 available-skills.json 中引用的技能创建实际定义文件。

**Files:**
- Create: `skills/requirements/analyze-requirement.md`
- Create: `skills/requirements/write-prd.md`
- Create: `skills/requirements/write-user-story.md`
- Create: `skills/architecture/system-design.md`
- Create: `skills/architecture/tech-selection.md`
- Create: `skills/development/code-implementation.md`
- Create: `skills/development/code-review.md`
- Create: `skills/testing/test-planning.md`
- Create: `skills/testing/test-execution.md`

**Step 1: 创建需求分析技能 `skills/requirements/analyze-requirement.md`**

```markdown
# 需求分析 (Analyze Requirement)

## 概述

分析用户原始需求，挖掘真实需求，转化为可执行的产品需求。

## 适用角色

- product-manager（主要）
- tech-lead（辅助评估）

## 输入

- 用户原始需求描述
- 背景信息
- 参考资料（可选）

## 输出

- 需求分析报告
- 核心问题识别
- 初步功能清单

## 执行步骤

### Step 1: 需求理解

```
1. 仔细阅读用户需求描述
2. 标注关键词和疑问点
3. 列出需要澄清的问题
```

### Step 2: 需求澄清

与用户确认以下问题：

```
1. 目标用户是谁？
2. 要解决什么问题？
3. 期望的使用场景？
4. 成功的标准是什么？
5. 有什么约束条件？
```

### Step 3: 需求拆解

```
原始需求 → 用户目标 → 功能需求 → 具体功能点

示例：
"我想要一个任务管理系统"
↓
用户目标：高效管理日常任务
↓
功能需求：创建任务、分配任务、跟踪进度、提醒通知
↓
功能点：
- 创建任务（标题、描述、截止日期、优先级）
- 任务列表（筛选、排序、搜索）
- 任务状态（待办、进行中、已完成）
- ...
```

### Step 4: 优先级评估

使用 MoSCoW 方法：

| 类别 | 说明 | 优先级 |
|------|------|--------|
| Must | 必须有，没有就不能用 | P0 |
| Should | 应该有，核心体验需要 | P1 |
| Could | 可以有，锦上添花 | P2 |
| Won't | 不做，超出范围 | - |

### Step 5: 输出需求分析报告

```markdown
## 需求分析报告

### 1. 需求背景
{{背景描述}}

### 2. 目标用户
{{用户画像}}

### 3. 核心问题
{{要解决的问题}}

### 4. 功能清单

| 功能 | 优先级 | 说明 |
|------|--------|------|
| 功能1 | P0 | ... |
| 功能2 | P1 | ... |

### 5. 约束条件
{{技术、时间、资源约束}}

### 6. 待确认事项
{{需要进一步澄清的问题}}
```

## 质量检查

- [ ] 是否理解了用户的真实需求（而非表面需求）
- [ ] 是否识别了核心功能
- [ ] 优先级是否合理
- [ ] 是否考虑了边界情况
```

**Step 2: 创建架构设计技能 `skills/architecture/system-design.md`**

```markdown
# 系统架构设计 (System Design)

## 概述

设计系统整体架构，确定模块划分、技术选型和关键设计决策。

## 适用角色

- architect（主要）
- tech-lead（评审）

## 输入

- PRD文档
- 用户故事
- 非功能性需求

## 输出

- 系统架构图
- 模块划分
- 技术选型建议
- 架构决策记录

## 执行步骤

### Step 1: 需求分析

```
1. 阅读PRD，提取技术相关需求
2. 识别核心业务实体
3. 分析数据流向
4. 明确性能、安全、可用性要求
```

### Step 2: 架构选型

根据项目特点选择架构模式：

| 项目特点 | 推荐架构 |
|----------|----------|
| 小型项目、快速迭代 | 单体架构 |
| 中型项目、团队协作 | 模块化单体 |
| 大型项目、独立部署 | 微服务架构 |
| 事件驱动、高并发 | 事件驱动架构 |

### Step 3: 模块划分

```
原则：
- 高内聚低耦合
- 单一职责
- 接口隔离
- 依赖倒置

示例模块划分：
├── 表示层 (Presentation)
│   ├── Web端
│   └── Mobile端
├── 应用层 (Application)
│   ├── 用户服务
│   └── 业务服务
├── 领域层 (Domain)
│   ├── 实体
│   └── 业务逻辑
└── 基础设施层 (Infrastructure)
    ├── 数据库
    ├── 缓存
    └── 外部服务
```

### Step 4: 绘制架构图

```
使用标准符号：
- 矩形：服务/模块
- 圆柱：数据库
- 箭头：数据流向
- 云形：外部服务

图层次：
1. 系统上下文图（C4 Level 1）
2. 容器图（C4 Level 2）
3. 组件图（C4 Level 3）
```

### Step 5: 记录架构决策

使用 ADR (Architecture Decision Record) 格式：

```markdown
## ADR-001: {{决策标题}}

### 状态
已接受 / 已废弃 / 待定

### 背景
{{为什么需要做这个决策}}

### 决策
{{做出的决策}}

### 后果
{{这个决策带来的影响}}

### 备选方案
{{考虑过的其他方案}}
```

## 输出模板

```markdown
## 系统架构设计

### 1. 架构概述
{{一句话描述架构特点}}

### 2. 架构图
{{ASCII或Mermaid图}}

### 3. 模块说明

| 模块 | 职责 | 技术栈 |
|------|------|--------|
| ... | ... | ... |

### 4. 关键设计决策

#### ADR-001: ...
#### ADR-002: ...

### 5. 非功能性设计

| 维度 | 目标 | 实现方式 |
|------|------|----------|
| 性能 | ... | ... |
| 安全 | ... | ... |
| 可用性 | ... | ... |
```
```

**Step 3: 创建代码实现技能 `skills/development/code-implementation.md`**

```markdown
# 代码实现 (Code Implementation)

## 概述

根据设计文档实现功能代码，确保代码质量和规范。

## 适用角色

- frontend-engineer
- backend-engineer

## 输入

- 技术设计文档
- API规范
- UI设计稿（前端）

## 输出

- 功能代码
- 单元测试
- 代码文档

## 执行步骤

### Step 1: 理解需求

```
1. 阅读相关设计文档
2. 明确要实现的功能点
3. 确认接口规范
4. 识别依赖和约束
```

### Step 2: 设计实现方案

```
1. 拆分任务为小的实现单元
2. 设计数据结构和接口
3. 考虑边界情况
4. 规划测试策略
```

### Step 3: TDD开发（推荐）

```
1. 先写测试（失败）
2. 实现最小代码（通过）
3. 重构优化
4. 重复
```

### Step 4: 代码规范

#### 命名规范

```
变量/函数：camelCase
常量：UPPER_SNAKE_CASE
类/组件：PascalCase
文件：kebab-case 或 PascalCase

命名原则：
- 表意清晰
- 避免缩写（除非是通用缩写）
- 动词开头表示操作（getUserInfo, createOrder）
- 布尔值用 is/has/can 开头
```

#### 代码结构

```
- 单个函数不超过50行
- 嵌套不超过3层
- 参数不超过5个
- 相关代码放在一起
```

#### 注释规范

```
- 复杂逻辑必须注释
- 公共API必须有文档注释
- 不注释显而易见的代码
- 用代码表达意图，用注释解释为什么
```

### Step 5: 单元测试

```
测试覆盖要求：
- 核心业务逻辑 100%
- 工具函数 80%
- UI组件 60%

测试命名：
test_<功能>_<场景>_<期望结果>

示例：
test_createUser_withValidData_shouldReturnUserId
test_createUser_withDuplicateEmail_shouldThrowError
```

### Step 6: 提交代码

```
提交信息格式：
<type>(<scope>): <subject>

type: feat/fix/refactor/test/docs/chore
scope: 影响范围
subject: 简短描述

示例：
feat(user): add user registration API
fix(auth): fix token expiration issue
```

## 质量自检

- [ ] 代码符合规范
- [ ] 测试覆盖核心逻辑
- [ ] 无明显安全漏洞
- [ ] 无性能问题
- [ ] 文档/注释完整
- [ ] 通过本地测试
```

**Step 4: Commit**

```bash
git add skills/requirements/ skills/architecture/ skills/development/ skills/testing/
git commit -m "feat: add core skill definitions for requirements, architecture, development, and testing"
```

---

## 第四阶段：文档模板补充 (P1)

### Task 7: 补充缺失的文档模板

**目标：** 创建缺失的关键文档模板。

**Files:**
- Create: `templates/ui-design-spec.md`
- Create: `templates/project-plan.md`
- Create: `templates/weekly-report.md`
- Create: `templates/risk-management.md`
- Create: `templates/change-request.md`
- Create: `templates/meeting-notes.md`

**Step 1: 创建UI设计规范模板 `templates/ui-design-spec.md`**

```markdown
# UI设计规范

## 文档信息

| 属性 | 值 |
|------|-----|
| 项目名称 | {{project_name}} |
| 版本 | v1.0 |
| 创建日期 | {{date}} |
| 作者 | UI设计师 |

---

## 1. 设计原则

### 1.1 核心原则

- **简洁清晰** - 减少视觉噪音，突出核心内容
- **一致性** - 保持视觉和交互的一致性
- **可用性** - 易于理解和操作
- **响应式** - 适配多种设备尺寸

### 1.2 品牌调性

<!-- 描述产品的视觉调性 -->

---

## 2. 颜色系统

### 2.1 主色板

| 颜色名称 | 色值 | 使用场景 |
|----------|------|----------|
| Primary | #3B82F6 | 主要按钮、链接、强调 |
| Primary Light | #93C5FD | 悬停状态、次要强调 |
| Primary Dark | #1D4ED8 | 按下状态 |

### 2.2 中性色

| 颜色名称 | 色值 | 使用场景 |
|----------|------|----------|
| Gray 900 | #111827 | 主要文字 |
| Gray 600 | #4B5563 | 次要文字 |
| Gray 400 | #9CA3AF | 占位文字 |
| Gray 200 | #E5E7EB | 边框 |
| Gray 100 | #F3F4F6 | 背景 |

### 2.3 语义色

| 颜色名称 | 色值 | 使用场景 |
|----------|------|----------|
| Success | #10B981 | 成功状态 |
| Warning | #F59E0B | 警告状态 |
| Error | #EF4444 | 错误状态 |
| Info | #3B82F6 | 信息提示 |

---

## 3. 字体系统

### 3.1 字体族

```css
--font-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
--font-mono: 'JetBrains Mono', 'Fira Code', monospace;
```

### 3.2 字体大小

| 名称 | 大小 | 行高 | 使用场景 |
|------|------|------|----------|
| xs | 12px | 16px | 辅助文字 |
| sm | 14px | 20px | 正文（小） |
| base | 16px | 24px | 正文 |
| lg | 18px | 28px | 小标题 |
| xl | 20px | 28px | 标题 |
| 2xl | 24px | 32px | 页面标题 |
| 3xl | 30px | 36px | 大标题 |

### 3.3 字重

| 名称 | 值 | 使用场景 |
|------|-----|----------|
| Normal | 400 | 正文 |
| Medium | 500 | 强调、按钮 |
| Semibold | 600 | 小标题 |
| Bold | 700 | 标题 |

---

## 4. 间距系统

### 4.1 基础间距

基准单位：4px

| 名称 | 值 | 使用场景 |
|------|-----|----------|
| 1 | 4px | 紧凑间距 |
| 2 | 8px | 元素内部间距 |
| 3 | 12px | 相关元素间距 |
| 4 | 16px | 组件内部间距 |
| 6 | 24px | 区块间距 |
| 8 | 32px | 大区块间距 |

### 4.2 布局间距

| 区域 | 内边距 | 外边距 |
|------|--------|--------|
| 页面 | 24px | - |
| 卡片 | 16px | 16px |
| 表单项 | - | 24px（垂直） |

---

## 5. 圆角规范

| 名称 | 值 | 使用场景 |
|------|-----|----------|
| none | 0 | 无圆角 |
| sm | 4px | 小按钮、标签 |
| md | 8px | 按钮、输入框 |
| lg | 12px | 卡片 |
| xl | 16px | 模态框 |
| full | 9999px | 圆形按钮、头像 |

---

## 6. 阴影规范

| 名称 | 值 | 使用场景 |
|------|-----|----------|
| sm | 0 1px 2px rgba(0,0,0,0.05) | 轻微提升 |
| md | 0 4px 6px rgba(0,0,0,0.1) | 卡片 |
| lg | 0 10px 15px rgba(0,0,0,0.1) | 下拉菜单 |
| xl | 0 20px 25px rgba(0,0,0,0.15) | 模态框 |

---

## 7. 组件规范

### 7.1 按钮

| 类型 | 样式 | 使用场景 |
|------|------|----------|
| Primary | 实心主色 | 主要操作 |
| Secondary | 边框主色 | 次要操作 |
| Ghost | 文字 | 三级操作 |
| Danger | 实心红色 | 危险操作 |

**尺寸：**
| 尺寸 | 高度 | 内边距 | 字号 |
|------|------|--------|------|
| sm | 32px | 12px | 14px |
| md | 40px | 16px | 14px |
| lg | 48px | 20px | 16px |

### 7.2 输入框

- 高度：40px（默认）
- 内边距：12px 16px
- 边框：1px solid Gray 200
- 圆角：8px
- 聚焦：2px Primary 边框

### 7.3 卡片

- 背景：白色
- 边框：1px solid Gray 200（可选）
- 圆角：12px
- 阴影：md
- 内边距：16px

---

## 8. 响应式断点

| 名称 | 范围 | 设备 |
|------|------|------|
| sm | < 640px | 手机 |
| md | 640px - 1024px | 平板 |
| lg | 1024px - 1280px | 笔记本 |
| xl | > 1280px | 桌面 |

---

## 变更记录

| 版本 | 日期 | 修改人 | 修改内容 |
|------|------|--------|----------|
| v1.0 | {{date}} | - | 初始版本 |
```

**Step 2: 创建项目计划模板 `templates/project-plan.md`**

```markdown
# 项目计划

## 文档信息

| 属性 | 值 |
|------|-----|
| 项目名称 | {{project_name}} |
| 版本 | v1.0 |
| 创建日期 | {{date}} |
| 作者 | 项目经理 |

---

## 1. 项目概述

### 1.1 项目背景
{{项目背景描述}}

### 1.2 项目目标
{{项目目标描述}}

### 1.3 成功标准
{{如何衡量项目成功}}

---

## 2. 范围定义

### 2.1 项目范围内

- 功能1
- 功能2
- 功能3

### 2.2 项目范围外

- 不包含1
- 不包含2

---

## 3. 里程碑计划

| 里程碑 | 目标日期 | 交付物 | 状态 |
|--------|----------|--------|------|
| M1 需求确认 | {{date}} | PRD文档 | ⬜ 待开始 |
| M2 设计完成 | {{date}} | 设计稿、技术方案 | ⬜ 待开始 |
| M3 开发完成 | {{date}} | 可测试版本 | ⬜ 待开始 |
| M4 测试完成 | {{date}} | 测试报告 | ⬜ 待开始 |
| M5 上线发布 | {{date}} | 正式版本 | ⬜ 待开始 |

---

## 4. 任务分解 (WBS)

### 4.1 需求分析阶段

| 任务 | 负责人 | 开始 | 结束 | 状态 |
|------|--------|------|------|------|
| 需求调研 | PM | - | - | ⬜ |
| PRD编写 | PM | - | - | ⬜ |
| 需求评审 | PM | - | - | ⬜ |

### 4.2 设计阶段

| 任务 | 负责人 | 开始 | 结束 | 状态 |
|------|--------|------|------|------|
| UI设计 | UI | - | - | ⬜ |
| 架构设计 | Arch | - | - | ⬜ |
| 数据库设计 | DBA | - | - | ⬜ |
| API设计 | API | - | - | ⬜ |

### 4.3 开发阶段

| 任务 | 负责人 | 开始 | 结束 | 状态 |
|------|--------|------|------|------|
| 前端开发 | FE | - | - | ⬜ |
| 后端开发 | BE | - | - | ⬜ |
| 联调测试 | FE+BE | - | - | ⬜ |

### 4.4 测试阶段

| 任务 | 负责人 | 开始 | 结束 | 状态 |
|------|--------|------|------|------|
| 测试用例 | QA | - | - | ⬜ |
| 功能测试 | QA | - | - | ⬜ |
| 性能测试 | QA | - | - | ⬜ |
| 修复Bug | Dev | - | - | ⬜ |

### 4.5 上线阶段

| 任务 | 负责人 | 开始 | 结束 | 状态 |
|------|--------|------|------|------|
| 部署准备 | DevOps | - | - | ⬜ |
| 正式部署 | DevOps | - | - | ⬜ |
| 上线验证 | QA | - | - | ⬜ |

---

## 5. 资源计划

### 5.1 团队成员

| 角色 | 人员 | 投入度 |
|------|------|--------|
| 产品经理 | - | 100% |
| 技术负责人 | - | 50% |
| 前端工程师 | - | 100% |
| 后端工程师 | - | 100% |
| 测试工程师 | - | 100% |

### 5.2 其他资源

| 资源 | 说明 |
|------|------|
| 服务器 | - |
| 第三方服务 | - |

---

## 6. 风险管理

| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 需求变更 | 高 | 中 | 预留缓冲时间 |
| 技术难点 | 中 | 高 | 提前技术调研 |
| 人员变动 | 低 | 高 | 知识文档化 |

---

## 7. 沟通计划

| 会议 | 频率 | 参与人 | 目的 |
|------|------|--------|------|
| 日站会 | 每日 | 全员 | 同步进度 |
| 周会 | 每周 | 全员 | 总结复盘 |
| 评审会 | 按需 | 相关人员 | 交付评审 |

---

## 变更记录

| 版本 | 日期 | 修改人 | 修改内容 |
|------|------|--------|----------|
| v1.0 | {{date}} | - | 初始版本 |
```

**Step 3: 创建周报模板 `templates/weekly-report.md`**

```markdown
# 项目周报

## 基本信息

| 属性 | 值 |
|------|-----|
| 项目名称 | {{project_name}} |
| 报告周期 | {{week_start}} - {{week_end}} |
| 报告人 | {{reporter}} |
| 报告日期 | {{date}} |

---

## 1. 本周概要

**整体状态：** 🟢 正常 / 🟡 有风险 / 🔴 有阻塞

**进度：** {{completed_percentage}}%

```
[████████████░░░░░░░░] 60%
```

---

## 2. 本周完成

| 任务 | 负责人 | 计划 | 实际 | 状态 |
|------|--------|------|------|------|
| 任务1 | - | - | - | ✅ 完成 |
| 任务2 | - | - | - | ✅ 完成 |

---

## 3. 进行中

| 任务 | 负责人 | 进度 | 预计完成 |
|------|--------|------|----------|
| 任务3 | - | 60% | {{date}} |
| 任务4 | - | 30% | {{date}} |

---

## 4. 下周计划

| 任务 | 负责人 | 优先级 | 目标 |
|------|--------|--------|------|
| 任务5 | - | P0 | - |
| 任务6 | - | P1 | - |

---

## 5. 风险与问题

### 当前风险

| 风险 | 影响 | 状态 | 应对措施 |
|------|------|------|----------|
| 风险1 | 高/中/低 | 🟡 观察中 | - |

### 已解决问题

| 问题 | 解决方案 | 解决日期 |
|------|----------|----------|
| 问题1 | - | {{date}} |

### 待解决问题

| 问题 | 负责人 | 预计解决 |
|------|--------|----------|
| 问题2 | - | {{date}} |

---

## 6. 需要支持

<!-- 列出需要其他人或部门支持的事项 -->

- 支持事项1
- 支持事项2

---

## 7. 下周重点

1. 重点1
2. 重点2
3. 重点3
```

**Step 4: 创建变更申请模板 `templates/change-request.md`**

```markdown
# 变更申请

## 基本信息

| 属性 | 值 |
|------|-----|
| 变更编号 | CR-{{序号}} |
| 项目名称 | {{project_name}} |
| 申请人 | {{applicant}} |
| 申请日期 | {{date}} |
| 变更类型 | 需求变更 / 技术变更 / 范围变更 |
| 紧急程度 | 紧急 / 高 / 中 / 低 |

---

## 1. 变更内容

### 1.1 原始状态
{{原来的需求/设计/功能}}

### 1.2 期望状态
{{变更后的需求/设计/功能}}

### 1.3 变更原因
{{为什么需要变更}}

---

## 2. 影响分析

### 2.1 范围影响

| 影响模块 | 影响程度 | 说明 |
|----------|----------|------|
| 模块1 | 高/中/低 | - |
| 模块2 | 高/中/低 | - |

### 2.2 进度影响

| 原计划完成 | 变更后完成 | 延期天数 |
|------------|------------|----------|
| {{date}} | {{date}} | X天 |

### 2.3 成本影响

| 项目 | 增加工作量 | 说明 |
|------|------------|------|
| 设计 | X人天 | - |
| 开发 | X人天 | - |
| 测试 | X人天 | - |
| **合计** | **X人天** | - |

### 2.4 风险评估

| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 风险1 | 高/中/低 | 高/中/低 | - |

---

## 3. 实施方案

### 3.1 变更步骤

1. 步骤1
2. 步骤2
3. 步骤3

### 3.2 回滚方案

{{如果变更失败，如何回滚}}

---

## 4. 审批意见

### 技术评审

| 评审人 | 角色 | 意见 | 日期 |
|--------|------|------|------|
| - | 技术负责人 | 同意/不同意 | - |
| - | 架构师 | 同意/不同意 | - |

### 产品评审

| 评审人 | 角色 | 意见 | 日期 |
|--------|------|------|------|
| - | 产品经理 | 同意/不同意 | - |

### 最终决策

| 决策人 | 决策 | 日期 |
|--------|------|------|
| - | 批准/拒绝/延期 | - |

---

## 5. 变更记录

| 版本 | 日期 | 修改人 | 修改内容 |
|------|------|--------|----------|
| v1.0 | {{date}} | - | 初始提交 |
```

**Step 5: Commit**

```bash
git add templates/ui-design-spec.md templates/project-plan.md templates/weekly-report.md templates/risk-management.md templates/change-request.md templates/meeting-notes.md
git commit -m "feat: add missing document templates"
```

---

### Task 8: 更新 package.json 和 README

**目标：** 更新配置文件，反映新增的文件和功能。

**Files:**
- Modify: `package.json`
- Modify: `README.md`

**Step 1: 更新 package.json**

添加新的 skills 引用：

```json
{
  "claude": {
    "skills": {
      "workflow": [
        "skills/workflow/engine.md",
        "skills/workflow/stage-control.md",
        "skills/workflow/review-gate.md"
      ],
      "requirements": [
        "skills/requirements/analyze-requirement.md",
        "skills/requirements/write-prd.md",
        "skills/requirements/write-user-story.md"
      ],
      "architecture": [
        "skills/architecture/system-design.md",
        "skills/architecture/tech-selection.md"
      ],
      "development": [
        "skills/development/code-implementation.md",
        "skills/development/code-review.md"
      ],
      "testing": [
        "skills/testing/test-planning.md",
        "skills/testing/test-execution.md"
      ]
    }
  }
}
```

**Step 2: 更新 README.md**

添加新增功能的说明和使用指南。

**Step 3: Commit**

```bash
git add package.json README.md
git commit -m "docs: update package.json and README with new features"
```

---

## 执行顺序总结

| 任务 | 优先级 | 预估复杂度 | 依赖 |
|------|--------|-----------|------|
| Task 1: 状态Schema | P0 | 低 | 无 |
| Task 2: 流程引擎技能 | P0 | 高 | Task 1 |
| Task 3: 更新入口技能 | P0 | 中 | Task 2 |
| Task 4: 权限矩阵 | P0 | 中 | 无 |
| Task 5: 场景指南 | P0 | 中 | Task 4 |
| Task 6: 核心技能定义 | P1 | 高 | 无 |
| Task 7: 文档模板 | P1 | 中 | 无 |
| Task 8: 更新配置 | P1 | 低 | Task 6, 7 |

---

## 验证检查点

每完成一个 Task 后验证：

1. **Task 1-3 完成后**：
   - [ ] 可以使用 `/ccteam-new` 创建项目并生成 status 文件
   - [ ] 可以使用 `/ccteam-status` 查看项目状态
   - [ ] 阶段可以正常流转

2. **Task 4-5 完成后**：
   - [ ] 权限矩阵文档完整
   - [ ] 场景指南覆盖常见场景
   - [ ] 智能体知道如何处理冲突

3. **Task 6-8 完成后**：
   - [ ] 所有技能都有实际定义
   - [ ] 模板覆盖全流程
   - [ ] 配置文件正确反映新增内容
