# 大表处理策略

## 分区策略

按时间范围分区（日志表）：

```sql
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

## 存储引擎选择

| 引擎 | 适用场景 |
|------|----------|
| InnoDB | 默认选择，支持事务和外键 |
| MyISAM | 只读场景，不推荐 |
| Memory | 临时表、缓存表 |

## 归档策略

1. 定期将历史数据迁移到归档表
2. 使用分区表便于删除历史数据
3. 考虑冷热数据分离

## 分库分表

**水平分表场景:**
- 单表数据量超过5000万
- 单表大小超过10GB

**分片键选择:**
- 用户ID（按用户查询）
- 时间（按时间范围查询）
