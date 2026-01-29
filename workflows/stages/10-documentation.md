# 阶段10：文档归档

## 基本信息

| 属性 | 值 |
|------|-----|
| 阶段ID | 10 |
| 阶段名称 | documentation |
| 显示名称 | 文档归档 |
| 负责角色 | doc-engineer |
| 评审方 | product-manager, tech-lead |
| 可跳过 | 否 |

## 阶段目标

- 整理项目文档
- 编写用户手册
- 编写开发文档
- 归档所有文档

## 输入要求

- 所有阶段产出的文档
- 代码库
- 测试报告
- 部署文档

## 输出产物

| 产物 | 存放位置 |
|------|----------|
| 用户手册 | `docs/ccteam/10-docs/user-manual.md` |
| 开发文档 | `docs/ccteam/10-docs/dev-guide.md` |
| API参考 | `docs/ccteam/10-docs/api-reference.md` |
| 项目总结 | `docs/ccteam/10-docs/project-summary.md` |
| 文档索引 | `docs/ccteam/README.md` |

## 执行流程

```
1. 加载文档工程师智能体
2. 收集所有阶段文档
3. 整理文档结构
4. 编写用户手册
5. 编写开发文档
6. 生成API参考文档
7. 编写项目总结
8. 创建文档索引
9. 提交评审
```

## 文档类型

### 用户文档
- 用户手册
- 快速入门指南
- FAQ

### 开发文档
- 开发环境搭建
- 代码规范
- 架构说明
- API参考

### 运维文档
- 部署指南
- 运维手册
- 故障排查

## 评审检查清单

- [ ] 文档结构清晰
- [ ] 用户手册完整
- [ ] 开发文档完整
- [ ] API文档与代码一致
- [ ] 文档格式统一
- [ ] 无拼写错误
- [ ] 文档索引完整

## 项目完成

评审通过后，项目开发流程完成。

### 完成后操作

1. 更新项目状态为 `completed`
2. 生成项目完成报告
3. 归档所有文档
4. 通知相关人员

### 项目状态更新

```json
{
  "status": "completed",
  "completedAt": "2024-XX-XX",
  "summary": {
    "totalStages": 10,
    "completedStages": 10,
    "totalDocuments": "XX",
    "totalCodeFiles": "XX"
  }
}
```
