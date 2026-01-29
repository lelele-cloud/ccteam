---
name: db-design
description: 数据库设计技能，系统化进行数据库方案设计。使用场景：(1) 设计新数据库表结构 (2) 数据库迁移设计 (3) 索引优化设计 (4) 数据模型评审 (5) 编写数据字典。输出规范、高效、可扩展的数据库
---

# 数据库设计技能

系统化进行数据库设计，从需求分析到物理实现，遵循数据库设计最佳实践，确保数据完整性、性能和可维护性。

## 适用角色

- DBA（主要）
- 后端工程师
- 架构师

## 输入/输出

**输入:** PRD、技术架构、性能要求、现有表结构

**输出:**
- ER图: `docs/ccteam/04-database/er-diagram.md`
- 表结构文档: `docs/ccteam/04-database/tables.md`
- 数据字典: `docs/ccteam/04-database/data-dictionary.md`
- 迁移脚本: `migrations/`

## 工作流程

```
需求分析 → 概念设计 → 逻辑设计 → 物理设计 → 输出文档
```

### 步骤1: 需求分析

1. 从需求文档中提取实体（名词）
2. 分析数据特征：

| 实体 | 预估数量 | 日增量 | 读写比 | 保留期限 |
|------|----------|--------|--------|----------|
| 用户 | 100万 | 1000 | 100:1 | 永久 |
| 订单 | 1000万 | 10万 | 50:1 | 3年 |

3. 确定约束：数据库类型、存储限制、性能指标

### 步骤2: 概念设计

1. 建立实体关系模型
2. 绘制ER图（使用Mermaid）
3. 确定主键策略：

| 类型 | 适用场景 |
|------|----------|
| 自增ID | 单机、简单场景 |
| UUID | 分布式、客户端生成 |
| 雪花ID | 分布式、有序要求 |

### 步骤3: 逻辑设计

规范化处理（1NF→2NF→3NF），必要时反规范化。

基本表结构：

```sql
CREATE TABLE table_name (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    -- 业务字段
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

字段类型规范见 [references/field-types.md](references/field-types.md)

### 步骤4: 物理设计

**索引设计原则:**
- 主键自动索引
- 外键建立索引
- WHERE条件常用字段建索引
- 联合索引遵循最左前缀

大表策略见 [references/large-table.md](references/large-table.md)

### 步骤5: 输出文档

数据字典格式和迁移脚本规范见 [references/doc-template.md](references/doc-template.md)

## 质量检查

**设计规范:**
- [ ] 表名字段名符合命名规范
- [ ] 每个表都有主键
- [ ] 字段类型选择合适
- [ ] 必填字段有NOT NULL

**索引检查:**
- [ ] 外键字段都有索引
- [ ] 常用查询条件有索引
- [ ] 没有冗余索引

**文档检查:**
- [ ] ER图完整准确
- [ ] 数据字典完整
- [ ] 迁移脚本有版本管理

## 常见陷阱

| 陷阱 | 规避方法 |
|------|----------|
| 字段类型过大 | 选择够用的最小类型 |
| 索引过多 | 只建必要索引，定期清理 |
| 缺少软删除 | 重要数据使用deleted_at |
| 时区问题 | 统一使用UTC或TIMESTAMP |
