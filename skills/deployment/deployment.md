# 部署技能

## 概述

系统化执行应用部署，从环境准备到上线验证，确保部署过程可靠、可回滚、可追踪。遵循DevOps最佳实践，实现持续交付。

## 适用角色

- DevOps工程师（主要）
- 后端工程师
- 技术负责人

## 输入

- **代码仓库**：已通过CI的代码分支
- **配置信息**：环境配置、密钥信息
- **部署计划**：部署时间、范围、回滚方案
- **变更说明**：本次部署的变更内容

## 输出

- **部署文档**：`docs/ccteam/09-deployment/deployment.md`
- **Dockerfile**：项目根目录
- **docker-compose.yml**：项目根目录
- **CI/CD配置**：`.github/workflows/` 或 `.gitlab-ci.yml`

## 执行步骤

### 步骤1：环境准备

1. **环境清单确认**
   ```markdown
   ## 环境清单

   | 环境 | 用途 | 地址 | 配置 |
   |------|------|------|------|
   | 开发环境 | 开发调试 | dev.example.com | 低配 |
   | 测试环境 | QA测试 | test.example.com | 中配 |
   | 预发环境 | 预发布验证 | staging.example.com | 与生产相同 |
   | 生产环境 | 正式运行 | api.example.com | 高配 |
   ```

2. **基础设施检查**
   - [ ] 服务器资源充足（CPU/内存/磁盘）
   - [ ] 网络配置正确（安全组、负载均衡）
   - [ ] 域名和SSL证书就绪
   - [ ] 数据库和缓存服务就绪

3. **依赖服务确认**
   - [ ] 数据库连接正常
   - [ ] Redis连接正常
   - [ ] 第三方服务可用
   - [ ] 消息队列就绪

### 步骤2：配置管理

1. **环境变量规范**
   ```bash
   # 应用配置
   APP_ENV=production
   APP_PORT=3000
   APP_LOG_LEVEL=info

   # 数据库配置
   DB_HOST=localhost
   DB_PORT=5432
   DB_NAME=myapp
   DB_USER=app_user
   DB_PASSWORD=${SECRET}

   # Redis配置
   REDIS_HOST=localhost
   REDIS_PORT=6379

   # 第三方服务
   API_KEY=${SECRET}
   ```

2. **密钥管理**
   ```yaml
   # 推荐方式
   密钥来源:
     - 环境变量（CI/CD注入）
     - 密钥管理服务（Vault、AWS Secrets Manager）
     - Kubernetes Secrets

   # 禁止方式
   禁止:
     - 代码中硬编码
     - 配置文件中明文存储
     - 提交到代码仓库
   ```

3. **配置文件模板**
   ```yaml
   # config/production.yaml
   server:
     port: ${APP_PORT}
     host: 0.0.0.0

   database:
     host: ${DB_HOST}
     port: ${DB_PORT}
     name: ${DB_NAME}

   logging:
     level: ${APP_LOG_LEVEL}
     format: json
   ```

### 步骤3：容器化配置

1. **Dockerfile最佳实践**
   ```dockerfile
   # 多阶段构建
   FROM node:18-alpine AS builder
   WORKDIR /app
   COPY package*.json ./
   RUN npm ci --only=production
   COPY . .
   RUN npm run build

   # 生产镜像
   FROM node:18-alpine
   WORKDIR /app

   # 安全：非root用户
   RUN addgroup -g 1001 -S nodejs && \
       adduser -S nodejs -u 1001
   USER nodejs

   # 复制构建产物
   COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
   COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules

   # 健康检查
   HEALTHCHECK --interval=30s --timeout=3s \
     CMD wget --quiet --tries=1 --spider http://localhost:3000/health || exit 1

   EXPOSE 3000
   CMD ["node", "dist/main.js"]
   ```

2. **docker-compose配置**
   ```yaml
   version: '3.8'

   services:
     app:
       build: .
       ports:
         - "3000:3000"
       environment:
         - NODE_ENV=production
       env_file:
         - .env
       depends_on:
         - db
         - redis
       restart: unless-stopped
       healthcheck:
         test: ["CMD", "wget", "-q", "--spider", "http://localhost:3000/health"]
         interval: 30s
         timeout: 10s
         retries: 3

     db:
       image: postgres:15-alpine
       volumes:
         - postgres_data:/var/lib/postgresql/data
       environment:
         POSTGRES_DB: ${DB_NAME}
         POSTGRES_USER: ${DB_USER}
         POSTGRES_PASSWORD: ${DB_PASSWORD}

     redis:
       image: redis:7-alpine
       volumes:
         - redis_data:/data

   volumes:
     postgres_data:
     redis_data:
   ```

### 步骤4：CI/CD配置

1. **GitHub Actions示例**
   ```yaml
   name: Deploy

   on:
     push:
       branches: [main]

   jobs:
     test:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - uses: actions/setup-node@v4
           with:
             node-version: '18'
         - run: npm ci
         - run: npm test

     build:
       needs: test
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - name: Build Docker image
           run: docker build -t myapp:${{ github.sha }} .
         - name: Push to registry
           run: |
             docker tag myapp:${{ github.sha }} registry.example.com/myapp:${{ github.sha }}
             docker push registry.example.com/myapp:${{ github.sha }}

     deploy:
       needs: build
       runs-on: ubuntu-latest
       environment: production
       steps:
         - name: Deploy to server
           run: |
             ssh deploy@server "docker pull registry.example.com/myapp:${{ github.sha }}"
             ssh deploy@server "docker-compose up -d"
   ```

### 步骤5：部署执行

1. **部署检查清单**
   ```markdown
   ## 部署前检查

   代码准备：
   - [ ] 代码已合并到部署分支
   - [ ] CI测试全部通过
   - [ ] 代码已Review通过

   配置准备：
   - [ ] 环境变量已配置
   - [ ] 密钥已就位
   - [ ] 数据库迁移脚本准备

   通知准备：
   - [ ] 已通知相关方
   - [ ] 值班人员就位
   - [ ] 回滚方案确认
   ```

2. **滚动更新策略**
   ```yaml
   # Kubernetes滚动更新
   spec:
     replicas: 3
     strategy:
       type: RollingUpdate
       rollingUpdate:
         maxSurge: 1
         maxUnavailable: 0
   ```

3. **部署验证**
   ```bash
   # 健康检查
   curl -f http://localhost:3000/health

   # 版本确认
   curl http://localhost:3000/version

   # 核心功能验证
   curl -X POST http://localhost:3000/api/test
   ```

### 步骤6：回滚方案

1. **快速回滚流程**
   ```bash
   # Docker回滚
   docker-compose down
   docker tag myapp:previous myapp:current
   docker-compose up -d

   # Kubernetes回滚
   kubectl rollout undo deployment/myapp

   # 手动回滚
   git revert HEAD
   git push origin main
   ```

2. **回滚检查清单**
   ```markdown
   回滚前：
   - [ ] 确认回滚版本
   - [ ] 确认数据库是否需要回滚
   - [ ] 通知相关方

   回滚中：
   - [ ] 执行回滚命令
   - [ ] 监控服务状态
   - [ ] 验证核心功能

   回滚后：
   - [ ] 确认服务恢复
   - [ ] 发送回滚通知
   - [ ] 记录事件
   ```

## 质量检查

### 部署配置检查
- [ ] Dockerfile遵循最佳实践
- [ ] 镜像大小合理
- [ ] 使用非root用户
- [ ] 健康检查配置
- [ ] 资源限制配置

### CI/CD检查
- [ ] 测试自动运行
- [ ] 构建自动化
- [ ] 生产部署需要审批
- [ ] 有版本标识
- [ ] 支持回滚

### 安全检查
- [ ] 密钥不在代码中
- [ ] 使用HTTPS
- [ ] 网络隔离配置
- [ ] 最小权限原则
- [ ] 安全扫描通过

### 监控检查
- [ ] 日志收集配置
- [ ] 健康检查端点
- [ ] 关键指标监控
- [ ] 告警规则配置

## 常见问题与解决方案

| 问题 | 解决方案 |
|------|----------|
| 部署后服务无法启动 | 检查日志、环境变量、依赖服务 |
| 数据库连接失败 | 检查网络、凭据、连接数限制 |
| 端口冲突 | 检查端口占用，调整配置 |
| 内存不足 | 检查资源限制，扩容或优化 |
| 健康检查失败 | 检查服务启动时间，调整超时 |

## 部署文档模板

```markdown
# ${PROJECT_NAME} 部署文档

## 环境信息

| 环境 | 地址 | 配置 |
|------|------|------|
| 生产 | xxx | xxx |

## 部署步骤

1. 拉取最新代码
2. 构建镜像
3. 执行部署
4. 验证服务

## 回滚方案

1. 执行回滚命令
2. 验证服务恢复

## 监控地址

- 日志：xxx
- 监控：xxx
- 告警：xxx

## 联系人

- DevOps: xxx
- 开发: xxx
```

## 相关技能

- [system-design](../architecture/system-design.md) - 系统设计技能
- [security-audit](../security/security-audit.md) - 安全审计技能
