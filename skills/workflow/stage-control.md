# 阶段控制 (Stage Control)

## 概述

阶段控制技能提供手动干预开发流程的能力，支持启动、完成、跳过、回退阶段等操作。

## 可用命令

### 1. 启动阶段

```
/ccteam-stage start <stage-name>
```

**功能**：手动启动指定阶段

**参数**：
- `stage-name`：阶段名称或编号

**示例**：
```
/ccteam-stage start requirements
/ccteam-stage start 3
```

**执行逻辑**：
```javascript
function startStage(stageName) {
  const stage = resolveStage(stageName);

  // 1. 权限检查
  if (!hasPermission(currentUser, "start-stage")) {
    throw new PermissionError("您没有启动阶段的权限");
  }

  // 2. 状态检查
  if (stage.status !== "pending" && stage.status !== "rejected") {
    throw new StateError(`阶段${stage.name}当前状态为${stage.status}，无法启动`);
  }

  // 3. 依赖检查
  const blockers = getBlockingStages(stage.id);
  if (blockers.length > 0) {
    throw new BlockedError(`阶段被以下阶段阻塞: ${blockers.join(", ")}`);
  }

  // 4. 启动阶段
  engine.startStage(stage.id);

  return {
    success: true,
    message: `阶段 "${stage.displayName}" 已启动`,
    assignedAgent: stage.agent
  };
}
```

---

### 2. 完成阶段

```
/ccteam-stage complete [--skip-review]
```

**功能**：标记当前阶段为完成，触发评审流程

**选项**：
- `--skip-review`：跳过评审直接进入下一阶段（需特殊权限）

**执行逻辑**：
```javascript
function completeStage(options = {}) {
  const currentStage = getCurrentStage();

  // 1. 权限检查
  if (!hasPermission(currentUser, "complete-stage")) {
    throw new PermissionError("您没有完成阶段的权限");
  }

  // 2. 状态检查
  if (currentStage.status !== "in-progress") {
    throw new StateError("当前阶段不在进行中状态");
  }

  // 3. 产出物验证
  const outputs = collectStageOutputs(currentStage.id);
  const validation = validateOutputs(currentStage.id, outputs);

  if (!validation.passed) {
    return {
      success: false,
      errors: validation.errors,
      message: "产出物验证失败，请补充以下内容",
      missingOutputs: validation.missing
    };
  }

  // 4. 处理跳过评审
  if (options.skipReview) {
    if (!hasPermission(currentUser, "skip-review")) {
      throw new PermissionError("您没有跳过评审的权限");
    }
    engine.handleReviewPassed(currentStage.id, {
      comments: "评审已跳过",
      reviewer: "system",
      skipped: true
    });
    return {
      success: true,
      message: `阶段 "${currentStage.displayName}" 已完成（已跳过评审）`
    };
  }

  // 5. 进入评审
  engine.completeStage(currentStage.id, outputs);

  return {
    success: true,
    message: `阶段 "${currentStage.displayName}" 已完成，等待评审`,
    reviewer: currentStage.reviewer
  };
}
```

---

### 3. 跳过阶段

```
/ccteam-stage skip <stage-name> --reason "<reason>"
```

**功能**：跳过指定阶段

**参数**：
- `stage-name`：阶段名称或编号
- `--reason`：跳过原因（必填）

**示例**：
```
/ccteam-stage skip ui-ux-design --reason "本次为后端接口更新，无需UI变更"
/ccteam-stage skip 4 --reason "使用现有数据库结构"
```

**可跳过的阶段**：

| 阶段 | 说明 |
|------|------|
| `ui-ux-design` (3) | 纯后端项目或无UI变更时 |
| `database-design` (4) | 使用现有数据结构时 |
| `deployment` (9) | 仅本地开发或测试时 |

**执行逻辑**：
```javascript
function skipStage(stageName, reason) {
  const stage = resolveStage(stageName);

  // 1. 权限检查
  if (!hasPermission(currentUser, "skip-stage")) {
    throw new PermissionError("您没有跳过阶段的权限");
  }

  // 2. 可跳过检查
  const skippableStages = ["ui-ux-design", "database-design", "deployment"];
  if (!skippableStages.includes(stage.name)) {
    throw new ValidationError(`阶段 "${stage.displayName}" 不可跳过`);
  }

  // 3. 原因验证
  if (!reason || reason.trim().length < 10) {
    throw new ValidationError("请提供有效的跳过原因（至少10个字符）");
  }

  // 4. 执行跳过
  updateStageStatus(stage.id, "skipped");
  recordSkipReason(stage.id, reason);

  // 5. 更新指标
  incrementSkippedStages();

  // 6. 记录日志
  logEvent("STAGE_SKIPPED", { stageId: stage.id, reason });

  // 7. 解锁下一阶段
  const nextStages = getNextStages(stage.id);
  nextStages.forEach(s => unlockStage(s.id));

  return {
    success: true,
    message: `阶段 "${stage.displayName}" 已跳过`,
    reason: reason,
    nextStage: nextStages[0]?.displayName
  };
}
```

---

### 4. 回退阶段

```
/ccteam-stage rollback [--to <stage-name>]
```

**功能**：回退到指定阶段或上一阶段

**选项**：
- `--to`：指定回退到的阶段（默认回退到上一阶段）

**示例**：
```
/ccteam-stage rollback
/ccteam-stage rollback --to api-design
```

**执行逻辑**：
```javascript
function rollbackStage(options = {}) {
  const currentStage = getCurrentStage();

  // 1. 权限检查
  if (!hasPermission(currentUser, "rollback-stage")) {
    throw new PermissionError("您没有回退阶段的权限");
  }

  // 2. 确定目标阶段
  let targetStage;
  if (options.to) {
    targetStage = resolveStage(options.to);
    if (targetStage.id >= currentStage.id) {
      throw new ValidationError("只能回退到之前的阶段");
    }
  } else {
    targetStage = getPreviousStage(currentStage.id);
  }

  // 3. 创建回退记录
  const rollbackRecord = {
    fromStage: currentStage.id,
    toStage: targetStage.id,
    reason: options.reason || "手动回退",
    timestamp: new Date().toISOString()
  };

  // 4. 重置中间阶段状态
  for (let i = currentStage.id; i > targetStage.id; i--) {
    updateStageStatus(i, "pending");
    clearStageOutputs(i);
  }

  // 5. 设置目标阶段为 in-progress
  updateStageStatus(targetStage.id, "in-progress");
  setCurrentStage(targetStage.id);

  // 6. 记录日志
  logEvent("STAGE_ROLLBACK", rollbackRecord);

  return {
    success: true,
    message: `已回退到阶段 "${targetStage.displayName}"`,
    rollbackRecord
  };
}
```

---

### 5. 查看状态

```
/ccteam-stage status
```

**功能**：查看当前项目流程状态

**输出示例**：
```
========== CCTeam 项目状态 ==========

项目名称: 示例项目
项目类型: 新项目
创建时间: 2026-01-28 10:00:00

当前阶段: [6] 前端开发 (进行中)
负责人: frontend-engineer

阶段进度:
  [1] 需求分析      ✅ 已完成
  [2] 技术评审      ✅ 已完成
  [3] UI/UX设计     ✅ 已完成
  [4] 数据库设计    ⏭️ 已跳过
  [5] API设计       ✅ 已完成
  [6] 前端开发      🔄 进行中  ← 当前
  [7] 后端开发      🔄 进行中  (并行)
  [8] 测试          ⏳ 等待中
  [9] 部署上线      ⏳ 等待中
  [10] 文档归档     ⏳ 等待中

统计:
  - 总阶段数: 10
  - 已完成: 5
  - 已跳过: 1
  - 评审退回: 0
  - 预计完成: 2026-01-28 18:00

====================================
```

---

## 权限控制

### 角色权限矩阵

| 命令 | PM | PMO | TL | Arch | 开发人员 | CR |
|------|:--:|:---:|:--:|:----:|:--------:|:--:|
| `start` | - | ✓ | ✓ | - | 仅负责阶段 | - |
| `complete` | ✓ | ✓ | ✓ | ✓ | 仅负责阶段 | - |
| `skip` | - | ✓ | ✓ | - | - | - |
| `rollback` | - | ✓ | ✓ | - | - | - |
| `status` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| `--skip-review` | - | - | ✓ | - | - | - |

### 权限说明

- **PM (产品经理)**：可完成自己负责的阶段，查看状态
- **PMO (项目经理)**：拥有所有阶段控制权限
- **TL (技术负责人)**：拥有所有阶段控制权限，可跳过评审
- **Arch (架构师)**：可完成自己负责的阶段，查看状态
- **开发人员**：只能完成自己负责的阶段
- **CR (代码审查员)**：只能查看状态

---

## 可跳过的阶段

| 阶段 | 跳过场景 |
|------|----------|
| `ui-ux-design` | 纯后端项目、API项目、无界面变更的功能迭代 |
| `database-design` | 使用现有数据结构、NoSQL项目、前端独立项目 |
| `deployment` | 本地开发环境、测试验证阶段、文档类项目 |

**不可跳过的阶段**：

- 需求分析 - 项目的基础
- 技术评审 - 确保技术可行性
- API设计 - 前后端协作的契约
- 前端/后端开发 - 核心实现
- 测试 - 质量保证
- 文档归档 - 知识传承

---

## 错误处理

### 常见错误码

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| `PERMISSION_DENIED` | 权限不足 | 联系项目经理或技术负责人 |
| `STAGE_BLOCKED` | 阶段被阻塞 | 先完成依赖阶段 |
| `INVALID_STATE` | 状态不正确 | 检查当前阶段状态 |
| `OUTPUT_MISSING` | 产出物缺失 | 补充必需的产出物 |
| `NOT_SKIPPABLE` | 阶段不可跳过 | 该阶段必须执行 |
| `INVALID_REASON` | 原因无效 | 提供详细的跳过原因 |
