# 代码实现技能

## 概述

按照工程化标准进行代码实现，遵循TDD（测试驱动开发）方法，产出高质量、可维护、可测试的代码，满足功能需求和代码质量要求。

## 适用角色

- 后端开发工程师
- 前端开发工程师
- 全栈开发工程师
- API开发工程师

## 输入

- **技术设计文档**：架构设计、接口规范
- **用户故事**：功能需求和验收标准
- **代码规范**：项目编码规范
- **设计稿**：UI设计稿（前端开发）

## 输出

- **源代码**：符合规范的实现代码
- **单元测试**：覆盖核心逻辑的测试代码
- **技术文档**：必要的代码注释和README

## 实现流程

### 阶段1：理解需求

1. **阅读相关文档**
   - 用户故事及验收标准
   - 技术设计文档
   - 接口规范
   - 相关的现有代码

2. **澄清疑问**
   - 列出不清楚的点
   - 与产品/技术负责人确认
   - 记录确认结果

3. **任务拆解**
   ```markdown
   ## 实现任务清单

   - [ ] 任务1：[描述]
     - 预估时间：X小时
     - 依赖：无/[依赖项]
   - [ ] 任务2：[描述]
     - 预估时间：X小时
     - 依赖：任务1
   ```

### 阶段2：设计方案

1. **技术方案设计**
   - 数据结构设计
   - 算法选择
   - 模块划分
   - 接口设计

2. **方案评审（复杂功能）**
   ```markdown
   ## 技术方案概要

   ### 背景
   [需求背景]

   ### 方案
   [技术方案描述]

   ### 关键设计
   - [设计点1]
   - [设计点2]

   ### 影响范围
   - [影响的模块/文件]

   ### 风险
   - [潜在风险]
   ```

### 阶段3：TDD开发

遵循 Red-Green-Refactor 循环：

```
┌──────────────────────────────────────────┐
│                                          │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐
│   │  RED    │───▶│  GREEN  │───▶│REFACTOR │
│   │写失败测试│    │写最少代码│    │  重构   │
│   └─────────┘    └─────────┘    └─────────┘
│        ▲                              │
│        │                              │
│        └──────────────────────────────┘
│                                          │
└──────────────────────────────────────────┘
```

1. **Red - 编写测试**
   ```python
   # 先写测试，此时测试会失败
   def test_user_registration():
       # Given
       user_data = {"email": "test@example.com", "password": "SecurePass123"}

       # When
       result = register_user(user_data)

       # Then
       assert result.success == True
       assert result.user.email == "test@example.com"
   ```

2. **Green - 实现代码**
   ```python
   # 写最少的代码让测试通过
   def register_user(user_data):
       user = User(email=user_data["email"])
       user.set_password(user_data["password"])
       user.save()
       return RegistrationResult(success=True, user=user)
   ```

3. **Refactor - 重构优化**
   - 消除重复代码
   - 优化代码结构
   - 提升可读性
   - 确保测试仍然通过

### 阶段4：代码实现

1. **目录结构规范**
   ```
   src/
   ├── modules/           # 功能模块
   │   └── user/
   │       ├── __init__.py
   │       ├── models.py      # 数据模型
   │       ├── services.py    # 业务逻辑
   │       ├── controllers.py # 控制器
   │       └── validators.py  # 验证器
   ├── common/            # 公共模块
   │   ├── utils.py
   │   └── exceptions.py
   └── tests/            # 测试代码
       └── user/
           ├── test_models.py
           └── test_services.py
   ```

2. **代码实现要点**
   - 每个函数/方法只做一件事
   - 控制函数长度（建议不超过30行）
   - 控制文件长度（建议不超过500行）
   - 适当的错误处理
   - 日志记录关键操作

## 代码规范

### 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| 类名 | PascalCase | `UserService`, `OrderController` |
| 函数/方法 | camelCase 或 snake_case | `getUserById`, `get_user_by_id` |
| 变量 | camelCase 或 snake_case | `userName`, `user_name` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 私有成员 | 前缀下划线 | `_privateMethod`, `_private_var` |
| 布尔变量 | is/has/can前缀 | `isActive`, `hasPermission` |

### 命名最佳实践

```python
# 好的命名
user_list = get_active_users()
is_valid = validate_email(email)
order_total_price = calculate_order_total(order_items)

# 避免的命名
data = get_data()  # 太笼统
flag = check()     # 不明确
temp = process()   # 无意义
```

### 代码结构规范

1. **导入顺序**
   ```python
   # 1. 标准库
   import os
   import sys

   # 2. 第三方库
   import requests
   from flask import Flask

   # 3. 本地模块
   from .models import User
   from .services import UserService
   ```

2. **类结构**
   ```python
   class UserService:
       """用户服务类

       提供用户相关的业务操作
       """

       # 类常量
       MAX_LOGIN_ATTEMPTS = 5

       def __init__(self, user_repository):
           """初始化

           Args:
               user_repository: 用户数据仓库
           """
           self._repository = user_repository

       # 公共方法
       def get_user(self, user_id: str) -> User:
           """获取用户信息"""
           pass

       # 私有方法
       def _validate_user(self, user: User) -> bool:
           """验证用户数据"""
           pass
   ```

### 注释规范

1. **文档注释（Docstring）**
   ```python
   def calculate_discount(price: float, discount_rate: float) -> float:
       """计算折扣后价格

       根据原价和折扣率计算最终价格，折扣率应在0-1之间

       Args:
           price: 原价，必须大于0
           discount_rate: 折扣率，0-1之间，0.1表示9折

       Returns:
           折扣后的价格

       Raises:
           ValueError: 当价格小于0或折扣率不在0-1之间时

       Examples:
           >>> calculate_discount(100, 0.1)
           90.0
       """
       if price < 0:
           raise ValueError("价格不能小于0")
       if not 0 <= discount_rate <= 1:
           raise ValueError("折扣率必须在0-1之间")
       return price * (1 - discount_rate)
   ```

2. **行内注释**
   ```python
   # 好的注释 - 解释为什么
   # 使用二分查找，因为数据已排序且数据量大
   result = binary_search(sorted_list, target)

   # 避免的注释 - 重复代码
   # 将x加1
   x = x + 1
   ```

## 单元测试要求

### 测试覆盖标准

| 代码类型 | 最低覆盖率 | 说明 |
|----------|------------|------|
| 核心业务逻辑 | 90% | 必须高覆盖 |
| 工具函数 | 80% | 关注边界条件 |
| 控制器层 | 70% | 主要流程覆盖 |
| 数据模型 | 60% | 关注验证逻辑 |

### 测试组织结构

```python
class TestUserRegistration:
    """用户注册测试套件"""

    def setup_method(self):
        """每个测试方法执行前的准备"""
        self.service = UserService()

    def teardown_method(self):
        """每个测试方法执行后的清理"""
        pass

    # 正常场景
    def test_register_with_valid_data_should_succeed(self):
        """有效数据注册应成功"""
        pass

    # 边界条件
    def test_register_with_min_length_password_should_succeed(self):
        """最小长度密码注册应成功"""
        pass

    # 异常场景
    def test_register_with_invalid_email_should_raise_error(self):
        """无效邮箱注册应抛出异常"""
        pass

    # 并发场景（如有）
    def test_concurrent_registration_should_handle_correctly(self):
        """并发注册应正确处理"""
        pass
```

### 测试命名规范

```
test_[被测方法]_[场景]_[预期结果]
```

示例：
- `test_login_with_valid_credentials_should_return_token`
- `test_login_with_wrong_password_should_raise_auth_error`
- `test_create_order_with_empty_cart_should_raise_validation_error`

## 提交规范

### Commit Message格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type类型

| Type | 说明 |
|------|------|
| feat | 新功能 |
| fix | 修复bug |
| docs | 文档变更 |
| style | 代码格式（不影响逻辑） |
| refactor | 重构（非新功能、非修复） |
| perf | 性能优化 |
| test | 测试相关 |
| chore | 构建过程或辅助工具变更 |

### 示例

```
feat(user): 实现用户注册功能

- 添加用户注册API
- 实现邮箱格式验证
- 添加密码强度校验
- 发送注册确认邮件

Closes #123
```

## 质量检查

### 代码质量
- [ ] 符合项目编码规范
- [ ] 命名清晰、有意义
- [ ] 函数职责单一
- [ ] 无明显重复代码
- [ ] 适当的错误处理

### 测试质量
- [ ] 核心逻辑有单元测试
- [ ] 测试覆盖正常和异常场景
- [ ] 测试命名规范
- [ ] 测试独立、可重复

### 文档质量
- [ ] 公共接口有文档注释
- [ ] 复杂逻辑有说明注释
- [ ] README 更新（如有必要）

### 提交质量
- [ ] Commit message 符合规范
- [ ] 每个提交是独立的完整变更
- [ ] 不包含调试代码
- [ ] 不包含敏感信息

## 相关技能

- [code-review](./code-review.md) - 代码评审技能
