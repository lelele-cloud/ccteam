# 项目状态结构

## project-status.json 结构

```json
{
  "projectName": "项目名称",
  "projectType": "new-project",
  "description": "项目描述",
  "targetUsers": "目标用户",
  "techStack": {
    "frontend": "Next.js + TypeScript",
    "backend": "Node.js + Express",
    "database": "PostgreSQL"
  },
  "createdAt": "2026-01-30T00:00:00Z",
  "updatedAt": "2026-01-30T00:00:00Z",

  "currentStage": {
    "id": 1,
    "name": "requirements",
    "status": "in-progress",
    "startedAt": "2026-01-30T00:00:00Z",
    "assignedAgent": "product-manager"
  },

  "stages": [
    { "id": 1, "name": "requirements", "displayName": "需求分析", "status": "in-progress" },
    { "id": 2, "name": "tech-review", "displayName": "技术评审", "status": "pending" },
    { "id": 3, "name": "ui-ux-design", "displayName": "UI/UX设计", "status": "pending" },
    { "id": 4, "name": "database-design", "displayName": "数据库设计", "status": "pending" },
    { "id": 5, "name": "api-design", "displayName": "API设计", "status": "pending" },
    { "id": 6, "name": "frontend-dev", "displayName": "前端开发", "status": "pending" },
    { "id": 7, "name": "backend-dev", "displayName": "后端开发", "status": "pending" },
    { "id": 8, "name": "testing", "displayName": "测试", "status": "pending" },
    { "id": 9, "name": "deployment", "displayName": "部署上线", "status": "pending" },
    { "id": 10, "name": "documentation", "displayName": "文档归档", "status": "pending" }
  ],

  "issues": [],
  "metrics": {
    "totalStages": 10,
    "completedStages": 0,
    "skippedStages": 0,
    "reviewRejections": 0
  }
}
```

## 状态值说明

| 状态 | 说明 |
|------|------|
| pending | 待开始 |
| in-progress | 进行中 |
| review | 评审中 |
| completed | 已完成 |
| rejected | 评审未通过 |
| skipped | 已跳过 |
