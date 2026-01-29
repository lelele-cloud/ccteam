# CI/CD配置

## GitHub Actions示例

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

## GitLab CI示例

```yaml
stages:
  - test
  - build
  - deploy

test:
  stage: test
  script:
    - npm ci
    - npm test

build:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

deploy:
  stage: deploy
  environment: production
  when: manual
  script:
    - ssh deploy@server "docker-compose pull && docker-compose up -d"
```

## 最佳实践

1. **测试先行** - 测试通过才构建
2. **环境隔离** - 使用environment保护生产
3. **审批机制** - 生产部署需要审批
4. **版本标识** - 使用commit SHA标记镜像
5. **缓存利用** - 缓存依赖加速构建
