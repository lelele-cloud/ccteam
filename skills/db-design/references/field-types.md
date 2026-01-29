# 字段类型规范

| 数据类型 | 推荐类型 | 说明 |
|----------|----------|------|
| 主键 | BIGINT | 预留足够空间 |
| 状态 | TINYINT | 0-127足够 |
| 金额 | DECIMAL(10,2) | 避免浮点精度问题 |
| 时间 | TIMESTAMP | 带时区，4字节 |
| 日期时间 | DATETIME | 不带时区，8字节 |
| 短文本 | VARCHAR(n) | 按需设置长度 |
| 长文本 | TEXT | 不与其他字段同表 |
| JSON数据 | JSON | MySQL 5.7+/PostgreSQL |

## 命名规范

**表名:**
- 使用小写字母和下划线
- 使用复数形式
- 示例: `users`, `order_items`

**字段名:**
- 使用小写字母和下划线
- 示例: `user_id`, `created_at`

**索引名:**
- 主键: `pk_表名`
- 唯一索引: `uk_表名_字段`
- 普通索引: `idx_表名_字段`

## 通用字段

每个表应包含：

```sql
id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '主键ID',
created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
deleted_at TIMESTAMP NULL COMMENT '删除时间（软删除）'
```
