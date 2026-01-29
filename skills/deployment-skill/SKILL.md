---
name: deployment
description: 应用部署技能，系统化执行应用部署流程。使用场景：(1) 配置CI/CD流水线 (2) 编写Dockerfile (3) 配置docker-compose (4) 制定部署和回滚方案 (5) 环境配置管
---

# 部署技能

系统化执行应用部署，从环境准备到上线验证，遵循DevOps最佳实践，实现持续交付。

## 适用角色

- DevOps工程师（主要）
- 后端工程师
- 技术负责人

## 输入/输出

**输入:** 代码仓库、配置信息、部署计划、变更说明

**输出:**
- 部署文档: `docs/ccteam/09-deployment/deployment.md`
- Dockerfile: 项目根目录
- docker-compose.yml: 项目根目录
- CI/CD配置: `.github/workflows/` 或 `.gitlab-ci.yml`

## 工作流程

```
环境准备 → 配置管理 → 容器化 → CI/CD配置 → 部署执行 → 回滚方案
```

### 步骤1: 环境准备

1. 确认环境清单（开发/测试/预发/生产）
2. 检查基础设施（CPU/内存/磁盘/网络）
3. 确认依赖服务（数据库/Redis/消息队列）

### 步骤2: 配置管理

环境变量规范：

```bash
# 应用配置
APP_ENV=production
APP_PORT=3000

# 数据库配置（密钥从密钥管理服务获取）
DB_HOST=localhost
DB_PORT=5432
DB_PASSWORD=${SECRET}
```

**密钥管理原则:**
- 使用CI/CD注入或密钥管理服务
- 禁止代码中硬编码
- 禁止提交到代码仓库

### 步骤3: 容器化

Dockerfile最佳实践见 [references/dockerfile.md](references/dockerfile.md)

docker-compose配置见 [references/docker-compose.md](references/docker-compose.md)

### 步骤4: CI/CD配置

GitHub Actions示例见 [references/cicd.md](references/cicd.md)

### 步骤5: 部署执行

**部署前检查:**
- [ ] CI测试全部通过
- [ ] 代码已Review
- [ ] 环境变量已配置
- [ ] 回滚方案确认

**部署验证:**
```bash
curl -f http://localhost:3000/health
curl http://localhost:3000/version
```

### 步骤6: 回滚方案

```bash
# Docker回滚
docker-compose down
docker tag myapp:previous myapp:current
docker-compose up -d

# Kubernetes回滚
kubectl rollout undo deployment/myapp
```

## 质量检查

**部署配置:**
- [ ] Dockerfile遵循最佳实践
- [ ] 使用非root用户
- [ ] 健康检查配置
- [ ] 资源限制配置

**CI/CD:**
- [ ] 测试自动运行
- [ ] 生产部署需要审批
- [ ] 支持回滚

**安全:**
- [ ] 密钥不在代码中
- [ ] 使用HTTPS
- [ ] 最小权限原则

## 常见陷阱

| 陷阱 | 规避方法 |
|------|----------|
| 密钥泄露 | 使用密钥管理服务 |
| 回滚困难 | 保留多个版本镜像 |
| 健康检查失败 | 调整启动时间和超时 |
| 配置不一致 | 使用配置模板 |
