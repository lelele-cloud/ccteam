# 响应格式规范

## 成功响应

```json
{
  "code": 0,
  "message": "success",
  "data": { ... },
  "meta": {
    "timestamp": "2024-01-01T00:00:00Z",
    "requestId": "req-uuid"
  }
}
```

## 列表响应

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "items": [ ... ],
    "pagination": {
      "page": 1,
      "pageSize": 20,
      "total": 100,
      "totalPages": 5
    }
  }
}
```

## 错误响应

```json
{
  "code": 40001,
  "message": "参数验证失败",
  "details": [
    {
      "field": "email",
      "message": "邮箱格式不正确"
    }
  ],
  "meta": {
    "timestamp": "2024-01-01T00:00:00Z",
    "requestId": "req-uuid"
  }
}
```

## HTTP状态码规范

| 状态码 | 使用场景 |
|--------|----------|
| 200 | 成功（GET/PUT/PATCH/DELETE） |
| 201 | 创建成功（POST） |
| 204 | 无内容（DELETE成功但无返回体） |
| 400 | 请求参数错误 |
| 401 | 未认证 |
| 403 | 无权限 |
| 404 | 资源不存在 |
| 409 | 资源冲突 |
| 422 | 业务校验失败 |
| 429 | 请求过于频繁 |
| 500 | 服务器内部错误 |
