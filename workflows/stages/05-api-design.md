# 阶段5：API设计

## 基本信息

| 属性 | 值 |
|------|-----|
| 阶段ID | 5 |
| 阶段名称 | api-design |
| 显示名称 | API设计 |
| 负责角色 | api-engineer |
| 评审方 | frontend-engineer, backend-engineer |
| 可跳过 | 否 |

## 阶段目标

- 设计API接口
- 定义数据格式
- 编写API文档
- 规划版本策略

## 输入要求

- PRD文档
- 技术方案
- 数据库设计

## 输出产物

| 产物 | 模板 | 存放位置 |
|------|------|----------|
| API文档 | `templates/api-spec.yaml` | `docs/ccteam/05-api/api-spec.yaml` |
| 接口说明 | - | `docs/ccteam/05-api/api-doc.md` |

## 执行流程

```
1. 加载API工程师智能体
2. 分析功能需求
3. 设计RESTful接口
4. 定义请求/响应格式
5. 编写OpenAPI文档
6. 提交评审
```

## 评审检查清单

- [ ] 接口设计符合RESTful规范
- [ ] 数据格式定义清晰
- [ ] 错误码定义完整
- [ ] 认证授权已考虑
- [ ] API文档完整

## 下一阶段

评审通过后，并行进入 **阶段6：前端开发** 和 **阶段7：后端开发**
