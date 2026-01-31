# Agent Skills

[English](#agent-skills-en) | [中文](#agent-skills-zh)

---

<a name="agent-skills-en"></a>
## Agent Skills (English)

This directory contains AI Agent Skills for assisting with code development, review, testing, documentation, and architecture.

### Skills Overview

| Skill | Description |
|-------|-------------|
| **mr-check** | MR/PR code review for GitLab and GitHub |
| **code-convention** | Development standards and best practices for Go, JavaScript, MySQL, MongoDB, Web API |
| **qa-case** | Test case generation based on code changes |
| **code-test** | Unit test, functional test, stress test, and destructive test generation |
| **code-docs** | Documentation generation (README.md, API.md, Page.md, Dependency.mmd) |
| **code-architecture** | Code architecture and file structure organization |
| **mysql-operator** | MySQL database operation standards and guidelines |

### Skill Details

#### 1. mr-check
Code review for GitLab/GitHub Merge Requests and Pull Requests.

**Features:**
- Fetch MR/PR code changes
- Deep code analysis (logic errors, security risks, performance issues)
- Generate structured review comments
- Publish comments to MR/PR

#### 2. code-convention
Development standards for multiple languages and scenarios.

**Features:**
- Go and JavaScript coding standards
- MySQL and MongoDB database standards
- Web API development standards
- High concurrency and cache consistency guidelines

#### 3. qa-case
Generate test cases from GitLab code changes.

**Features:**
- Analyze code change impact
- Generate test cases for frontend, server, PC, Android, iOS
- Generate system-level test cases

#### 4. code-test
Generate comprehensive tests for code.

**Features:**
- Unit tests (Go, Shell, JavaScript)
- System functional tests
- Stress tests for external APIs
- Destructive tests for error handling

#### 5. code-docs
Generate project documentation.

**Features:**
- README.md generation
- API.md for interface documentation
- Page.md for frontend pages
- Dependency.mmd for dependency graphs

#### 6. code-architecture
Guide code file structure organization.

**Features:**
- Language-specific architecture standards
- Directory and file naming conventions
- Best practices for Go, JavaScript, Vue, React, Angular, Electron, Node.js

#### 7. mysql-operator
MySQL database operation standards.

**Features:**
- Data operations (INSERT, UPDATE, DELETE, SELECT)
- Table operations (CREATE, DROP, ALTER)
- Index operations with Online DDL support
- High-risk operation warnings

---

<a name="agent-skills-zh"></a>
## Agent Skills (中文)

本目录包含 AI Agent Skills，用于辅助代码开发、审查、测试、文档和架构工作。

### Skills 概览

| Skill | 作用 |
|-------|------|
| **mr-check** | GitLab/GitHub MR/PR 代码审查 |
| **code-convention** | Go、JavaScript、MySQL、MongoDB、Web API 开发规范 |
| **qa-case** | 基于代码变更生成测试用例 |
| **code-test** | 生成单元测试、功能测试、压力测试、破坏性测试 |
| **code-docs** | 生成文档（README.md、API.md、Page.md、Dependency.mmd） |
| **code-architecture** | 代码架构和文件结构组织 |
| **mysql-operator** | MySQL 数据库操作规范 |

### Skill 详细说明

#### 1. mr-check
GitLab/GitHub 合并请求和拉取请求的代码审查。

**主要功能：**
- 获取 MR/PR 代码变更
- 深度代码分析（逻辑错误、安全风险、性能问题）
- 生成结构化审查评论
- 发布评论到 MR/PR

#### 2. code-convention
多语言和场景的开发规范。

**主要功能：**
- Go 和 JavaScript 编码规范
- MySQL 和 MongoDB 数据库规范
- Web API 开发规范
- 高并发和缓存一致性指南

#### 3. qa-case
基于 GitLab 代码变更生成测试用例。

**主要功能：**
- 分析代码变更影响
- 为前端、服务端、PC、安卓、iOS 生成测试用例
- 生成系统级测试用例

#### 4. code-test
为代码生成全面的测试。

**主要功能：**
- 单元测试（Go、Shell、JavaScript）
- 系统功能测试
- 外部 API 压力测试
- 错误处理破坏性测试

#### 5. code-docs
生成项目文档。

**主要功能：**
- 生成 README.md
- 生成 API.md 接口文档
- 生成 Page.md 前端页面文档
- 生成 Dependency.mmd 依赖图

#### 6. code-architecture
指导代码文件结构组织。

**主要功能：**
- 语言特定的架构标准
- 目录和文件命名规范
- Go、JavaScript、Vue、React、Angular、Electron、Node.js 最佳实践

#### 7. mysql-operator
MySQL 数据库操作规范。

**主要功能：**
- 数据操作（INSERT、UPDATE、DELETE、SELECT）
- 表操作（CREATE、DROP、ALTER）
- 索引操作（支持 Online DDL）
- 高风险操作警告

---

## Directory Structure

```
agent-skills/
├── README.md              # This file
├── code-architecture/     # Code architecture skill
│   ├── SKILL.md
│   └── references/
├── code-convention/       # Code convention skill
│   ├── SKILL.md
│   └── references/
├── code-docs/             # Code documentation skill
│   ├── SKILL.md
│   └── references/
├── code-test/             # Code testing skill
│   ├── SKILL.md
│   └── references/
├── mr-check/              # MR/PR review skill
│   ├── SKILL.md
│   └── references/
├── mysql-operator/        # MySQL operation skill
│   ├── SKILL.md
│   └── references/
└── qa-case/               # QA test case skill
    ├── SKILL.md
    └── references/
```

## Usage

Each skill is automatically activated when the user's request matches its description. The AI will:

1. Parse the SKILL.md YAML frontmatter
2. Follow the workflow defined in the skill
3. Reference the appropriate documentation in the `references/` directory
4. Provide guidance or perform actions based on the skill's capabilities

## Customization

To customize a skill:

1. Edit the `SKILL.md` file to modify activation conditions, role definition, or workflow
2. Edit files in the `references/` directory to modify specific standards or guidelines
3. Add resources to `assets/` or scripts to `scripts/` if needed

## License

MIT
