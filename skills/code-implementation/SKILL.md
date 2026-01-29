---
name: code-implementation
description: 代码实现技能。使用场景：(1) TDD开发 (2) 代码编写 (3) 单元测试 (4) 代码规范 (5) 提交规范。按工程化标准产出高质量代码。
---

# 代码实现技能

按照工程化标准进行代码实现，遵循TDD方法，产出高质量代码。

## 适用角色

- 后端开发工程师
- 前端开发工程师
- 全栈开发工程师

## 输入/输出

**输入:** 技术设计文档、用户故事、代码规范、设计稿

**输出:** 源代码、单元测试、技术文档

## 实现流程

### 阶段1：理解需求

阅读文档、澄清疑问、任务拆解

### 阶段2：设计方案

数据结构、算法选择、模块划分、接口设计

### 阶段3：TDD开发

```
Red → Green → Refactor
(写失败测试 → 写最少代码 → 重构优化)
```

### 阶段4：代码实现

- 每个函数只做一件事
- 控制函数长度（≤30行）
- 适当的错误处理
- 日志记录关键操作

## 代码规范

| 类型 | 规范 | 示例 |
|------|------|------|
| 类名 | PascalCase | `UserService` |
| 函数 | camelCase | `getUserById` |
| 常量 | UPPER_SNAKE | `MAX_RETRY` |
| 布尔变量 | is/has前缀 | `isActive` |

## 提交规范

```
<type>(<scope>): <subject>

type: feat/fix/docs/style/refactor/test/chore
```

详细规范见 [references/coding-standards.md](references/coding-standards.md)
