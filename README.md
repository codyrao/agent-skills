# Agent Skills

本目录包含三个 AI Agent Skills，用于辅助代码开发、审查和测试工作。

## Skills 概览

### 1. mr-check-skill
**作用**：MR/PR 代码审查

当用户需要审查 GitLab 或 GitHub Merge Request (MR) / Pull Request (PR) 中的代码变更时激活此技能。扮演严谨、客观的资深技术负责人，深度分析代码变更并提供专业审查意见。

**主要功能**：
- 从 GitLab/GitHub 获取 MR/PR 代码变更
- 深度分析代码变更（逻辑错误、安全风险、性能瓶颈、异常风险、编码规范）
- 生成结构化审查评论
- 将审查评论发布到 MR/PR

**适用场景**：
- 代码审查
- MR/PR 质量检查
- 代码质量评估

**依赖**：
- GitLab MCP 服务（获取 MR 变更、发布评论）
- GitHub MCP 服务（获取 PR 变更、发布评论）
- 代码分析 MCP 服务（可选，用于深度分析）

### 2. code-skill
**作用**：AI Vibe Coding 开发规范

当用户需要编写代码、进行代码检查或编写代码测试时激活此技能。提供全面的开发规范指导，确保代码质量、安全性和可维护性。

**主要功能**：
- 提供编程语言规范（Go、JavaScript）
- 提供数据库规范（MySQL、MongoDB）
- 提供 Web API 开发规范
- 提供高并发处理规范
- 提供数据库请求规范
- 提供缓存一致性规范

**适用场景**：
- 编写新代码
- 进行代码审查和检查
- 编写单元测试
- 重构现有代码
- 优化代码性能
- 设计数据库结构
- 开发 Web API 接口

**规范参考**：
- Go 官方标准、字节跳动和阿里云的 Go 代码开发规范
- JS 官方标准、字节跳动和阿里云的 JS 代码开发规范
- MySQL 官方标准、字节跳动和阿里云的 MySQL SQL 规范
- MongoDB 官方标准、字节跳动和阿里云的 MongoDB NoSQL 规范
- API 官方标准、字节跳动和阿里云的 API 开发规范
- 字节跳动和阿里云的高并发、数据库请求、缓存一致性规范

### 3. qa-case-skill
**作用**：测试用例生成

当用户需要根据 GitLab 代码变更生成测试用例时激活此技能。分析代码变更，识别影响的接口、页面、功能和流程，生成各端的功能测试用例、压测用例以及整体系统测试用例。

**主要功能**：
- 从 GitLab 获取 MR 代码变更
- 分析项目信息
- 分析代码变更影响（接口、页面、功能、流程、端）
- 生成各端测试用例（前端、服务端、PC 端、安卓端、iOS 端）
- 生成整体系统测试用例（跨端功能、端到端流程、整体性能、整体压测）
- 将测试用例发布到 MR

**适用场景**：
- 代码变更后的测试用例生成
- MR/PR 的测试用例生成
- 功能测试用例生成
- 性能测试用例生成
- 压测测试用例生成

**依赖**：
- GitLab MCP 服务（获取 MR 变更、发布评论）
- 分析 GitLab 仓库代码的 MCP 服务（可选，用于分析项目信息）

## 如何调整 Skill 内容以适应自定义要求

### 1. 修改 SKILL.md 文件
每个 skill 的主文件是 `SKILL.md`，包含以下部分：

#### YAML 前置元数据
```yaml
---
name: skill-name
description: 技能描述
references:
  - reference1.md
  - reference2.md
---
```

**自定义方式**：
- 修改 `name`：更改技能名称
- 修改 `description`：更新技能描述，使其更符合你的使用场景
- 修改 `references`：添加或删除引用的规范文件

#### 激活条件
在 `## 激活条件` 部分定义何时激活此技能。

**自定义方式**：
- 添加新的激活条件
- 修改现有激活条件
- 使激活条件更具体或更宽泛

#### 角色定位
在 `## 角色定位` 部分定义 AI 的角色和行为。

**自定义方式**：
- 修改角色描述
- 添加新的角色要求
- 调整角色的行为方式

#### 工作流程
在 `## 工作流程` 部分定义技能的执行步骤。

**自定义方式**：
- 添加新的工作步骤
- 修改现有工作步骤
- 调整工作流程的顺序

### 2. 修改 references 目录下的规范文件
每个 skill 的 `references` 目录包含具体的规范文件。

**自定义方式**：
- 修改现有规范内容
- 添加新的规范要求
- 删除不需要的规范要求
- 调整规范的优先级

**示例**：
- 修改 `go.md`：添加新的 Go 语言规范
- 修改 `web-api.md`：调整 API 命名规范
- 修改 `frontend.md`：添加新的前端测试要求

### 3. 修改 assets 和 scripts 目录
- `assets` 目录：存放静态资源文件（如图片、配置文件等）
- `scripts` 目录：存放可执行脚本文件

**自定义方式**：
- 添加资源文件到 `assets` 目录
- 添加脚本文件到 `scripts` 目录
- 在 `SKILL.md` 中引用这些资源

### 4. 重新打包 Skill
修改完成后，需要重新打包 skill：

```bash
# 删除旧的压缩包
rm -f skill-name.tar.gz

# 创建新的压缩包
tar -czf skill-name.tar.gz skill-name/
```

## Skill 使用方式

### Filesystem-based Agent
对于基于文件系统的 Agent（如 Trae IDE），Skills 通过文件系统进行集成。

#### Skill 发现
Agent 在启动时会扫描配置的目录，查找包含 `SKILL.md` 文件的文件夹。

#### Skill 加载
Agent 只解析 `SKILL.md` 文件的 YAML 前置元数据，保持初始上下文使用较低。

#### Skill 激活
当用户任务与 Skill 的描述匹配时，Agent 会激活相应的 Skill。

#### 资源访问
Agent 可以通过 shell 命令访问 Skill 捆绑的资源（assets 和 scripts 目录）。

### Tool-based Agent
对于基于工具的 Agent，Skills 通过工具实现进行集成。

#### 工具实现
开发者需要实现特定的工具，允许模型触发 Skill 并访问捆绑的资源。

#### 工具调用
模型通过调用工具来激活 Skill 和访问资源。

### 详细集成指南
详细的集成指南请参考：[https://agentskills.io/integrate-skills](https://agentskills.io/integrate-skills)

该指南包含：
- 集成方法概述
- Skill 发现和加载
- 上下文注入
- 安全考虑
- 参考实现

## 安装和使用

### 1. 安装 Skill
将 skill 目录复制到 Agent 的 skills 配置目录。

### 2. 配置 Agent
根据 Agent 的配置方式，配置 skills 目录路径。

### 3. 重启 Agent
重启 Agent 以加载新的 Skills。

### 4. 使用 Skill
根据 Skill 的激活条件，触发相应的功能。

## 示例

### 示例 1：使用 mr-check-skill 审查代码
```
用户：请帮我审查 https://gitlab.com/example/project 的 MR !123，token 是 abc123

AI：开始执行 MR 代码审查...
正在从 GitLab 获取 MR !123 的代码变更...
成功获取 5 个文件的变更
开始分析代码变更...
分析完成，发现 3 个问题
正在发布评论到 MR...
成功发布评论到 GitLab MR
MR/PR 代码审查完成！
```

### 示例 2：使用 code-skill 编写代码
```
用户：请帮我编写一个 Go 函数，用于从 MySQL 数据库中查询用户信息

AI：根据 [go.md](references/go.md) 和 [database-request.md](references/database-request.md) 规范，以下是一个符合 Go 语言命名规范的函数：

[函数代码]

**符合的规范：**
1. 命名规范
2. 错误处理
3. 数据库请求规范
4. 并发安全
5. 代码格式
```

### 示例 3：使用 qa-case-skill 生成测试用例
```
用户：请为 GitLab MR !456 生成测试用例

AI：正在获取 GitLab MR !456 的代码变更...
成功获取 3 个文件的变更
正在分析项目信息...
分析完成，识别到以下影响：
- 前端：用户列表页面、用户详情页面
- 服务端：用户列表接口、用户详情接口
- 安卓端：用户列表页面、用户详情页面
- iOS 端：用户列表页面、用户详情页面
- 整体系统：用户数据同步流程

正在生成测试用例...
测试用例生成完成
正在发布测试用例到 MR...
成功发布测试用例到 GitLab MR

**发布的评论内容：**
(AI生成) 以下是这次合并的测试用例：
[测试用例列表]

任务完成
```

## 注意事项

### 1. MCP 服务依赖
部分 Skill 依赖 MCP 服务才能正常工作，请确保已配置相应的 MCP 工具。

### 2. Token 安全
使用 Skill 时需要提供访问 token，请妥善保管，不要在公开场合泄露。

### 3. 权限要求
确保 token 有相应的权限（读取代码、发布评论等）。

### 4. 网络连接
确保能够访问相应的 Git 平台和 MCP 服务。

### 5. 自定义建议
在自定义 Skill 时，建议：
- 保持 YAML 前置元数据格式正确
- 保持激活条件清晰明确
- 保持工作流程逻辑清晰
- 保持规范内容具体可操作
- 定期测试 Skill 是否正常工作

## 目录结构

```
agent-skills/
├── README.md                          # 本文件
├── mr-check-skill/                   # MR/PR 代码审查技能
│   ├── SKILL.md
│   └── references/
│       ├── code-analysis-skill.md
│       └── repo-operations-skill.md
├── code-skill/                       # AI Vibe Coding 开发规范技能
│   ├── SKILL.md
│   ├── assets/
│   ├── scripts/
│   └── references/
│       ├── go.md
│       ├── js.md
│       ├── mysql.md
│       ├── mongodb.md
│       ├── web-api.md
│       ├── high-concurrency.md
│       ├── database-request.md
│       └── cache-consistency.md
└── qa-case-skill/                    # 测试用例生成技能
    ├── SKILL.md
    ├── assets/
    ├── scripts/
    └── references/
        ├── frontend.md
        ├── server.md
        ├── pc.md
        ├── android.md
        ├── ios.md
        └── system.md
```

## 许可证

请根据你的项目需求，遵循相应的开源许可证。

## 贡献

欢迎提交 Issue 和 Pull Request 来改进这些 Skills。

## 联系方式

如有问题或建议，请通过以下方式联系：
- 提交 Issue
- 发送邮件
- 参与讨论

## 参考资料

- [Agent Skills 集成指南](https://agentskills.io/integrate-skills)
- [skills-ref 库](https://github.com/agentskills/skills-ref)
