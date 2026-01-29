# 评审关卡 (Review Gate)

## 概述

评审关卡技能管理每个阶段完成后的评审流程，确保产出物质量符合标准。它定义了评审规则、流程和各阶段的检查清单。

## 评审规则

### 1. 启动条件

评审在以下条件满足时自动启动：

```javascript
function canStartReview(stageId) {
  const stage = getStageStatus(stageId);

  // 条件1: 负责人已宣告完成
  const declaredComplete = stage.status === "review";

  // 条件2: 所有必需产出物存在
  const requiredOutputs = getRequiredOutputs(stageId);
  const actualOutputs = getActualOutputs(stageId);
  const outputsExist = requiredOutputs.every(
    req => actualOutputs.some(act => matchOutput(req, act))
  );

  // 条件3: 产出物基本验证通过
  const basicValidation = validateOutputsBasic(stageId);

  return declaredComplete && outputsExist && basicValidation.passed;
}
```

### 2. 通过条件

| 评审类型 | 通过条件 |
|----------|----------|
| 单人评审 | 评审人通过即可 |
| 多人评审 | >= 2/3 评审人通过 |
| 技术委员会评审 | 全员通过 |

```javascript
function isReviewPassed(stageId, reviews) {
  const reviewConfig = getReviewConfig(stageId);
  const totalReviewers = reviews.length;
  const passedCount = reviews.filter(r => r.passed).length;

  switch (reviewConfig.type) {
    case "single":
      return passedCount >= 1;

    case "majority":
      return passedCount >= Math.ceil(totalReviewers * 2 / 3);

    case "unanimous":
      return passedCount === totalReviewers;

    default:
      return passedCount >= 1;
  }
}
```

### 3. 超时处理

| 超时节点 | 时间 | 处理 |
|----------|------|------|
| 提醒 | 24小时 | 发送提醒给评审人 |
| 升级 | 48小时 | 升级到项目经理 |
| 自动处理 | 72小时 | 项目经理介入决策 |

```javascript
function handleReviewTimeout(stageId) {
  const review = getReviewStatus(stageId);
  const elapsed = Date.now() - new Date(review.startedAt).getTime();
  const hours = elapsed / (1000 * 60 * 60);

  if (hours >= 72) {
    // 72小时：升级到项目经理决策
    escalateToProjectManager(stageId, {
      reason: "评审超时72小时",
      options: ["强制通过", "延期", "指定其他评审人"]
    });
  } else if (hours >= 48) {
    // 48小时：升级提醒
    notifyProjectManager(stageId, {
      type: "review-timeout-escalation",
      message: "评审已超时48小时，请介入处理"
    });
  } else if (hours >= 24) {
    // 24小时：提醒评审人
    notifyReviewers(stageId, {
      type: "review-reminder",
      message: "您有一个待处理的评审，请及时处理"
    });
  }
}
```

---

## 评审流程

### 1. 发起评审

当阶段负责人完成工作后：

```javascript
function initiateReview(stageId) {
  const stage = getStageStatus(stageId);

  // 1. 获取评审人
  const reviewers = getReviewers(stageId);

  // 2. 创建评审记录
  const review = {
    id: generateReviewId(),
    stageId: stageId,
    stageName: stage.name,
    reviewers: reviewers.map(r => ({
      agent: r,
      status: "pending",
      result: null
    })),
    outputs: getActualOutputs(stageId),
    checklist: getReviewChecklist(stageId),
    startedAt: new Date().toISOString(),
    deadline: calculateDeadline(24), // 24小时后
    status: "in-progress"
  };

  // 3. 保存评审记录
  saveReview(review);

  // 4. 通知评审人
  reviewers.forEach(reviewer => {
    notifyAgent(reviewer, {
      type: "review-request",
      stageId: stageId,
      stageName: stage.displayName,
      outputs: review.outputs,
      checklist: review.checklist,
      deadline: review.deadline
    });
  });

  // 5. 记录日志
  logEvent("REVIEW_INITIATED", {
    stageId,
    reviewers: reviewers,
    deadline: review.deadline
  });

  return review;
}
```

### 2. 执行评审

评审人按照检查清单逐项审核：

```javascript
function executeReview(reviewId, reviewerAgent) {
  const review = getReview(reviewId);
  const checklist = review.checklist;

  // 显示评审界面
  displayReviewInterface({
    title: `${review.stageName} 阶段评审`,
    outputs: review.outputs,
    checklist: checklist,
    instructions: `
      请逐项检查以下内容：
      1. 阅读所有产出物
      2. 按照检查清单逐项确认
      3. 记录问题和建议
      4. 给出评审结论
    `
  });
}
```

### 3. 反馈格式

评审结果使用标准格式提交：

```json
{
  "reviewId": "review-001",
  "stageId": 1,
  "reviewer": "tech-lead",
  "reviewedAt": "2026-01-28T10:35:00Z",
  "passed": true,
  "checklistResults": [
    { "item": "项目背景描述清晰", "passed": true, "comment": "" },
    { "item": "目标用户定义明确", "passed": true, "comment": "" },
    { "item": "核心功能列表完整", "passed": true, "comment": "" },
    { "item": "功能优先级已排序", "passed": true, "comment": "" },
    { "item": "验收标准可量化", "passed": false, "comment": "部分功能缺少具体的量化指标" }
  ],
  "overallComments": "整体需求文档质量良好，建议补充登录功能的量化验收标准",
  "suggestions": [
    "为登录功能添加响应时间要求（如：< 2秒）",
    "明确密码强度的具体规则"
  ],
  "blockers": []
}
```

### 4. 处理结果

```javascript
function processReviewResult(reviewResult) {
  const review = getReview(reviewResult.reviewId);

  // 1. 更新评审人状态
  updateReviewerStatus(review.id, reviewResult.reviewer, {
    status: "completed",
    result: reviewResult
  });

  // 2. 检查是否所有评审人都已完成
  const allCompleted = review.reviewers.every(r => r.status === "completed");

  if (allCompleted) {
    // 3. 汇总评审结果
    const results = review.reviewers.map(r => r.result);
    const passed = isReviewPassed(review.stageId, results);

    // 4. 调用流程引擎处理
    if (passed) {
      engine.handleReviewPassed(review.stageId, {
        comments: aggregateComments(results),
        reviewer: results.map(r => r.reviewer).join(", "),
        details: results
      });
    } else {
      engine.handleReviewRejected(review.stageId, {
        comments: aggregateComments(results),
        suggestions: aggregateSuggestions(results),
        blockers: aggregateBlockers(results),
        reviewer: results.map(r => r.reviewer).join(", "),
        details: results
      });
    }

    // 5. 更新评审状态
    updateReviewStatus(review.id, passed ? "passed" : "rejected");
  }
}
```

---

## 各阶段评审检查清单

### 阶段1: 需求分析

**评审人**: 技术负责人

```markdown
- [ ] 项目背景描述清晰完整
- [ ] 目标用户群体定义明确
- [ ] 核心功能列表完整（3-5个核心功能）
- [ ] 功能优先级已排序（P0/P1/P2/P3）
- [ ] 每个功能都有验收标准
- [ ] 验收标准可量化、可测试
- [ ] 异常场景已充分考虑
- [ ] 非功能性需求已定义（性能、安全、可用性）
- [ ] 文档格式规范，无遗漏章节
```

---

### 阶段2: 技术评审

**评审人**: 全员

```markdown
- [ ] 技术选型合理，有明确理由
- [ ] 系统架构图清晰完整
- [ ] 模块划分合理，职责明确
- [ ] 技术风险已识别并有应对方案
- [ ] 性能指标已定义（QPS、响应时间等）
- [ ] 扩展性设计已考虑
- [ ] 安全设计已考虑
- [ ] 第三方依赖已评估
- [ ] 开发工作量估算合理
```

---

### 阶段3: UI/UX设计

**评审人**: 产品经理、前端工程师

```markdown
- [ ] 设计稿覆盖所有核心功能页面
- [ ] 交互原型逻辑清晰
- [ ] 视觉规范定义完整（颜色、字体、间距）
- [ ] 响应式设计已考虑
- [ ] 状态切换完整（loading、空状态、错误状态）
- [ ] 无障碍设计已考虑
- [ ] 设计稿与需求文档一致
- [ ] 切图资源标注清晰
```

---

### 阶段4: 数据库设计

**评审人**: 系统架构师

```markdown
- [ ] ER图完整，实体关系清晰
- [ ] 表结构设计合理，无冗余
- [ ] 字段命名规范统一
- [ ] 主键、外键设计正确
- [ ] 索引设计合理，覆盖主要查询场景
- [ ] 数据类型选择恰当
- [ ] 已考虑数据增长和分表策略
- [ ] 数据迁移方案已准备（如适用）
- [ ] 敏感数据加密方案已定义
```

---

### 阶段5: API设计

**评审人**: 前端工程师、后端工程师

```markdown
- [ ] API文档符合OpenAPI规范
- [ ] 接口路径设计RESTful
- [ ] 请求/响应格式定义清晰
- [ ] 错误码定义完整统一
- [ ] 认证授权方式已定义
- [ ] 接口版本控制策略已定义
- [ ] 分页、排序、过滤规范统一
- [ ] 接口覆盖所有功能需求
- [ ] 示例请求和响应已提供
```

---

### 阶段6: 前端开发

**评审人**: 代码审查员

```markdown
- [ ] 代码符合项目编码规范
- [ ] 组件划分合理，可复用性好
- [ ] 状态管理清晰
- [ ] 类型定义完整（TypeScript）
- [ ] 无明显性能问题
- [ ] 错误处理完善
- [ ] 无控制台错误和警告
- [ ] 响应式适配正常
- [ ] 与设计稿还原度 >= 95%
- [ ] 单元测试覆盖核心逻辑
```

---

### 阶段7: 后端开发

**评审人**: 代码审查员

```markdown
- [ ] 代码符合项目编码规范
- [ ] 架构分层清晰（Controller/Service/Repository）
- [ ] 业务逻辑正确完整
- [ ] 数据库操作优化（无N+1问题）
- [ ] 异常处理完善
- [ ] 日志记录规范
- [ ] 接口与API文档一致
- [ ] 无SQL注入等安全漏洞
- [ ] 单元测试覆盖率 >= 70%
- [ ] 并发处理正确
```

---

### 阶段8: 测试

**评审人**: 技术负责人

```markdown
- [ ] 测试计划覆盖所有功能模块
- [ ] 测试用例设计完整
- [ ] 核心功能测试通过率 100%
- [ ] 边界条件和异常测试已覆盖
- [ ] 性能测试达到预期指标
- [ ] 安全测试无高危漏洞
- [ ] 兼容性测试通过
- [ ] 回归测试无遗漏
- [ ] 测试报告格式规范
- [ ] 缺陷已修复或有处理计划
```

---

### 阶段9: 部署上线

**评审人**: 技术负责人

```markdown
- [ ] CI/CD流程配置正确
- [ ] 部署脚本可重复执行
- [ ] 环境变量和配置管理规范
- [ ] 数据库迁移脚本已验证
- [ ] 回滚方案已准备
- [ ] 监控告警已配置
- [ ] 日志收集已配置
- [ ] 服务健康检查已配置
- [ ] 部署文档完整
- [ ] 灰度发布方案已准备（如适用）
```

---

### 阶段10: 文档归档

**评审人**: 产品经理

```markdown
- [ ] 项目README完整
- [ ] 技术文档已更新
- [ ] API文档与实现一致
- [ ] 用户手册/操作指南已编写
- [ ] 部署运维文档完整
- [ ] 变更日志已记录
- [ ] 知识库/Wiki已更新
- [ ] 文档版本号已更新
- [ ] 所有文档已归档到指定位置
```

---

## 评审人映射

| 阶段 | 评审人 | 评审类型 |
|------|--------|----------|
| 需求分析 | 技术负责人 | 单人 |
| 技术评审 | 全员 | 多人(2/3通过) |
| UI/UX设计 | 产品经理、前端工程师 | 多人 |
| 数据库设计 | 系统架构师 | 单人 |
| API设计 | 前端工程师、后端工程师 | 多人 |
| 前端开发 | 代码审查员 | 单人 |
| 后端开发 | 代码审查员 | 单人 |
| 测试 | 技术负责人 | 单人 |
| 部署上线 | 技术负责人 | 单人 |
| 文档归档 | 产品经理 | 单人 |

---

## 评审结果处理

### 通过处理

```javascript
function onReviewPassed(stageId, result) {
  // 1. 记录通过信息
  recordReviewPass(stageId, result);

  // 2. 发送通知
  notifyStageOwner(stageId, {
    type: "review-passed",
    message: "恭喜！您的工作已通过评审",
    comments: result.comments
  });

  // 3. 触发下一阶段
  engine.handleReviewPassed(stageId, result);
}
```

### 不通过处理

```javascript
function onReviewRejected(stageId, result) {
  // 1. 创建Issue
  const issue = createIssue({
    stageId,
    type: "review-rejection",
    title: `${getStageDisplayName(stageId)}评审不通过`,
    description: result.comments,
    suggestions: result.suggestions,
    blockers: result.blockers,
    assignee: getStageAgent(stageId),
    priority: "high"
  });

  // 2. 发送通知
  notifyStageOwner(stageId, {
    type: "review-rejected",
    message: "评审未通过，请根据反馈进行修改",
    issue: issue,
    suggestions: result.suggestions
  });

  // 3. 更新状态
  engine.handleReviewRejected(stageId, result);
}
```
