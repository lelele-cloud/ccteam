# 阶段7：后端开发

## 基本信息

| 属性 | 值 |
|------|-----|
| 阶段ID | 7 |
| 阶段名称 | backend-dev |
| 显示名称 | 后端开发 |
| 负责角色 | backend-engineer |
| 评审方 | tech-lead, code-reviewer, security-engineer |
| 可跳过 | 否 |
| 并行阶段 | 阶段6（前端开发） |

## 阶段目标

- 实现后端服务
- 实现业务逻辑
- 实现数据访问层
- 实现API接口

## 输入要求

- PRD文档
- 技术方案
- 数据库设计
- API文档

## 输出产物

| 产物 | 存放位置 |
|------|----------|
| 后端代码 | `src/backend/` 或项目指定目录 |
| 服务文档 | `docs/ccteam/07-backend/services.md` |
| 后端说明 | `docs/ccteam/07-backend/backend-doc.md` |

## 执行流程

```
1. 加载后端工程师智能体
2. 分析技术方案和API文档
3. 搭建项目结构
4. 实现数据访问层
5. 实现业务逻辑层
6. 实现API接口
7. 编写单元测试
8. 提交代码评审
```

## 评审检查清单

- [ ] 代码符合编码规范
- [ ] 业务逻辑正确
- [ ] 数据库操作正确
- [ ] API实现符合文档
- [ ] 错误处理完善
- [ ] 日志记录完整
- [ ] 单元测试覆盖
- [ ] 安全检查通过

## 技术栈选项

根据项目配置，可选择：

### Node.js
- Express + TypeScript
- NestJS
- Fastify

### Python
- FastAPI
- Django
- Flask

### Java
- Spring Boot

### Go
- Gin
- Echo

## 下一阶段

与 **阶段6：前端开发** 并行执行，两者都完成后进入 **阶段8：测试**
