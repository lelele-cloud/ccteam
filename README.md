# CCTeam - 互联网大厂开发团队

将 Claude Code 转变为一支完整的互联网大厂开发团队，包含15个专业角色智能体，遵循标准软件开发流程。

## v2.0 新特性

- **完整工作流引擎**：基于状态机的10阶段流程控制，支持自动/手动阶段转换
- **JSON Schema 项目状态管理**：结构化的项目状态追踪，支持持久化和恢复
- **智能体权限矩阵**：15个角色×22种文档的精细权限控制（CRUD+VA）
- **协作规则系统**：角色边界定义、冲突解决机制、紧急情况处理
- **场景行为指南**：6大通用场景 + 角色特定场景的处理指导
- **评审门禁机制**：可配置的评审规则（单人/多数/全票通过）
- **核心技能库**：需求分析、架构设计、代码实现、测试执行等完整技能定义
- **18个文档模板**：覆盖全开发周期的标准化文档模板

## 特性

- **15个专业角色**：产品经理、项目经理、技术负责人、架构师、UI设计师、UX设计师、前端工程师、后端工程师、API工程师、QA工程师、DevOps工程师、DBA、安全工程师、文档工程师、代码审查员
- **10阶段开发流程**：需求分析 → 技术评审 → UI/UX设计 → 数据库设计 → API设计 → 前端开发 → 后端开发 → 测试 → 部署 → 文档归档
- **200+技能库**：涵盖需求、架构、设计、开发、测试、运维等各个领域
- **14个MCP工具**：Playwright、SQLite、PostgreSQL、MySQL、Redis、GitHub、GitLab、Figma、Fetch、Docker、Sentry、Notion、Slack、OpenAPI
- **多项目类型支持**：Web应用、移动端（React Native/Flutter）、小程序
- **文档自动化**：自动生成PRD、技术方案、API文档、测试报告等

## 快速开始

### 安装

将本插件目录放置于 Claude Code 的 skills 目录下。

### 使用

```bash
# 启动新项目
/ccteam-new

# 添加新功能
/ccteam-feature

# 查看项目状态
/ccteam-status

# 恢复项目
/ccteam-resume

# 配置设置
/ccteam-config

# 阶段控制（v2.0新增）
/ccteam-stage start|complete|skip|rollback|status
```

### 调用特定角色

```bash
# 产品经理
/ccteam-pm

# 技术负责人
/ccteam-tech-lead

# 架构师
/ccteam-architect

# 前端工程师
/ccteam-frontend

# 后端工程师
/ccteam-backend

# 更多角色...
```

## 目录结构

```
ccteam/
├── agents/                    # 智能体定义
│   ├── product-manager/       # 产品经理
│   ├── project-manager/       # 项目经理
│   ├── tech-lead/            # 技术负责人
│   ├── architect/            # 架构师
│   ├── ui-designer/          # UI设计师
│   ├── ux-designer/          # UX设计师
│   ├── frontend-engineer/    # 前端工程师
│   ├── backend-engineer/     # 后端工程师
│   ├── api-engineer/         # API工程师
│   ├── qa-engineer/          # QA工程师
│   ├── devops-engineer/      # DevOps工程师
│   ├── dba/                  # DBA
│   ├── security-engineer/    # 安全工程师
│   ├── doc-engineer/         # 文档工程师
│   ├── code-reviewer/        # 代码审查员
│   ├── permissions.md        # 权限矩阵 (v2.0)
│   ├── collaboration-rules.md # 协作规则 (v2.0)
│   └── scenario-guides/      # 场景指南 (v2.0)
├── skills/                    # 技能定义
│   ├── entry/                # 入口技能
│   ├── workflow/             # 工作流技能 (v2.0)
│   │   ├── engine.md         # 流程引擎
│   │   ├── stage-control.md  # 阶段控制
│   │   └── review-gate.md    # 评审门禁
│   ├── requirements/         # 需求技能 (v2.0)
│   │   ├── analyze-requirement.md
│   │   ├── write-prd.md
│   │   └── write-user-story.md
│   ├── architecture/         # 架构技能 (v2.0)
│   │   ├── system-design.md
│   │   └── tech-selection.md
│   ├── development/          # 开发技能 (v2.0)
│   │   ├── code-implementation.md
│   │   └── code-review.md
│   ├── testing/              # 测试技能 (v2.0)
│   │   ├── test-planning.md
│   │   └── test-execution.md
│   └── agents/               # 角色快捷技能
├── schemas/                   # JSON Schema (v2.0)
│   └── project-status.schema.json
├── workflows/                 # 工作流定义
│   ├── engine.md             # 流程引擎
│   └── stages/               # 10个阶段定义
├── templates/                 # 文档模板 (18个)
│   ├── prd.md               # PRD模板
│   ├── user-story.md        # 用户故事模板
│   ├── feature-list.md      # 功能清单模板
│   ├── tech-design.md       # 技术方案模板
│   ├── architecture-doc.md  # 架构文档模板
│   ├── db-design.md         # 数据库设计模板
│   ├── test-plan.md         # 测试计划模板
│   ├── test-cases.md        # 测试用例模板
│   ├── test-report.md       # 测试报告模板
│   ├── deployment.md        # 部署文档模板
│   ├── security-checklist.md # 安全检查清单
│   ├── code-review-checklist.md # 代码评审清单
│   ├── ui-design-spec.md    # UI设计规范 (v2.0)
│   ├── project-plan.md      # 项目计划 (v2.0)
│   ├── weekly-report.md     # 周报模板 (v2.0)
│   ├── risk-management.md   # 风险管理 (v2.0)
│   ├── change-request.md    # 变更请求 (v2.0)
│   └── meeting-notes.md     # 会议纪要 (v2.0)
├── package.json              # 插件配置
├── settings.json             # 全局设置
├── mcp.json                  # MCP工具配置
└── README.md                 # 说明文档
```

## 工作流引擎 (v2.0)

### 项目状态管理

项目状态基于 JSON Schema 定义，存储在 `docs/ccteam/project-status.json`：

```json
{
  "projectInfo": { "name": "...", "type": "web-app" },
  "currentStage": { "id": 1, "name": "requirements", "status": "in_progress" },
  "stages": [ ... ],
  "team": { "members": [ ... ] },
  "issues": [ ... ],
  "metrics": { "completionRate": 0.3, "reviewPassRate": 0.95 }
}
```

### 阶段控制命令

```bash
# 开始当前阶段
/ccteam-stage start

# 完成当前阶段（触发评审门禁）
/ccteam-stage complete

# 跳过当前阶段（需要权限）
/ccteam-stage skip --reason "原因说明"

# 回滚到上一阶段
/ccteam-stage rollback

# 查看当前状态
/ccteam-stage status
```

### 评审门禁

每个阶段完成时自动触发评审，支持三种通过模式：
- **single**: 单人评审通过即可
- **majority**: 2/3多数通过
- **unanimous**: 全票通过

## 智能体协作 (v2.0)

### 权限矩阵

15个角色对22种文档类型的权限控制：
- **C** (Create): 创建权限
- **R** (Read): 读取权限
- **U** (Update): 更新权限
- **D** (Delete): 删除权限
- **V** (Review): 评审权限
- **A** (Approve): 审批权限

### 协作规则

- **角色边界**：明确定义各角色的职责边界和决策权限
- **冲突解决**：3级冲突处理流程（直接沟通→TL协调→PMO裁决）
- **紧急处理**：P0问题的权限提升和快速响应机制

### 场景指南

内置6大通用场景处理指南：
1. 拒绝不合理请求
2. 技术债务处理
3. 安全漏洞响应
4. 需求变更管理
5. 紧急Bug修复
6. 技术选型决策

## 开发流程

### 阶段1：需求分析
- 负责角色：产品经理
- 产出：PRD文档、用户故事、功能清单
- 技能：analyze-requirement、write-prd、write-user-story

### 阶段2：技术评审
- 负责角色：技术负责人、架构师
- 产出：技术方案、架构文档、技术选型
- 技能：system-design、tech-selection

### 阶段3：UI/UX设计
- 负责角色：UI设计师、UX设计师
- 产出：交互原型、视觉规范、设计稿

### 阶段4：数据库设计
- 负责角色：DBA、后端工程师
- 产出：数据库设计、ER图、数据字典

### 阶段5：API设计
- 负责角色：API工程师
- 产出：API文档、接口规范

### 阶段6-7：前后端开发（并行）
- 负责角色：前端工程师、后端工程师
- 产出：前端代码、后端代码
- 技能：code-implementation、code-review

### 阶段8：测试
- 负责角色：QA工程师
- 产出：测试计划、测试用例、测试报告
- 技能：test-planning、test-execution

### 阶段9：部署
- 负责角色：DevOps工程师
- 产出：部署文档、CI/CD配置

### 阶段10：文档归档
- 负责角色：文档工程师
- 产出：用户手册、开发文档、项目总结

## 技术栈支持

### 前端
- React / Vue3 / Next.js
- React Native / Flutter
- 微信小程序 / Taro / uni-app

### 后端
- Node.js (Express/NestJS/Fastify)
- Python (FastAPI/Django)
- Java (Spring Boot)
- Go (Gin/Echo)

### 数据库
- PostgreSQL / MySQL
- Redis
- MongoDB

### 部署
- Docker / Docker Compose
- GitHub Actions / GitLab CI
- Nginx

## 配置说明

### settings.json

```json
{
  "language": {
    "documentation": "zh-CN",
    "code": "en"
  },
  "workflow": {
    "autoTransition": true,
    "requireReview": true,
    "reviewMode": "single"
  }
}
```

### MCP工具配置

在 `mcp.json` 中配置各MCP工具的连接信息。

## 输出位置

所有项目文档输出到项目的 `docs/ccteam/` 目录：

```
docs/ccteam/
├── 01-requirements/    # 需求文档
├── 02-architecture/    # 架构文档
├── 03-design/          # 设计文档
├── 04-database/        # 数据库文档
├── 05-api/             # API文档
├── 06-frontend/        # 前端文档
├── 07-backend/         # 后端文档
├── 08-testing/         # 测试文档
├── 09-deployment/      # 部署文档
├── 10-docs/            # 归档文档
└── project-status.json # 项目状态
```

## 版本历史

### v2.0.0 (2026-01-29)
- 新增：完整工作流引擎（状态机模式）
- 新增：JSON Schema 项目状态管理
- 新增：智能体权限矩阵和协作规则
- 新增：场景行为指南（通用+角色特定）
- 新增：评审门禁机制
- 新增：9个核心技能定义
- 新增：6个文档模板
- 改进：智能体行为指导增强

### v1.0.0
- 初始版本
- 15个专业角色智能体
- 10阶段开发流程
- 12个文档模板
- 14个MCP工具集成

## 许可证

MIT License

## 贡献

欢迎提交Issue和Pull Request！
