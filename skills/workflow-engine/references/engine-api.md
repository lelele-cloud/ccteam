# 流程引擎API

## 初始化项目

```javascript
function initializeProject(projectInfo) {
  const status = {
    projectName: projectInfo.name,
    projectType: projectInfo.type,
    techStack: projectInfo.techStack,
    createdAt: new Date().toISOString(),
    currentStage: { id: 1, name: "requirements", status: "pending" },
    stages: generateStageList(),
    issues: [],
    metrics: { totalStages: 10, completedStages: 0 }
  };
  saveStatus(status, "docs/ccteam/project-status.json");
}
```

## 启动阶段

```javascript
function startStage(stageId) {
  if (!checkBlockers(stageId)) throw new Error("存在阻塞依赖");
  updateStageStatus(stageId, "in-progress");
  const agent = loadAgent(getStageAgent(stageId));
  createStageDirectory(stageId);
  logEvent("STAGE_STARTED", { stageId, agent: agent.name });
}
```

## 完成阶段

```javascript
function completeStage(stageId, outputs) {
  const validation = validateOutputs(stageId, outputs);
  if (!validation.passed) return { success: false, errors: validation.errors };
  updateStageStatus(stageId, "review");
  recordOutputs(stageId, outputs);
  initiateReview(stageId);
}
```

## 评审通过处理

```javascript
function handleReviewPassed(stageId, reviewResult) {
  recordReviewResult(stageId, { passed: true, comments: reviewResult.comments });
  updateStageStatus(stageId, "completed");
  incrementCompletedStages();
  const nextStages = getNextStages(stageId);
  nextStages.forEach(stage => unlockStage(stage.id));
  if (nextStages.length === 1) startStage(nextStages[0].id);
}
```

## 评审不通过处理

```javascript
function handleReviewRejected(stageId, reviewResult) {
  const issue = createIssue({
    stageId,
    type: "review-rejection",
    description: reviewResult.comments,
    status: "open"
  });
  updateStageStatus(stageId, "rejected");
  notifyAgent(getStageAgent(stageId), { type: "review-rejected", issue });
}
```

## 钩子函数

| 钩子 | 触发时机 |
|------|----------|
| `onStageStart` | 阶段开始 |
| `onStageComplete` | 阶段完成 |
| `onReviewPass` | 评审通过 |
| `onReviewReject` | 评审不通过 |
| `onProjectComplete` | 项目完成 |
