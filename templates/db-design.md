# 数据库设计文档

## 文档信息

| 属性 | 值 |
|------|-----|
| 项目名称 | {{project_name}} |
| 版本 | v1.0 |
| 创建日期 | {{date}} |
| 作者 | DBA / 后端工程师 |
| 数据库类型 | PostgreSQL / MySQL |

---

## 1. 概述

### 1.1 设计目标

- 数据完整性
- 查询性能
- 可扩展性
- 数据安全

### 1.2 设计原则

- 遵循第三范式（适当反范式优化性能）
- 合理设计索引
- 考虑数据量增长
- 敏感数据加密存储

---

## 2. ER图

```
┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│    User     │       │    Order    │       │   Product   │
├─────────────┤       ├─────────────┤       ├─────────────┤
│ id (PK)     │───┐   │ id (PK)     │   ┌───│ id (PK)     │
│ name        │   │   │ user_id(FK) │───┘   │ name        │
│ email       │   └──>│ product_id  │       │ price       │
│ created_at  │       │ quantity    │       │ stock       │
└─────────────┘       │ status      │       │ created_at  │
                      │ created_at  │       └─────────────┘
                      └─────────────┘
```

---

## 3. 数据字典

### 3.1 用户表 (users)

| 字段名 | 类型 | 长度 | 允许空 | 默认值 | 说明 |
|--------|------|------|--------|--------|------|
| id | BIGINT | - | NO | AUTO_INCREMENT | 主键 |
| username | VARCHAR | 50 | NO | - | 用户名 |
| email | VARCHAR | 100 | NO | - | 邮箱 |
| password_hash | VARCHAR | 255 | NO | - | 密码哈希 |
| status | TINYINT | - | NO | 1 | 状态：0禁用 1正常 |
| created_at | TIMESTAMP | - | NO | CURRENT_TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | - | NO | CURRENT_TIMESTAMP | 更新时间 |

**索引：**
| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| uk_email | 唯一 | email | 邮箱唯一 |
| idx_username | 普通 | username | 用户名查询 |
| idx_created_at | 普通 | created_at | 时间查询 |

### 3.2 订单表 (orders)

| 字段名 | 类型 | 长度 | 允许空 | 默认值 | 说明 |
|--------|------|------|--------|--------|------|
| id | BIGINT | - | NO | AUTO_INCREMENT | 主键 |
| order_no | VARCHAR | 32 | NO | - | 订单号 |
| user_id | BIGINT | - | NO | - | 用户ID |
| total_amount | DECIMAL | (10,2) | NO | - | 总金额 |
| status | TINYINT | - | NO | 0 | 订单状态 |
| created_at | TIMESTAMP | - | NO | CURRENT_TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | - | NO | CURRENT_TIMESTAMP | 更新时间 |

**索引：**
| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| uk_order_no | 唯一 | order_no | 订单号唯一 |
| idx_user_id | 普通 | user_id | 用户查询 |
| idx_status | 普通 | status | 状态查询 |

---

## 4. 表结构DDL

### 4.1 users表

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '主键',
    username VARCHAR(50) NOT NULL COMMENT '用户名',
    email VARCHAR(100) NOT NULL COMMENT '邮箱',
    password_hash VARCHAR(255) NOT NULL COMMENT '密码哈希',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0禁用 1正常',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    UNIQUE KEY uk_email (email),
    KEY idx_username (username),
    KEY idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

### 4.2 orders表

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '主键',
    order_no VARCHAR(32) NOT NULL COMMENT '订单号',
    user_id BIGINT NOT NULL COMMENT '用户ID',
    total_amount DECIMAL(10,2) NOT NULL COMMENT '总金额',
    status TINYINT NOT NULL DEFAULT 0 COMMENT '订单状态',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    UNIQUE KEY uk_order_no (order_no),
    KEY idx_user_id (user_id),
    KEY idx_status (status),
    FOREIGN KEY (user_id) REFERENCES users(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='订单表';
```

---

## 5. 索引策略

### 5.1 索引设计原则

- 主键使用自增ID
- 外键字段建立索引
- 高频查询字段建立索引
- 组合索引遵循最左前缀
- 避免过度索引

### 5.2 查询优化

| 查询场景 | 推荐索引 | 说明 |
|----------|----------|------|
| 按用户查订单 | idx_user_id | 单字段索引 |
| 按状态+时间查询 | idx_status_created | 组合索引 |

---

## 6. 数据安全

### 6.1 敏感字段

| 表 | 字段 | 加密方式 |
|------|------|----------|
| users | password_hash | BCrypt |
| users | email | 脱敏显示 |

### 6.2 数据备份

| 备份类型 | 频率 | 保留时间 |
|----------|------|----------|
| 全量备份 | 每日 | 30天 |
| 增量备份 | 每小时 | 7天 |
| 日志备份 | 实时 | 7天 |

---

## 7. 数据迁移

### 7.1 迁移脚本命名

```
V{版本号}__{描述}.sql
例如：V1.0.0__create_users_table.sql
```

### 7.2 迁移示例

```sql
-- V1.0.0__create_users_table.sql
CREATE TABLE IF NOT EXISTS users (...);

-- V1.0.1__add_phone_column.sql
ALTER TABLE users ADD COLUMN phone VARCHAR(20) AFTER email;
```

---

## 变更记录

| 版本 | 日期 | 修改人 | 修改内容 |
|------|------|--------|----------|
| v1.0 | {{date}} | - | 初始版本 |
