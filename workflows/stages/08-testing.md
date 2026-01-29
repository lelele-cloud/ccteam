# 阶段8：测试

## 基本信息

| 属性 | 值 |
|------|-----|
| 阶段ID | 8 |
| 阶段名称 | testing |
| 显示名称 | 测试 |
| 负责角色 | qa-engineer |
| 评审方 | tech-lead, product-manager |
| 可跳过 | 否 |

## 阶段目标

- 执行功能测试
- 执行集成测试
- 执行性能测试
- 执行安全测试
- 输出测试报告

## 输入要求

- PRD文档
- 前端代码
- 后端代码
- API文档
- 测试用例

## 输出产物

| 产物 | 模板 | 存放位置 |
|------|------|----------|
| 测试计划 | `templates/test-plan.md` | `docs/ccteam/08-testing/test-plan.md` |
| 测试用例 | `templates/test-cases.md` | `docs/ccteam/08-testing/test-cases.md` |
| 测试报告 | `templates/test-report.md` | `docs/ccteam/08-testing/test-report.md` |
| 缺陷清单 | - | `docs/ccteam/08-testing/bug-list.md` |

## 执行流程

```
1. 加载QA工程师智能体
2. 制定测试计划
3. 编写测试用例
4. 执行功能测试
5. 执行集成测试
6. 执行性能测试
7. 执行安全测试
8. 记录缺陷
9. 输出测试报告
10. 提交评审
```

## 测试类型

### 功能测试
- 单元测试
- 集成测试
- E2E测试
- 回归测试

### 非功能测试
- 性能测试
- 安全测试
- 兼容性测试
- 可用性测试

## 评审检查清单

- [ ] 测试计划完整
- [ ] 测试用例覆盖率达标
- [ ] 功能测试通过
- [ ] 集成测试通过
- [ ] 性能指标达标
- [ ] 安全测试通过
- [ ] 缺陷已修复或已记录
- [ ] 测试报告完整

## 测试工具

### 前端测试
- Jest
- Vitest
- Playwright
- Cypress

### 后端测试
- Jest
- pytest
- JUnit
- Go test

### 性能测试
- k6
- Artillery
- JMeter

## 下一阶段

评审通过后，自动进入 **阶段9：部署**
