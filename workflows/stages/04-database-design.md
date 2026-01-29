# 阶段4：数据库设计

## 基本信息

| 属性 | 值 |
|------|-----|
| 阶段ID | 4 |
| 阶段名称 | database-design |
| 显示名称 | 数据库设计 |
| 负责角色 | dba, backend-engineer |
| 评审方 | architect |
| 可跳过 | 是 |

## 阶段目标

- 设计数据模型
- 定义表结构
- 规划索引策略
- 考虑数据安全

## 输入要求

- PRD文档
- 技术方案
- 业务实体分析

## 输出产物

| 产物 | 模板 | 存放位置 |
|------|------|----------|
| 数据库设计 | `templates/db-design.md` | `docs/ccteam/04-database/db-design.md` |
| ER图 | - | `docs/ccteam/04-database/er-diagram.md` |
| 数据字典 | - | `docs/ccteam/04-database/data-dictionary.md` |

## 执行流程

```
1. 加载DBA和后端工程师智能体
2. 分析业务实体
3. 设计ER图
4. 定义表结构
5. 规划索引
6. 提交评审
```

## 评审检查清单

- [ ] 数据模型符合业务需求
- [ ] 表结构设计合理
- [ ] 索引策略已规划
- [ ] 数据安全已考虑
- [ ] 扩展性已考虑

## 下一阶段

评审通过后，自动进入 **阶段5：API设计**
