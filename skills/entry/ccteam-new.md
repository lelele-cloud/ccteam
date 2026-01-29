# CCTeam - 新项目开发

## 概述

启动一个全新项目的完整开发流程，从需求分析到部署上线。

## 使用方法

当用户调用 `/ccteam-new` 时：

### 1. 收集项目信息

询问用户以下信息：

```
请提供项目基本信息：

1. 项目名称：
2. 一句话描述：
3. 目标用户是谁：
4. 核心功能（3-5个）：
5. 项目类型：
   - [ ] Web应用
   - [ ] 移动端App
   - [ ] 小程序
   - [ ] 多端（Web + 移动端）
```

### 2. 技术栈确认

根据项目类型推荐技术栈：

**Web全栈推荐**：
- 前端：Next.js + TypeScript + Tailwind CSS
- 后端：Node.js + Express + Prisma
- 数据库：PostgreSQL + Redis

**移动端推荐**：
- React Native + TypeScript
- 或 Flutter + Dart

**小程序推荐**：
- Taro + React + TypeScript

### 3. 初始化项目

1. 创建 `docs/ccteam/` 目录结构
2. 初始化 `project-status.json`
3. 记录技术栈选择

#### 3.5 创建项目状态文件

调用流程引擎初始化项目状态：

```javascript
// 调用 workflow/engine.md 中的 initializeProject
const projectStatus = {
  projectName: userInput.projectName,
  projectType: "new-project",
  description: userInput.description,
  targetUsers: userInput.targetUsers,
  techStack: selectedTechStack,
  createdAt: new Date().toISOString(),
  updatedAt: new Date().toISOString(),

  currentStage: {
    id: 1,
    name: "requirements",
    status: "pending",
    startedAt: null,
    assignedAgent: null
  },

  stages: [
    { id: 1, name: "requirements", displayName: "需求分析", status: "pending" },
    { id: 2, name: "tech-review", displayName: "技术评审", status: "pending" },
    { id: 3, name: "ui-ux-design", displayName: "UI/UX设计", status: "pending" },
    { id: 4, name: "database-design", displayName: "数据库设计", status: "pending" },
    { id: 5, name: "api-design", displayName: "API设计", status: "pending" },
    { id: 6, name: "frontend-dev", displayName: "前端开发", status: "pending" },
    { id: 7, name: "backend-dev", displayName: "后端开发", status: "pending" },
    { id: 8, name: "testing", displayName: "测试", status: "pending" },
    { id: 9, name: "deployment", displayName: "部署上线", status: "pending" },
    { id: 10, name: "documentation", displayName: "文档归档", status: "pending" }
  ],

  issues: [],
  metrics: {
    totalStages: 10,
    completedStages: 0,
    skippedStages: 0,
    reviewRejections: 0,
    estimatedCompletion: null
  }
};

// 保存到 docs/ccteam/project-status.json
saveStatus(projectStatus);

// 记录日志到 docs/ccteam/workflow.log
logEvent("PROJECT_INITIALIZED", { projectName: userInput.projectName });
```

#### 3.6 启动第一阶段

自动启动需求分析阶段：

```javascript
// 调用 workflow/engine.md 中的 startStage
startStage(1); // 启动需求分析阶段

// 更新状态
updateStageStatus(1, "in-progress");

// 加载产品经理智能体
loadAgent("ccteam-pm");

// 记录日志
logEvent("STAGE_STARTED", {
  stageId: 1,
  agent: "product-manager",
  timestamp: new Date().toISOString()
});
```

**提示用户**：

```
项目 "{{projectName}}" 创建成功！

当前阶段：需求分析
负责角色：产品经理

接下来，产品经理将开始分析您的需求并生成PRD文档。

提示：
- 使用 /ccteam-status 查看项目状态
- 使用 /ccteam-resume 恢复中断的项目
```

### 4. 启动开发流程

按顺序执行10个阶段：

1. **需求分析** - 产品经理
2. **技术评审** - 技术负责人 + 架构师
3. **UI/UX设计** - UI设计师 + UX设计师
4. **数据库设计** - DBA + 后端工程师
5. **API设计** - API工程师
6. **前端开发** - 前端工程师
7. **后端开发** - 后端工程师（与前端并行）
8. **测试** - 测试工程师 + 安全工程师
9. **部署上线** - DevOps工程师
10. **文档归档** - 技术文档工程师

## 阶段流转

- 每阶段完成后进行评审
- 评审通过自动进入下一阶段
- 评审不通过则打回修改

## 依赖技能

本技能依赖以下流程引擎技能：

| 技能 | 路径 | 用途 |
|------|------|------|
| 流程引擎 | `skills/workflow/engine.md` | 初始化项目状态、启动阶段、管理流程 |
| 阶段控制 | `skills/workflow/stage-control.md` | 阶段启动、完成、跳过、回退 |
| 评审关卡 | `skills/workflow/review-gate.md` | 评审流程、检查清单、通过/拒绝处理 |

### 调用关系

```
/ccteam-new
    │
    ├── 收集项目信息
    ├── 确认技术栈
    │
    └── 初始化项目
        │
        ├── engine.initializeProject()  ← 创建项目状态
        │       │
        │       └── 生成 project-status.json
        │
        └── engine.startStage(1)  ← 启动第一阶段
                │
                ├── 更新阶段状态为 in-progress
                ├── 加载 ccteam-pm 智能体
                └── 记录 STAGE_STARTED 日志
```
