# 阶段6：前端开发

## 基本信息

| 属性 | 值 |
|------|-----|
| 阶段ID | 6 |
| 阶段名称 | frontend-dev |
| 显示名称 | 前端开发 |
| 负责角色 | frontend-engineer |
| 评审方 | tech-lead, code-reviewer |
| 可跳过 | 否 |
| 并行阶段 | 阶段7（后端开发） |

## 阶段目标

- 实现前端功能
- 构建用户界面
- 实现交互逻辑
- 集成API接口

## 输入要求

- PRD文档
- UI/UX设计稿
- API文档
- 技术方案

## 输出产物

| 产物 | 存放位置 |
|------|----------|
| 前端代码 | `src/frontend/` 或项目指定目录 |
| 组件文档 | `docs/ccteam/06-frontend/components.md` |
| 前端说明 | `docs/ccteam/06-frontend/frontend-doc.md` |

## 执行流程

```
1. 加载前端工程师智能体
2. 分析设计稿和API文档
3. 搭建项目结构
4. 实现UI组件
5. 实现业务逻辑
6. 集成API接口
7. 编写单元测试
8. 提交代码评审
```

## 评审检查清单

- [ ] 代码符合编码规范
- [ ] UI还原度达标
- [ ] 交互逻辑正确
- [ ] API集成正确
- [ ] 单元测试覆盖
- [ ] 响应式适配完成
- [ ] 性能指标达标

## 技术栈选项

根据项目配置，可选择：

### Web项目
- React + TypeScript + Vite
- Vue3 + TypeScript + Vite
- Next.js

### 移动端项目
- React Native
- Flutter

### 小程序项目
- 微信原生
- Taro
- uni-app

## 下一阶段

与 **阶段7：后端开发** 并行执行，两者都完成后进入 **阶段8：测试**
