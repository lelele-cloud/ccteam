# 阶段2：技术评审

## 基本信息

| 属性 | 值 |
|------|-----|
| 阶段ID | 2 |
| 阶段名称 | tech-review |
| 显示名称 | 技术评审 |
| 负责角色 | tech-lead, architect |
| 评审方 | 全员 |
| 可跳过 | 否 |

## 阶段目标

- 评估技术可行性
- 确定系统架构
- 选择技术栈
- 识别技术风险
- 制定技术方案

## 输入要求

- PRD文档
- 用户故事
- 功能清单

## 输出产物

| 产物 | 模板 | 存放位置 |
|------|------|----------|
| 技术方案 | `templates/tech-design.md` | `docs/ccteam/02-architecture/tech-design.md` |
| 架构文档 | `templates/architecture-doc.md` | `docs/ccteam/02-architecture/architecture.md` |
| 技术选型 | - | `docs/ccteam/02-architecture/tech-selection.md` |

## 执行流程

```
1. 加载技术负责人和架构师智能体
2. 分析PRD中的技术需求
3. 评估技术可行性
4. 设计系统架构
5. 选择技术栈
6. 撰写技术方案
7. 提交全员评审
```

## 评审检查清单

- [ ] 架构设计合理
- [ ] 技术选型有依据
- [ ] 性能指标已定义
- [ ] 安全需求已考虑
- [ ] 可扩展性已规划
- [ ] 技术风险已识别
- [ ] 开发规范已定义

## 下一阶段

评审通过后，自动进入 **阶段3：UI/UX设计**
