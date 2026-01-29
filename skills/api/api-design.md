# API设计技能

## 概述

系统化进行API接口设计，输出清晰、一致、易用的API规范。遵循RESTful最佳实践，确保API具有良好的可用性、可扩展性和可维护性。

## 适用角色

- API工程师（主要）
- 后端工程师
- 架构师
- 技术负责人

## 输入

- **需求文档**：PRD、用户故事、功能需求
- **数据模型**：数据库设计、实体关系图
- **技术架构**：系统架构文档、技术选型
- **现有API**：已有API规范（用于保持一致性）

## 输出

- **API设计文档**：`docs/ccteam/05-api/api-design.md`
- **OpenAPI规范**：`docs/ccteam/05-api/openapi.yaml`
- **接口示例**：`docs/ccteam/05-api/examples/`
- **错误码文档**：`docs/ccteam/05-api/error-codes.md`

## 执行步骤

### 步骤1：资源识别

1. **分析业务实体**
   - 从需求中识别核心业务实体
   - 确定实体的唯一标识（ID策略）
   - 建立实体之间的关系图

2. **定义资源**
   ```markdown
   ## 资源列表

   | 资源名称 | 路径 | 说明 |
   |----------|------|------|
   | 用户 | /users | 系统用户资源 |
   | 订单 | /orders | 用户订单资源 |
   | 商品 | /products | 商品信息资源 |
   ```

3. **确定资源关系**
   - 独立资源：`/users`, `/products`
   - 从属资源：`/users/{id}/orders`
   - 关联资源：`/orders/{id}/products`

### 步骤2：接口设计

1. **定义CRUD操作**
   ```markdown
   ## 用户资源接口

   | 操作 | 方法 | 路径 | 说明 |
   |------|------|------|------|
   | 列表 | GET | /users | 获取用户列表 |
   | 详情 | GET | /users/{id} | 获取单个用户 |
   | 创建 | POST | /users | 创建新用户 |
   | 更新 | PUT | /users/{id} | 完整更新用户 |
   | 部分更新 | PATCH | /users/{id} | 部分更新用户 |
   | 删除 | DELETE | /users/{id} | 删除用户 |
   ```

2. **设计查询参数**
   ```yaml
   # 分页参数
   page: 页码（从1开始）
   pageSize: 每页数量（默认20，最大100）
   # 或使用offset/limit
   offset: 偏移量
   limit: 限制数量

   # 排序参数
   sort: 排序字段（-field表示降序）
   # 示例：sort=-createdAt,name

   # 过滤参数
   filter[field]: 精确匹配
   filter[field][like]: 模糊匹配
   filter[field][gte]: 大于等于
   filter[field][lte]: 小于等于

   # 字段选择
   fields: 返回指定字段
   # 示例：fields=id,name,email

   # 关联扩展
   expand: 展开关联资源
   # 示例：expand=department,roles
   ```

3. **设计请求体**
   ```yaml
   # 创建用户请求
   POST /users
   Content-Type: application/json

   {
     "name": "string, required, 2-50字符",
     "email": "string, required, 邮箱格式",
     "phone": "string, optional, 手机号格式",
     "departmentId": "string, optional, 部门ID"
   }
   ```

### 步骤3：响应设计

1. **统一响应格式**
   ```json
   // 成功响应
   {
     "code": 0,
     "message": "success",
     "data": { ... },
     "meta": {
       "timestamp": "2024-01-01T00:00:00Z",
       "requestId": "req-uuid"
     }
   }

   // 列表响应
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

2. **设计错误响应**
   ```json
   // 错误响应
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

3. **HTTP状态码规范**
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

### 步骤4：错误码设计

1. **错误码体系**
   ```markdown
   ## 错误码规范

   格式：XYZNN
   - X: 错误类别（4=客户端错误, 5=服务端错误）
   - Y: 业务模块（0=通用, 1=用户, 2=订单...）
   - Z: 子模块（0=通用, 1-9=具体子模块）
   - NN: 错误序号（01-99）

   ## 通用错误码（40xxx）
   | 错误码 | HTTP状态码 | 说明 |
   |--------|------------|------|
   | 40001 | 400 | 参数缺失 |
   | 40002 | 400 | 参数格式错误 |
   | 40003 | 400 | 参数值超出范围 |
   | 40101 | 401 | 未登录或Token失效 |
   | 40301 | 403 | 无访问权限 |
   | 40401 | 404 | 资源不存在 |

   ## 用户模块错误码（41xxx）
   | 错误码 | HTTP状态码 | 说明 |
   |--------|------------|------|
   | 41001 | 400 | 用户名已存在 |
   | 41002 | 400 | 邮箱已被注册 |
   | 41003 | 400 | 密码强度不足 |
   ```

### 步骤5：文档编写

1. **编写OpenAPI规范**
   ```yaml
   openapi: 3.0.3
   info:
     title: 项目名称 API
     version: 1.0.0
     description: API文档描述

   servers:
     - url: https://api.example.com/v1
       description: 生产环境
     - url: https://api-staging.example.com/v1
       description: 测试环境

   paths:
     /users:
       get:
         summary: 获取用户列表
         tags: [用户]
         parameters:
           - name: page
             in: query
             schema:
               type: integer
               default: 1
         responses:
           '200':
             description: 成功
             content:
               application/json:
                 schema:
                   $ref: '#/components/schemas/UserListResponse'

   components:
     schemas:
       User:
         type: object
         properties:
           id:
             type: string
           name:
             type: string
   ```

2. **提供请求示例**
   ```markdown
   ## 创建用户

   ### 请求
   ```bash
   curl -X POST https://api.example.com/v1/users \
     -H "Authorization: Bearer <token>" \
     -H "Content-Type: application/json" \
     -d '{
       "name": "张三",
       "email": "zhangsan@example.com"
     }'
   ```

   ### 响应
   ```json
   {
     "code": 0,
     "message": "success",
     "data": {
       "id": "user-123",
       "name": "张三",
       "email": "zhangsan@example.com",
       "createdAt": "2024-01-01T00:00:00Z"
     }
   }
   ```
   ```

## 质量检查

### RESTful规范检查
- [ ] URL使用名词复数表示资源
- [ ] HTTP方法正确使用（GET查询/POST创建/PUT更新/DELETE删除）
- [ ] URL层级清晰，不超过3层
- [ ] 避免在URL中使用动词
- [ ] 查询参数命名一致（驼峰/蛇形统一）

### 响应设计检查
- [ ] 响应格式统一
- [ ] 使用正确的HTTP状态码
- [ ] 错误响应包含足够信息
- [ ] 分页信息完整
- [ ] 时间格式使用ISO 8601

### 文档完整性检查
- [ ] 所有接口都有文档说明
- [ ] 参数有类型、必填、示例说明
- [ ] 有请求和响应示例
- [ ] 错误码有完整说明
- [ ] 认证方式有说明

### 安全性检查
- [ ] 敏感接口需要认证
- [ ] 敏感数据不在URL中传输
- [ ] 有速率限制设计
- [ ] 输入参数有校验说明

### 兼容性检查
- [ ] 与现有API风格一致
- [ ] 版本策略明确
- [ ] 向后兼容性考虑

## 常见问题与解决方案

| 问题 | 解决方案 |
|------|----------|
| 资源命名困难 | 使用业务领域术语，与团队讨论达成共识 |
| 复杂查询设计 | 简单用查询参数，复杂考虑GraphQL或专用搜索接口 |
| 响应数据过大 | 使用fields参数按需返回，支持分页 |
| 版本升级困难 | 提前设计版本策略，新增字段优于修改字段 |
| 文档同步困难 | 使用代码生成文档，如从注解生成OpenAPI |

## OpenAPI模板

```yaml
openapi: 3.0.3
info:
  title: ${PROJECT_NAME} API
  version: ${VERSION}
  description: |
    ${PROJECT_DESCRIPTION}

    ## 认证
    API使用Bearer Token认证，在请求头中添加：
    ```
    Authorization: Bearer <token>
    ```

    ## 错误处理
    所有错误返回统一格式，详见错误码文档。

servers:
  - url: ${API_BASE_URL}
    description: ${ENV_NAME}

tags:
  - name: ${TAG_NAME}
    description: ${TAG_DESCRIPTION}

paths:
  # 在此添加接口定义

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  schemas:
    # 在此添加数据模型定义

    Error:
      type: object
      properties:
        code:
          type: integer
        message:
          type: string
        details:
          type: array
          items:
            type: object

security:
  - bearerAuth: []
```

## 相关技能

- [system-design](../architecture/system-design.md) - 系统设计技能
- [code-implementation](../development/code-implementation.md) - 代码实现技能
