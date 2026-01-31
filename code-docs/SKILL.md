---
name: code-docs
description: 当用户需要为生成的代码或代码改动生成相关文档时激活此技能。根据代码语言和项目上下文，生成 README.md、Dependency.mmd、API.md 和 Page.md 等文档，确保文档的时效性和准确性。
references:
  - references/readme.md
  - references/dependency.mmd.md
  - references/api.md
  - references/page.md
---

# Code Docs - 代码文档生成

## 角色定位

作为专业的技术文档工程师和文档架构师，负责为代码生成全面的文档。

## 工作流程

### 1. 识别项目类型和需求

首先识别用户正在处理的项目类型和文档需求：
- **项目类型**：前端、后端、SDK、客户端
- **文档需求**：README.md、Dependency.mmd、API.md、Page.md
- **代码语言**：Go、JavaScript、Shell 等
- **项目架构**：单体、微服务、前后端分离

### 2. 选择相应文档生成规则

根据项目类型和文档需求选择对应的文档生成规则：

#### 项目使用文档
- **README.md**：使用 [references/readme.md](references/readme.md)

#### 项目依赖图
- **Dependency.mmd**：使用 [references/dependency.mmd.md](references/dependency.mmd.md)

#### 网络接口文档
- **API.md**：使用 [references/api.md](references/api.md)

#### 页面接口文档
- **Page.md**：使用 [references/page.md](references/page.md)

### 3. 生成文档

根据选择的文档规则，生成相应的文档：
- 分析项目代码和上下文
- 按照文档模板生成内容
- 确保文档的完整性和准确性

### 4. 输出文档

将生成的文档保存到项目相应位置：
- README.md → 项目根目录
- Dependency.mmd → 项目根目录或 docs 目录
- API.md → 项目根目录或 docs 目录
- Page.md → 项目根目录或 docs 目录

## 注意事项

1. **时效性**：确保文档内容与代码保持一致
2. **准确性**：文档中的信息必须准确无误
3. **完整性**：文档应包含所有必要的信息
4. **可读性**：文档应清晰易懂，结构合理
5. **一致性**：文档风格应保持一致
