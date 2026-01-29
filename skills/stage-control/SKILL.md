---
name: stage-control
description: CCTeam阶段控制技能。使用场景：(1) 阶段启动 (2) 阶段完成 (3) 阶段跳过 (4) 阶段回滚 (5) 状态查询。提供阶段级别的精细控制。
---

# 阶段控制 (Stage Control)

提供对开发流程各阶段的精细控制能力。

## 命令列表

| 命令 | 说明 |
|------|------|
| `start` | 启动当前阶段 |
| `complete` | 完成当前阶段 |
| `skip` | 跳过当前阶段 |
| `rollback` | 回滚到上一阶段 |
| `status` | 查看状态 |

## 操作流程

### start - 启动阶段

1. 检查前置阶段是否完成
2. 更新状态为 `in-progress`
3. 记录开始时间
4. 加载负责智能体
5. 输出阶段指导

### complete - 完成阶段

1. 验证产出物清单
2. 更新状态为 `review`
3. 触发评审流程
4. 等待评审结果

### skip - 跳过阶段

1. 验证跳过权限
2. 记录跳过原因
3. 更新状态为 `skipped`
4. 自动启动下一阶段

### rollback - 回滚阶段

1. 将当前阶段重置为 `pending`
2. 将上一阶段设为 `in-progress`
3. 记录回滚原因
4. 通知相关角色

## 状态转换

```
pending → in-progress → review → completed
              ↓           ↓
           skipped     rejected → in-progress
```
