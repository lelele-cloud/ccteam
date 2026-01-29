# Dockerfile最佳实践

## 多阶段构建示例

```dockerfile
# 构建阶段
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

## 最佳实践清单

1. **使用多阶段构建** - 减小最终镜像大小
2. **使用非root用户** - 提高安全性
3. **添加健康检查** - 便于容器编排
4. **使用.dockerignore** - 排除不必要文件
5. **固定基础镜像版本** - 保证构建一致性
6. **合并RUN命令** - 减少镜像层数
7. **清理缓存** - `npm cache clean --force`

## Python示例

```dockerfile
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

FROM python:3.11-slim
WORKDIR /app
RUN useradd -m appuser
USER appuser
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY . .
CMD ["python", "main.py"]
```
