# 部署文档

## 文档信息

| 属性 | 值 |
|------|-----|
| 项目名称 | {{project_name}} |
| 版本 | v1.0 |
| 创建日期 | {{date}} |
| 作者 | DevOps工程师 |

---

## 1. 概述

### 1.1 部署架构

```
                          ┌─────────────┐
                          │     DNS     │
                          └──────┬──────┘
                                 │
                          ┌──────┴──────┐
                          │     CDN     │
                          └──────┬──────┘
                                 │
                          ┌──────┴──────┐
                          │    Nginx    │
                          │  负载均衡   │
                          └──────┬──────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
       ┌──────┴──────┐    ┌──────┴──────┐    ┌──────┴──────┐
       │   App 01    │    │   App 02    │    │   App N     │
       │  Container  │    │  Container  │    │  Container  │
       └──────┬──────┘    └──────┬──────┘    └──────┬──────┘
              │                  │                  │
              └──────────────────┼──────────────────┘
                                 │
                     ┌───────────┼───────────┐
                     │           │           │
              ┌──────┴──────┐ ┌──┴──┐ ┌──────┴──────┐
              │  Database   │ │Redis│ │    MQ       │
              └─────────────┘ └─────┘ └─────────────┘
```

### 1.2 环境列表

| 环境 | 域名 | 用途 |
|------|------|------|
| 开发 | dev.example.com | 开发调试 |
| 测试 | test.example.com | 功能测试 |
| 预发 | staging.example.com | 上线前验证 |
| 生产 | www.example.com | 正式运行 |

---

## 2. 环境要求

### 2.1 服务器配置

| 服务 | 最低配置 | 推荐配置 |
|------|----------|----------|
| 应用服务器 | 2核4G | 4核8G |
| 数据库服务器 | 4核8G | 8核16G |
| 缓存服务器 | 2核4G | 4核8G |

### 2.2 软件依赖

| 软件 | 版本 | 说明 |
|------|------|------|
| Docker | 24.x+ | 容器运行时 |
| Docker Compose | 2.x+ | 容器编排 |
| Node.js | 20.x | 运行环境 |
| PostgreSQL | 15.x | 数据库 |
| Redis | 7.x | 缓存 |
| Nginx | 1.24+ | 反向代理 |

---

## 3. 部署步骤

### 3.1 环境准备

```bash
# 1. 安装Docker
curl -fsSL https://get.docker.com | sh

# 2. 安装Docker Compose
sudo apt-get install docker-compose-plugin

# 3. 创建部署目录
mkdir -p /opt/app
cd /opt/app
```

### 3.2 配置文件

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  app:
    image: ${IMAGE_NAME}:${IMAGE_TAG}
    container_name: app
    restart: always
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
    depends_on:
      - postgres
      - redis
    networks:
      - app-network

  postgres:
    image: postgres:15
    container_name: postgres
    restart: always
    environment:
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=${DB_NAME}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network

  redis:
    image: redis:7
    container_name: redis
    restart: always
    volumes:
      - redis-data:/data
    networks:
      - app-network

  nginx:
    image: nginx:latest
    container_name: nginx
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - app
    networks:
      - app-network

volumes:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

**.env 文件:**
```bash
# 镜像配置
IMAGE_NAME=your-app
IMAGE_TAG=latest

# 数据库配置
DB_USER=postgres
DB_PASSWORD=your_password
DB_NAME=app_db
DATABASE_URL=postgresql://postgres:your_password@postgres:5432/app_db

# Redis配置
REDIS_URL=redis://redis:6379
```

### 3.3 执行部署

```bash
# 1. 拉取最新代码
git pull origin main

# 2. 构建镜像
docker build -t your-app:latest .

# 3. 启动服务
docker-compose up -d

# 4. 检查服务状态
docker-compose ps

# 5. 查看日志
docker-compose logs -f app
```

---

## 4. Dockerfile

```dockerfile
# 构建阶段
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# 运行阶段
FROM node:20-alpine AS runner

WORKDIR /app

ENV NODE_ENV=production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 appuser

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

USER appuser

EXPOSE 3000

CMD ["node", "dist/main.js"]
```

---

## 5. CI/CD配置

### 5.1 GitHub Actions

**.github/workflows/deploy.yml:**
```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build

      - name: Build Docker image
        run: |
          docker build -t ${{ secrets.REGISTRY }}/${{ secrets.IMAGE_NAME }}:${{ github.sha }} .

      - name: Push to registry
        run: |
          echo ${{ secrets.REGISTRY_PASSWORD }} | docker login -u ${{ secrets.REGISTRY_USERNAME }} --password-stdin ${{ secrets.REGISTRY }}
          docker push ${{ secrets.REGISTRY }}/${{ secrets.IMAGE_NAME }}:${{ github.sha }}

      - name: Deploy to server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            cd /opt/app
            docker-compose pull
            docker-compose up -d
```

---

## 6. 回滚方案

### 6.1 快速回滚

```bash
# 1. 查看历史版本
docker images | grep your-app

# 2. 回滚到指定版本
export IMAGE_TAG=previous_tag
docker-compose up -d

# 3. 验证回滚结果
curl http://localhost:3000/health
```

### 6.2 数据库回滚

```bash
# 1. 查看迁移历史
npm run migration:show

# 2. 回滚最近的迁移
npm run migration:revert
```

---

## 7. 监控配置

### 7.1 健康检查

```bash
# 应用健康检查
curl http://localhost:3000/health

# Docker健康检查
docker inspect --format='{{.State.Health.Status}}' app
```

### 7.2 日志收集

```bash
# 查看应用日志
docker-compose logs -f app

# 查看Nginx日志
docker-compose logs -f nginx
```

---

## 8. 运维手册

### 8.1 常用命令

| 操作 | 命令 |
|------|------|
| 启动服务 | `docker-compose up -d` |
| 停止服务 | `docker-compose down` |
| 重启服务 | `docker-compose restart` |
| 查看日志 | `docker-compose logs -f` |
| 进入容器 | `docker exec -it app sh` |

### 8.2 故障排查

| 问题 | 排查步骤 |
|------|----------|
| 服务无法启动 | 检查日志、检查配置、检查端口占用 |
| 连接数据库失败 | 检查数据库状态、检查网络、检查配置 |
| 响应超时 | 检查负载、检查资源使用、检查日志 |

---

## 变更记录

| 版本 | 日期 | 修改人 | 修改内容 |
|------|------|--------|----------|
| v1.0 | {{date}} | - | 初始版本 |
