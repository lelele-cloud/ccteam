---
name: ccteam-resume
description: 恢复中断的CCTeam项目技能。使用场景：(1) 恢复之前中断的项目 (2) 继续上次的开发进度 (3) 重新加载项目上下文。从project-status.json恢复项目状态。
---

# CCTeam - 项目恢复

恢复之前中断的项目，继续开发流程。

## 使用方法

调用 `/ccteam-resume` 后：

1. 检查 `docs/ccteam/project-status.json` 是否存在
2. 读取项目状态
3. 加载对应阶段的智能体
4. 提供上下文概要

## 恢复流程

```
读取项目状态
    │
    ├── 状态存在
    │   ├── 显示项目概要
    │   ├── 显示当前阶段
    │   ├── 加载负责智能体
    │   └── 继续开发
    │
    └── 状态不存在
        └── 提示使用 /ccteam-new 创建新项目
```

## 输出格式

```
🔄 项目恢复：{{projectName}}

项目描述：{{description}}
技术栈：{{techStack}}

当前阶段：{{currentStage.displayName}}
阶段状态：{{currentStage.status}}

上次更新：{{updatedAt}}

正在加载 {{assignedAgent}} 智能体...

提示：使用 /ccteam-status 查看详细进度
```

## 异常处理

| 情况 | 处理 |
|------|------|
| 文件不存在 | 提示创建新项目 |
| 文件格式错误 | 尝试修复或提示重新初始化 |
| 阶段状态异常 | 提示手动干预 |
