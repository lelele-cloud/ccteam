# CCTeam - 互联网大厂开发团队

将 Claude Code 转变为一支完整的互联网大厂开发团队，包含15个专业角色智能体，遵循标准软件开发流程。

## 特性

- **15个专业角色**：产品经理、项目经理、技术负责人、架构师、UI设计师、UX设计师、前端工程师、后端工程师、API工程师、QA工程师、DevOps工程师、DBA、安全工程师、文档工程师、代码审查员
- **10阶段开发流程**：需求分析 → 技术评审 → UI/UX设计 → 数据库设计 → API设计 → 前端开发 → 后端开发 → 测试 → 部署 → 文档归档
- **193+技能库**：涵盖需求、架构、设计、开发、测试、运维等各个领域
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
│   └── code-reviewer/        # 代码审查员
├── skills/                    # 技能定义
│   ├── entry/                # 入口技能
│   └── agents/               # 角色快捷技能
├── workflows/                 # 工作流定义
│   ├── engine.md             # 流程引擎
│   └── stages/               # 10个阶段定义
├── templates/                 # 文档模板
│   ├── prd.md               # PRD模板
│   ├── user-story.md        # 用户故事模板
│   ├── tech-design.md       # 技术方案模板
│   ├── architecture-doc.md  # 架构文档模板
│   ├── db-design.md         # 数据库设计模板
│   ├── api-spec.yaml        # API规范模板
│   ├── test-plan.md         # 测试计划模板
│   ├── test-cases.md        # 测试用例模板
│   ├── test-report.md       # 测试报告模板
│   ├── deployment.md        # 部署文档模板
│   ├── security-checklist.md # 安全检查清单
│   └── code-review-checklist.md # 代码评审清单
├── package.json              # 插件配置
├── settings.json             # 全局设置
├── mcp.json                  # MCP工具配置
└── README.md                 # 说明文档
```

## 开发流程

### 阶段1：需求分析
- 负责角色：产品经理
- 产出：PRD文档、用户故事、功能清单

### 阶段2：技术评审
- 负责角色：技术负责人、架构师
- 产出：技术方案、架构文档、技术选型

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

### 阶段8：测试
- 负责角色：QA工程师
- 产出：测试计划、测试用例、测试报告

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
    "requireReview": true
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

## 许可证

MIT License

## 贡献

欢迎提交Issue和Pull Request！
