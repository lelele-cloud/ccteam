# 数据库设计技能

## 概述

系统化进行数据库设计，从需求分析到物理实现，输出规范、高效、可扩展的数据库方案。遵循数据库设计最佳实践，确保数据完整性、性能和可维护性。

## 适用角色

- DBA（主要）
- 后端工程师
- 架构师
- 技术负责人

## 输入

- **需求文档**：PRD、用户故事、功能需求
- **技术架构**：系统架构文档
- **性能要求**：预期数据量、并发量、响应时间
- **现有数据库**：已有表结构（如有）

## 输出

- **ER图**：`docs/ccteam/04-database/er-diagram.md`
- **表结构文档**：`docs/ccteam/04-database/tables.md`
- **数据字典**：`docs/ccteam/04-database/data-dictionary.md`
- **迁移脚本**：`migrations/`

## 执行步骤

### 步骤1：需求分析

1. **识别业务实体**
   - 从需求文档中提取名词作为候选实体
   - 与业务方确认实体列表
   - 记录实体的业务含义

2. **分析数据特征**
   ```markdown
   ## 数据分析

   | 实体 | 预估数量 | 日增量 | 读写比 | 保留期限 |
   |------|----------|--------|--------|----------|
   | 用户 | 100万 | 1000 | 100:1 | 永久 |
   | 订单 | 1000万 | 10万 | 50:1 | 3年 |
   | 日志 | 10亿 | 1000万 | 1000:1 | 30天 |
   ```

3. **确定约束条件**
   - 数据库类型选择（MySQL/PostgreSQL/...）
   - 存储限制
   - 性能指标要求
   - 合规要求

### 步骤2：概念设计

1. **实体关系建模**
   ```
   用户(User) ─1:N─> 订单(Order) ─N:1─> 商品(Product)
                           │
                           └─1:N─> 订单项(OrderItem)
   ```

2. **绘制ER图**
   ```markdown
   ## ER图

   ```mermaid
   erDiagram
       USER ||--o{ ORDER : places
       ORDER ||--|{ ORDER_ITEM : contains
       PRODUCT ||--o{ ORDER_ITEM : "ordered in"

       USER {
           bigint id PK
           string name
           string email UK
           timestamp created_at
       }

       ORDER {
           bigint id PK
           bigint user_id FK
           decimal total_amount
           string status
           timestamp created_at
       }
   ```
   ```

3. **确定主键策略**
   | 类型 | 适用场景 | 示例 |
   |------|----------|------|
   | 自增ID | 单机、简单场景 | BIGINT AUTO_INCREMENT |
   | UUID | 分布式、需要客户端生成 | CHAR(36) |
   | 雪花ID | 分布式、有序要求 | BIGINT |

### 步骤3：逻辑设计

1. **规范化处理**
   ```
   第一范式(1NF)：字段原子性，不可再分
   第二范式(2NF)：消除部分依赖
   第三范式(3NF)：消除传递依赖

   反规范化场景：
   - 高频查询需要JOIN多表 → 冗余常用字段
   - 统计数据实时计算慢 → 增加汇总字段
   ```

2. **设计表结构**
   ```sql
   -- 用户表
   CREATE TABLE users (
       id BIGINT PRIMARY KEY AUTO_INCREMENT,
       name VARCHAR(50) NOT NULL,
       email VARCHAR(100) NOT NULL,
       password_hash VARCHAR(255) NOT NULL,
       status TINYINT NOT NULL DEFAULT 1,
       created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
       updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
       deleted_at TIMESTAMP NULL,

       UNIQUE KEY uk_email (email),
       KEY idx_status (status),
       KEY idx_created_at (created_at)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
   ```

3. **定义字段规范**
   | 数据类型 | 推荐类型 | 说明 |
   |----------|----------|------|
   | 主键 | BIGINT | 预留足够空间 |
   | 状态 | TINYINT | 0-127足够 |
   | 金额 | DECIMAL(10,2) | 避免浮点精度问题 |
   | 时间 | TIMESTAMP/DATETIME | 时区处理需注意 |
   | 长文本 | TEXT | 不与其他字段同表（如需要） |
   | JSON | JSON | MySQL 5.7+/PostgreSQL |

### 步骤4：物理设计

1. **索引设计**
   ```sql
   -- 索引设计原则
   -- 1. 主键自动索引
   -- 2. 外键建立索引
   -- 3. WHERE条件常用字段建立索引
   -- 4. 联合索引遵循最左前缀原则

   -- 示例：订单查询场景
   -- 常见查询：按用户ID和状态查询订单
   CREATE INDEX idx_user_status ON orders(user_id, status);

   -- 常见查询：按时间范围查询
   CREATE INDEX idx_created_at ON orders(created_at);
   ```

2. **分区策略**（大表）
   ```sql
   -- 按时间范围分区（日志表）
   CREATE TABLE logs (
       id BIGINT NOT NULL,
       message TEXT,
       created_at TIMESTAMP NOT NULL
   ) PARTITION BY RANGE (UNIX_TIMESTAMP(created_at)) (
       PARTITION p202401 VALUES LESS THAN (UNIX_TIMESTAMP('2024-02-01')),
       PARTITION p202402 VALUES LESS THAN (UNIX_TIMESTAMP('2024-03-01')),
       PARTITION pmax VALUES LESS THAN MAXVALUE
   );
   ```

3. **存储引擎选择**
   | 引擎 | 适用场景 |
   |------|----------|
   | InnoDB | 默认选择，支持事务和外键 |
   | MyISAM | 只读场景，不推荐 |
   | Memory | 临时表、缓存表 |

### 步骤5：输出文档

1. **数据字典格式**
   ```markdown
   ## 用户表 (users)

   | 字段 | 类型 | 可空 | 默认值 | 说明 |
   |------|------|------|--------|------|
   | id | BIGINT | NO | AUTO | 主键 |
   | name | VARCHAR(50) | NO | - | 用户名 |
   | email | VARCHAR(100) | NO | - | 邮箱（唯一） |
   | status | TINYINT | NO | 1 | 状态：1=正常，0=禁用 |
   | created_at | TIMESTAMP | NO | CURRENT | 创建时间 |
   | updated_at | TIMESTAMP | NO | CURRENT | 更新时间 |
   | deleted_at | TIMESTAMP | YES | NULL | 删除时间（软删除） |

   ### 索引
   | 索引名 | 类型 | 字段 | 说明 |
   |--------|------|------|------|
   | PRIMARY | 主键 | id | 主键索引 |
   | uk_email | 唯一 | email | 邮箱唯一索引 |
   | idx_status | 普通 | status | 状态查询索引 |
   ```

2. **迁移脚本规范**
   ```sql
   -- migrations/20240115_001_create_users_table.sql

   -- Up
   CREATE TABLE users (
       ...
   );

   -- Down
   DROP TABLE IF EXISTS users;
   ```

## 质量检查

### 设计规范检查
- [ ] 表名和字段名符合命名规范
- [ ] 每个表都有主键
- [ ] 字段类型选择合适
- [ ] 必填字段有NOT NULL约束
- [ ] 有默认值的字段设置了DEFAULT
- [ ] 时间字段使用统一类型

### 索引检查
- [ ] 外键字段都有索引
- [ ] 常用查询条件有索引支持
- [ ] 没有冗余索引
- [ ] 联合索引顺序正确

### 数据完整性检查
- [ ] 外键关系正确定义（或应用层保证）
- [ ] 枚举值有约束或文档说明
- [ ] 唯一约束正确设置
- [ ] 级联删除策略明确

### 性能检查
- [ ] 预估数据量和查询性能
- [ ] 大表有分区或归档策略
- [ ] 避免过宽的表（字段过多）
- [ ] TEXT/BLOB字段单独处理

### 文档检查
- [ ] ER图完整准确
- [ ] 数据字典完整
- [ ] 每个字段都有说明
- [ ] 索引设计有说明
- [ ] 迁移脚本有版本管理

## 常见问题与解决方案

| 问题 | 解决方案 |
|------|----------|
| 字段类型选择困难 | 参考字段规范表，选择够用的最小类型 |
| 是否需要外键 | 推荐应用层保证，外键影响性能但保证一致性 |
| 大表如何处理 | 分区、归档、分库分表 |
| 如何保证数据一致性 | 事务 + 唯一约束 + 应用层校验 |
| 软删除还是物理删除 | 重要数据软删除，临时数据物理删除 |

## 数据库设计模板

```sql
-- ===========================================
-- 项目名称: ${PROJECT_NAME}
-- 数据库: ${DATABASE_NAME}
-- 版本: v1.0.0
-- 创建时间: ${DATE}
-- ===========================================

-- 创建数据库
CREATE DATABASE IF NOT EXISTS ${DATABASE_NAME}
  DEFAULT CHARACTER SET utf8mb4
  DEFAULT COLLATE utf8mb4_unicode_ci;

USE ${DATABASE_NAME};

-- ===========================================
-- 表: ${TABLE_NAME}
-- 说明: ${TABLE_DESCRIPTION}
-- ===========================================
CREATE TABLE ${TABLE_NAME} (
    -- 主键
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键ID',

    -- 业务字段
    ${BUSINESS_FIELDS}

    -- 通用字段
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted_at TIMESTAMP NULL COMMENT '删除时间',

    -- 主键和索引
    PRIMARY KEY (id),
    ${INDEXES}
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='${TABLE_COMMENT}';
```

## 相关技能

- [system-design](../architecture/system-design.md) - 系统设计技能
- [code-implementation](../development/code-implementation.md) - 代码实现技能
