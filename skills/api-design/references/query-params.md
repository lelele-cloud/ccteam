# 查询参数规范

## 分页参数

```yaml
# 方式1: page/pageSize
page: 页码（从1开始）
pageSize: 每页数量（默认20，最大100）

# 方式2: offset/limit
offset: 偏移量
limit: 限制数量
```

## 排序参数

```yaml
sort: 排序字段（-field表示降序）
# 示例：sort=-createdAt,name
```

## 过滤参数

```yaml
filter[field]: 精确匹配
filter[field][like]: 模糊匹配
filter[field][gte]: 大于等于
filter[field][lte]: 小于等于
```

## 字段选择

```yaml
fields: 返回指定字段
# 示例：fields=id,name,email
```

## 关联扩展

```yaml
expand: 展开关联资源
# 示例：expand=department,roles
```
