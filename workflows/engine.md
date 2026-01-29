# CCTeam 流程引擎

## 概述

流程引擎负责管理项目开发的10个阶段，实现自动流转、评审关卡和状态持久化。

## 核心功能

### 1. 阶段管理

```
阶段状态：pending → in-progress → review → completed/rejected
```

### 2. 自动流转

当前阶段评审通过后，自动触发下一阶段：

```javascript
function autoTransition(currentStage) {
  if (currentStage.reviewResult.passed) {
    markCompleted(currentStage);
    unlockNextStages(currentStage);
    startNextStage();
  } else {
    createIssue(currentStage.reviewResult);
    rollbackToAgent(currentStage);
  }
}
```

### 3. 评审关卡

每个阶段完成后需要评审：

| 阶段 | 评审方 | 评审内容 |
|------|--------|----------|
| 需求分析 | 技术负责人 | PRD完整性、可行性 |
| 技术评审 | 全员 | 架构合理性、技术选型 |
| UI/UX设计 | 产品经理、前端 | 设计稿质量、交互体验 |
| 数据库设计 | 架构师 | 表结构、索引设计 |
| API设计 | 前端、后端 | 接口规范、完整性 |
| 前端开发 | 代码审查员 | 代码质量、规范 |
| 后端开发 | 代码审查员 | 代码质量、规范 |
| 测试 | 技术负责人 | 测试覆盖、质量 |
| 部署 | 技术负责人 | 部署配置、监控 |
| 文档归档 | 产品经理 | 文档完整性 |

### 4. 状态持久化

所有状态保存到 `docs/ccteam/project-status.json`：

```json
{
  "currentStage": {
    "id": 1,
    "name": "requirements",
    "status": "in-progress"
  },
  "stages": [...],
  "issues": [...],
  "metrics": {...}
}
```

## 流程控制命令

### 启动阶段
```
/ccteam-stage start <stage-name>
```

### 完成阶段
```
/ccteam-stage complete
```

### 跳过阶段
```
/ccteam-stage skip <stage-name>
```

### 回退阶段
```
/ccteam-stage rollback
```

## 并行执行

前端开发(6)和后端开发(7)可以并行执行：

```
阶段5(API设计) ──┬──→ 阶段6(前端开发) ──┬──→ 阶段8(测试)
                 │                       │
                 └──→ 阶段7(后端开发) ──┘
```

等待两者都完成后，才能进入测试阶段。

## 异常处理

### 评审不通过

1. 创建Issue记录问题
2. 打回给上游角色
3. 附带修改意见
4. 修改完成后重新评审

### 阶段阻塞

1. 检查依赖阶段状态
2. 通知相关角色
3. 等待依赖完成

## 钩子函数

### onStageStart
阶段开始时触发，加载对应智能体。

### onStageComplete
阶段完成时触发，生成产出物。

### onReviewPass
评审通过时触发，解锁下一阶段。

### onReviewReject
评审不通过时触发，创建Issue。
