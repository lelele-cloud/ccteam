---
name: ccteam-config
description: 修改CCTeam项目配置技能。使用场景：(1) 修改项目配置 (2) 调整工作流设置 (3) 配置评审模式 (4) 自定义流程参数。管理settings.json配置。
---

# CCTeam - 项目配置

修改CCTeam项目的配置参数。

## 使用方法

调用 `/ccteam-config` 后显示当前配置并提供修改选项。

## 可配置项

### 语言设置

```json
{
  "language": {
    "documentation": "zh-CN",
    "code": "en"
  }
}
```

### 工作流设置

```json
{
  "workflow": {
    "autoTransition": true,
    "requireReview": true,
    "reviewMode": "single"
  }
}
```

### 评审模式

| 模式 | 说明 |
|------|------|
| single | 单人评审通过即可 |
| majority | 2/3多数通过 |
| unanimous | 全票通过 |

### 阶段超时设置

```json
{
  "stageTimeout": {
    "requirements": "2d",
    "tech-review": "1d",
    "ui-ux-design": "3d"
  }
}
```

## 配置命令

```
/ccteam-config show           # 显示当前配置
/ccteam-config set key value  # 设置配置项
/ccteam-config reset          # 重置为默认配置
```
