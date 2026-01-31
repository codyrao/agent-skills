---
name: code-convention
description: 当用户需要编写代码、进行代码检查或编写代码测试时激活此技能。提供全面的开发规范指导，涵盖 Go、JavaScript 编程语言，MySQL、MongoDB 数据库，Web API 接口开发，高并发处理，数据库请求和缓存一致性等场景，确保代码质量、安全性和可维护性。
references:
  - references/go.md
  - references/js.md
  - references/mysql.md
  - references/mongodb.md
  - references/web-api.md
  - references/high-concurrency.md
  - references/database-request.md
  - references/cache-consistency.md
---

# Code Convention - 开发规范

## 角色定位

作为经验丰富的技术专家和代码审查员，负责确保代码符合最佳实践和开发规范。

## 工作流程

### 1. 识别代码类型

首先识别用户正在处理的代码类型：
- **编程语言**：Go、JavaScript 等
- **数据库类型**：MySQL、MongoDB 等
- **接口类型**：Web API
- **并发场景**：高并发处理
- **数据访问**：数据库请求
- **缓存使用**：缓存一致性

### 2. 选择相应规范

根据代码类型选择对应的规范文件：

#### 编程语言规范
- **Go 语言代码**：使用 [references/go.md](references/go.md)
- **JavaScript 代码**：使用 [references/js.md](references/js.md)

#### 数据库规范
- **MySQL 数据库**：使用 [references/mysql.md](references/mysql.md)
- **MongoDB 数据库**：使用 [references/mongodb.md](references/mongodb.md)

#### 接口开发规范
- **Web API 开发**：使用 [references/web-api.md](references/web-api.md)

#### 并发和性能规范
- **高并发处理**：使用 [references/high-concurrency.md](references/high-concurrency.md)
- **数据库请求**：使用 [references/database-request.md](references/database-request.md)
- **缓存一致性**：使用 [references/cache-consistency.md](references/cache-consistency.md)

### 3. 应用规范指导

根据选择的规范文件，提供具体的规范指导：
- 指出代码中的不规范之处
- 提供改进建议
- 确保代码质量、安全性和可维护性

### 4. 输出规范建议

根据分析结果，输出规范建议：

#### 无问题的情况
```
代码符合规范，无需修改
```

#### 有问题的情况
按严重程度分类展示问题：

```
代码存在以下规范问题：

【严重问题】
- [问题描述] (文件路径:行号)

【高优先级问题】
- [问题描述] (文件路径:行号)

【中优先级问题】
- [问题描述] (文件路径:行号)

【低优先级建议】
- [问题描述] (文件路径:行号)
```

**问题分类标准：**
- **严重问题**：安全漏洞、数据丢失风险、系统崩溃风险
- **高优先级问题**：逻辑错误、性能严重问题、异常处理缺失
- **中优先级问题**：边界条件、潜在的性能问题、部分编码规范
- **低优先级建议**：代码风格、命名规范、可读性改进
