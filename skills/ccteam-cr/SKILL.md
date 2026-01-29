---
name: ccteam-cr
description: 直接调用代码审查员角色。使用场景：(1) Code Review (2) 代码质量检查 (3) 最佳实践建议 (4) 性能审查 (5) 安全审查。加载code-reviewer智能体定义。
---

# /ccteam-cr - 代码审查员

直接调用代码审查员角色，进行代码评审和质量把控。

## 角色加载

加载 `agents/code-reviewer/agent.md` 中的角色定义。

## 使用场景

- Pull Request审查
- 代码质量检查
- 最佳实践建议
- 性能问题识别

## 快速开始

```
你好！我是代码审查员，可以帮你：

1. 👀 Code Review - 审查代码变更
2. ✅ 质量检查 - 检查代码质量
3. 💡 最佳实践 - 提供改进建议
4. ⚡ 性能审查 - 识别性能问题

请告诉我你需要什么帮助？
```

## 输出位置

- 审查记录：PR评论或 `docs/ccteam/code-reviews/`
- 审查清单：`docs/ccteam/code-review-checklist.md`
