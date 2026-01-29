# 流程引擎 (Workflow Engine)

## 概述

流程引擎是 CCTeam 的核心控制中心，负责管理项目开发的10个阶段，实现自动流转、评审关卡和状态持久化。它是连接入口技能、智能体和阶段定义的枢纽。

## 触发条件

流程引擎在以下场景被触发：

| 触发点 | 说明 |
|--------|------|
| `/ccteam-new` | 新项目启动，初始化完整状态 |
| `/ccteam-resume` | 恢复项目，加载现有状态 |
| `/ccteam-stage` | 阶段控制命令 |
| 评审完成 | 处理评审结果，决定流转方向 |

## 核心操作

### 1. 初始化项目状态

当新项目启动时，生成完整的 `project-status.json`：

```javascript
function initializeProject(projectInfo) {
  const status = {
    projectName: projectInfo.name,
    projectType: projectInfo.type, // "new-project" | "new-feature"
    techStack: projectInfo.techStack,
    createdAt: new Date().toISOString(),
    updatedAt: new Date().toISOString(),

    currentStage: {
      id: 1,
      name: "requirements",
      status: "pending",
      startedAt: null,
      assignedAgent: null
    },

    stages: generateStageList(),
    issues: [],
    metrics: {
      totalStages: 10,
      completedStages: 0,
      skippedStages: 0,
      reviewRejections: 0,
      estimatedCompletion: null
    }
  };

  saveStatus(status, "docs/ccteam/project-status.json");
  logEvent("PROJECT_INITIALIZED", { projectName: projectInfo.name });
}
```

### 2. 启动阶段

当进入新阶段时执行：

```javascript
function startStage(stageId) {
  // 1. 检查前置条件
  if (!checkBlockers(stageId)) {
    throw new Error("存在阻塞的依赖阶段");
  }

  // 2. 更新状态为 in-progress
  updateStageStatus(stageId, "in-progress");

  // 3. 加载对应智能体
  const agent = loadAgent(getStageAgent(stageId));

  // 4. 创建阶段工作目录
  createStageDirectory(stageId);

  // 5. 记录日志
  logEvent("STAGE_STARTED", {
    stageId,
    agent: agent.name,
    timestamp: new Date().toISOString()
  });

  // 6. 触发 onStageStart 钩子
  triggerHook("onStageStart", { stageId, agent });
}
```

### 3. 完成阶段

当负责人宣告完成时执行：

```javascript
function completeStage(stageId, outputs) {
  // 1. 验证产出物
  const validation = validateOutputs(stageId, outputs);
  if (!validation.passed) {
    return {
      success: false,
      errors: validation.errors
    };
  }

  // 2. 更新状态为 review
  updateStageStatus(stageId, "review");

  // 3. 记录产出物
  recordOutputs(stageId, outputs);

  // 4. 触发评审
  initiateReview(stageId);

  // 5. 记录日志
  logEvent("STAGE_COMPLETED", {
    stageId,
    outputs,
    timestamp: new Date().toISOString()
  });
}
```

### 4. 评审通过处理

```javascript
function handleReviewPassed(stageId, reviewResult) {
  // 1. 记录评审结果
  recordReviewResult(stageId, {
    passed: true,
    comments: reviewResult.comments,
    reviewer: reviewResult.reviewer,
    reviewedAt: new Date().toISOString()
  });

  // 2. 标记阶段完成
  updateStageStatus(stageId, "completed");

  // 3. 更新指标
  incrementCompletedStages();

  // 4. 解锁下一阶段
  const nextStages = getNextStages(stageId);
  nextStages.forEach(stage => unlockStage(stage.id));

  // 5. 自动启动下一阶段
  if (nextStages.length === 1) {
    startStage(nextStages[0].id);
  } else if (nextStages.length > 1) {
    // 并行阶段处理
    handleParallelStages(nextStages);
  }

  // 6. 触发 onReviewPass 钩子
  triggerHook("onReviewPass", { stageId, reviewResult });

  // 7. 记录日志
  logEvent("REVIEW_PASSED", { stageId, reviewer: reviewResult.reviewer });
}
```

### 5. 评审不通过处理

```javascript
function handleReviewRejected(stageId, reviewResult) {
  // 1. 创建 Issue 记录问题
  const issue = createIssue({
    stageId,
    type: "review-rejection",
    description: reviewResult.comments,
    suggestions: reviewResult.suggestions,
    createdBy: reviewResult.reviewer,
    createdAt: new Date().toISOString(),
    status: "open"
  });

  // 2. 记录评审结果
  recordReviewResult(stageId, {
    passed: false,
    comments: reviewResult.comments,
    issueId: issue.id,
    reviewer: reviewResult.reviewer,
    reviewedAt: new Date().toISOString()
  });

  // 3. 更新状态为 rejected
  updateStageStatus(stageId, "rejected");

  // 4. 更新指标
  incrementReviewRejections();

  // 5. 回退给负责人
  const agent = getStageAgent(stageId);
  notifyAgent(agent, {
    type: "review-rejected",
    issue: issue,
    action: "请根据评审意见修改后重新提交"
  });

  // 6. 触发 onReviewReject 钩子
  triggerHook("onReviewReject", { stageId, issue, reviewResult });

  // 7. 记录日志
  logEvent("REVIEW_REJECTED", { stageId, issueId: issue.id });
}
```

### 6. 并行阶段处理

阶段6（前端开发）和阶段7（后端开发）支持并行执行：

```javascript
function handleParallelStages(stages) {
  // 阶段5（API设计）完成后，同时启动阶段6和7
  if (stages.some(s => s.id === 6) && stages.some(s => s.id === 7)) {
    logEvent("PARALLEL_STAGES_START", { stages: [6, 7] });

    // 同时启动两个阶段
    startStage(6);
    startStage(7);

    // 设置等待条件：两者都完成后才能启动阶段8
    setWaitCondition(8, {
      waitFor: [6, 7],
      condition: "ALL_COMPLETED"
    });
  }
}

function checkParallelCompletion(stageId) {
  // 当阶段6或7完成时检查
  if (stageId === 6 || stageId === 7) {
    const stage6 = getStageStatus(6);
    const stage7 = getStageStatus(7);

    if (stage6.status === "completed" && stage7.status === "completed") {
      logEvent("PARALLEL_STAGES_COMPLETED", { stages: [6, 7] });
      startStage(8); // 启动测试阶段
    }
  }
}
```

## 状态文件位置

| 文件 | 路径 | 用途 |
|------|------|------|
| 项目状态 | `docs/ccteam/project-status.json` | 持久化项目状态 |
| 工作流日志 | `docs/ccteam/workflow.log` | 记录所有流程事件 |
| Issues | `docs/ccteam/issues/` | 存放评审问题 |

## 日志记录

所有流程事件记录到 `docs/ccteam/workflow.log`：

```
[2026-01-28T10:00:00Z] PROJECT_INITIALIZED - 项目"示例项目"已初始化
[2026-01-28T10:00:01Z] STAGE_STARTED - 阶段1(需求分析)已启动，负责人: product-manager
[2026-01-28T10:30:00Z] STAGE_COMPLETED - 阶段1(需求分析)已完成，等待评审
[2026-01-28T10:30:05Z] REVIEW_STARTED - 阶段1评审已开始，评审人: tech-lead
[2026-01-28T10:35:00Z] REVIEW_PASSED - 阶段1评审通过
[2026-01-28T10:35:01Z] STAGE_STARTED - 阶段2(技术评审)已启动
...
```

日志格式：

```javascript
function logEvent(eventType, data) {
  const logEntry = {
    timestamp: new Date().toISOString(),
    type: eventType,
    data: data,
    message: formatEventMessage(eventType, data)
  };

  appendToFile("docs/ccteam/workflow.log", formatLogLine(logEntry));
}
```

## 钩子函数

流程引擎提供以下钩子点，可在 `settings.json` 中配置：

| 钩子 | 触发时机 | 参数 |
|------|----------|------|
| `onStageStart` | 阶段开始 | `{ stageId, agent }` |
| `onStageComplete` | 阶段完成 | `{ stageId, outputs }` |
| `onReviewStart` | 评审开始 | `{ stageId, reviewer }` |
| `onReviewPass` | 评审通过 | `{ stageId, reviewResult }` |
| `onReviewReject` | 评审不通过 | `{ stageId, issue, reviewResult }` |
| `onProjectComplete` | 项目完成 | `{ projectName, metrics }` |

## 异常处理

### 阶段阻塞

```javascript
function handleStageBlocked(stageId, blockers) {
  logEvent("STAGE_BLOCKED", { stageId, blockers });

  // 通知相关角色
  blockers.forEach(blockerId => {
    const agent = getStageAgent(blockerId);
    notifyAgent(agent, {
      type: "blocking-dependency",
      message: `您负责的阶段${blockerId}正在阻塞阶段${stageId}的启动`
    });
  });
}
```

### 超时处理

```javascript
function checkStageTimeout(stageId) {
  const stage = getStageStatus(stageId);
  const elapsed = Date.now() - new Date(stage.startedAt).getTime();
  const timeout = getStageTimeout(stageId);

  if (elapsed > timeout) {
    logEvent("STAGE_TIMEOUT", { stageId, elapsed });
    notifyProjectManager({
      type: "stage-timeout",
      stageId,
      elapsed,
      action: "请检查阶段进度"
    });
  }
}
```

## 与其他技能的协作

| 协作技能 | 协作方式 |
|----------|----------|
| `stage-control.md` | 接收阶段控制命令 |
| `review-gate.md` | 委托评审执行 |
| 入口技能 | 接收初始化请求 |
| 智能体技能 | 分派阶段任务 |
