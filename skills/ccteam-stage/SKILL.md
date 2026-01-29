---
name: ccteam-stage
description: CCTeam阶段控制技能。使用场景：(1) 启动当前阶段 (2) 完成当前阶段 (3) 跳过阶段 (4) 回滚到上一阶段 (5) 查看阶段状态。控制开发流程的阶段流转。
---

# CCTeam - 阶段控制

控制项目开发流程的阶段状态。

## 使用方法

```bash
/ccteam-stage start              # 开始当前阶段
/ccteam-stage complete           # 完成当前阶段（触发评审）
/ccteam-stage skip --reason "原因" # 跳过当前阶段
/ccteam-stage rollback           # 回滚到上一阶段
/ccteam-stage status             # 查看当前状态
```

## 阶段状态流转

```
pending → in-progress → review → completed
                           ↓
                       rejected → in-progress（修改后重新提交）
```

## 命令说明

### start
- 将当前阶段状态设为 in-progress
- 加载对应智能体
- 记录开始时间

### complete
- 验证产出物完整性
- 将状态设为 review
- 触发评审流程

### skip
- 需要提供跳过原因
- 将状态设为 skipped
- 自动启动下一阶段

### rollback
- 将当前阶段状态重置为 pending
- 将上一阶段状态设为 in-progress
- 记录回滚原因

### status
- 显示当前阶段信息
- 显示已完成阶段数
- 显示待处理问题
