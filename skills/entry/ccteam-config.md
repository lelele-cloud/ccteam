# CCTeam - 配置管理

## 概述

修改项目配置，如技术栈、流程设置等。

## 使用方法

当用户调用 `/ccteam-config` 时：

### 1. 展示当前配置

```
⚙️ CCTeam 配置

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📝 语言设置
- 文档语言：中文
- 代码语言：英文
- 注释语言：中文

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🛠️ 技术栈
前端：React + TypeScript + Tailwind
后端：Node.js + Express + Prisma
数据库：PostgreSQL + Redis
移动端：React Native

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔄 流程设置
- 自动流转：开启
- 评审要求：开启
- 并行开发：开启
- 可跳过阶段：UI/UX设计, 数据库设计

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📁 输出目录
docs/ccteam/

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

请选择要修改的配置：
1. 语言设置
2. 技术栈
3. 流程设置
4. 输出目录
5. 重置为默认
```

### 2. 修改配置

根据用户选择进入对应配置修改流程。

### 3. 保存配置

更新 `settings.json` 和 `project-status.json`。

## 可配置项

### 技术栈选项

**前端框架**：
- React + TypeScript
- Vue 3 + TypeScript
- Next.js
- Nuxt.js

**后端框架**：
- Node.js + Express
- Node.js + NestJS
- Python + FastAPI
- Go + Gin
- Java + Spring Boot

**数据库**：
- PostgreSQL
- MySQL
- MongoDB
- SQLite（开发）

**移动端**：
- React Native
- Flutter

### 流程选项

- 自动流转：是/否
- 评审要求：是/否
- 并行开发：是/否
- 跳过阶段：选择可跳过的阶段
