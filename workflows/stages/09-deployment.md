# 阶段9：部署

## 基本信息

| 属性 | 值 |
|------|-----|
| 阶段ID | 9 |
| 阶段名称 | deployment |
| 显示名称 | 部署 |
| 负责角色 | devops-engineer |
| 评审方 | tech-lead, security-engineer |
| 可跳过 | 是 |

## 阶段目标

- 配置CI/CD流水线
- 准备部署环境
- 执行部署
- 配置监控告警

## 输入要求

- 前端代码（已测试）
- 后端代码（已测试）
- 技术方案
- 测试报告

## 输出产物

| 产物 | 模板 | 存放位置 |
|------|------|----------|
| 部署文档 | `templates/deployment.md` | `docs/ccteam/09-deployment/deployment.md` |
| CI/CD配置 | - | 项目根目录（如 `.github/workflows/`） |
| 运维手册 | - | `docs/ccteam/09-deployment/ops-manual.md` |

## 执行流程

```
1. 加载DevOps工程师智能体
2. 分析部署需求
3. 配置CI/CD流水线
4. 准备部署环境
5. 编写Dockerfile
6. 配置容器编排
7. 执行部署
8. 配置监控告警
9. 输出部署文档
10. 提交评审
```

## 部署选项

### 容器化
- Docker
- Docker Compose

### CI/CD平台
- GitHub Actions
- GitLab CI
- Jenkins

### 云平台
- 阿里云
- 腾讯云
- AWS
- Azure

### 监控方案
- Sentry（错误监控）
- Prometheus + Grafana
- 云平台自带监控

## 评审检查清单

- [ ] CI/CD流水线配置正确
- [ ] 部署环境准备完成
- [ ] 部署脚本可执行
- [ ] 回滚方案已准备
- [ ] 监控告警已配置
- [ ] 安全配置已检查
- [ ] 部署文档完整

## 下一阶段

评审通过后，自动进入 **阶段10：文档归档**
