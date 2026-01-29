---
name: security-audit
description: 安全审计技能，系统化执行应用安全审计。使用场景：(1) 代码安全审查 (2) 渗透测试 (3) 威胁建模 (4) OWASP Top 10检查 (5) 漏洞评估和修复建议 (6) 安全规范制定。识别系
---

# 安全审计技能

系统化执行安全审计，从威胁建模到渗透测试，遵循OWASP安全标准，确保应用安全。

## 适用角色

- 安全工程师（主要）
- 技术负责人
- 后端工程师

## 输入/输出

**输入:** 系统架构、代码仓库、接口文档、测试环境

**输出:**
- 安全审计报告: `docs/ccteam/08-testing/security-report.md`
- 漏洞清单: `docs/ccteam/08-testing/vulnerabilities.md`
- 安全规范: `docs/ccteam/security-guidelines.md`

## 工作流程

```
威胁建模 → 代码审计 → 渗透测试 → 报告输出
```

### 步骤1: 威胁建模

1. **资产识别** - 列出敏感数据和服务
2. **攻击面分析** - 识别所有入口点
3. **威胁识别（STRIDE）**:

| 威胁类型 | 说明 |
|----------|------|
| Spoofing | 身份伪造 |
| Tampering | 数据篡改 |
| Repudiation | 抵赖 |
| Information Disclosure | 信息泄露 |
| Denial of Service | 拒绝服务 |
| Elevation of Privilege | 权限提升 |

### 步骤2: 代码审计

**静态分析工具:**
```bash
npm audit           # JavaScript
bandit -r src/      # Python
gosec ./...         # Go
semgrep --config=auto .
```

**人工审查清单:**
- 认证：密码哈希、Token安全、会话管理
- 授权：水平/垂直越权检查
- 输入：参数化查询、命令注入防护
- 输出：XSS防护、敏感信息过滤

详细检查清单见 [references/code-review-checklist.md](references/code-review-checklist.md)

### 步骤3: 渗透测试

**OWASP Top 10测试:**
- A01: 访问控制失效
- A02: 加密失败
- A03: 注入
- A07: 跨站脚本(XSS)

测试方法见 [references/pentest-guide.md](references/pentest-guide.md)

### 步骤4: 报告输出

漏洞报告格式见 [references/report-template.md](references/report-template.md)

## 质量检查

**审计覆盖:**
- [ ] 认证机制已审计
- [ ] 授权机制已审计
- [ ] 输入处理已审计
- [ ] 数据保护已审计

**OWASP Top 10:**
- [ ] 所有10项已检查
- [ ] 发现问题有修复建议

**报告质量:**
- [ ] 漏洞描述清晰
- [ ] 复现步骤完整
- [ ] 修复建议可操作

## 常见陷阱

| 陷阱 | 规避方法 |
|------|----------|
| 只做自动扫描 | 结合人工审查 |
| 漏报严重漏洞 | 全面覆盖OWASP Top 10 |
| 报告不可操作 | 提供具体修复代码 |
| 测试影响生产 | 使用独立测试环境 |
