---
name: api-design
description: API接口设计技能，系统化进行RESTful API设计。使用场景：(1) 设计新的API接口 (2) 评审API设计方案 (3) 编写OpenAPI规范 (4) 设计错误码体系 (5) API版本策
---

# API设计技能

系统化进行API接口设计，遵循RESTful最佳实践，确保API具有良好的可用性、可扩展性和可维护性。

## 适用角色

- API工程师（主要）
- 后端工程师
- 架构师

## 输入/输出

**输入:** PRD、数据模型、技术架构、现有API规范

**输出:**
- API设计文档: `docs/ccteam/05-api/api-design.md`
- OpenAPI规范: `docs/ccteam/05-api/openapi.yaml`
- 错误码文档: `docs/ccteam/05-api/error-codes.md`

## 工作流程

```
资源识别 → 接口设计 → 响应设计 → 错误码设计 → 文档编写
```

### 步骤1: 资源识别

1. 从需求中识别核心业务实体
2. 定义资源路径（使用名词复数）
3. 确定资源关系（独立/从属/关联）

```markdown
| 资源名称 | 路径 | 说明 |
|----------|------|------|
| 用户 | /users | 系统用户资源 |
| 订单 | /orders | 用户订单资源 |
```

### 步骤2: 接口设计

定义CRUD操作和查询参数：

| 操作 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 列表 | GET | /resources | 获取列表 |
| 详情 | GET | /resources/{id} | 获取单个 |
| 创建 | POST | /resources | 创建新资源 |
| 更新 | PUT/PATCH | /resources/{id} | 更新资源 |
| 删除 | DELETE | /resources/{id} | 删除资源 |

查询参数规范见 [references/query-params.md](references/query-params.md)

### 步骤3: 响应设计

统一响应格式：

```json
{
  "code": 0,
  "message": "success",
  "data": { ... },
  "meta": { "timestamp": "...", "requestId": "..." }
}
```

HTTP状态码规范：
- 200/201: 成功
- 400: 参数错误
- 401/403: 认证/授权失败
- 404: 资源不存在
- 500: 服务器错误

详细规范见 [references/response-format.md](references/response-format.md)

### 步骤4: 错误码设计

错误码格式: `XYZNN`
- X: 错误类别（4=客户端, 5=服务端）
- Y: 业务模块
- Z: 子模块
- NN: 错误序号

详细错误码体系见 [references/error-codes.md](references/error-codes.md)

### 步骤5: 文档编写

使用OpenAPI 3.0规范编写API文档，模板见 [references/openapi-template.md](references/openapi-template.md)

## 质量检查

**RESTful规范:**
- [ ] URL使用名词复数
- [ ] HTTP方法正确使用
- [ ] URL层级不超过3层

**响应设计:**
- [ ] 响应格式统一
- [ ] 状态码正确
- [ ] 分页信息完整

**文档完整性:**
- [ ] 所有接口有文档
- [ ] 参数有类型和示例
- [ ] 错误码有说明

## 常见陷阱

| 陷阱 | 规避方法 |
|------|----------|
| URL使用动词 | 使用资源名词，动作用HTTP方法表达 |
| 版本管理混乱 | 提前设计版本策略（URL/Header） |
| 响应格式不一致 | 定义并遵循统一响应格式 |
| 文档与实现脱节 | 从代码注解生成文档 |
