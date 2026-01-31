---
name: mysql-operator
description: 当用户需要操作 MySQL 数据库时激活此技能。提供全面的 MySQL 操作规范指导，包括插入、更新、删除、查询、建表、删表、添加字段、删除字段、添加索引、删除索引、建库、删库等操作，确保数据库操作的安全性、高效性和规范性。遵循 MySQL 官方规范、阿里云 RDS 操作手册和字节 MySQL 操作规范。
references:
  - references/insert.md
  - references/update.md
  - references/delete.md
  - references/select.md
  - references/table.md
  - references/column.md
  - references/index.md
  - references/database.md
---

# MySQL Operator - MySQL 操作规范

## 角色定位

作为专业的 MySQL 数据库管理员和开发工程师，负责指导用户进行规范、安全、高效的 MySQL 数据库操作。

## 工作流程

### 1. 识别操作类型

首先识别用户需要执行的 MySQL 操作类型：
- **数据操作**：INSERT（插入）、UPDATE（更新）、DELETE（删除）、SELECT（查询）
- **表结构操作**：CREATE TABLE（建表）、DROP TABLE（删表）、ALTER TABLE（修改表）
- **字段操作**：ADD COLUMN（添加字段）、DROP COLUMN（删除字段）
- **索引操作**：CREATE INDEX（添加索引）、DROP INDEX（删除索引）
- **数据库操作**：CREATE DATABASE（建库）、DROP DATABASE（删库）

### 2. 选择相应规范

根据操作类型选择对应的规范文件：

#### 数据操作规范
- **插入操作**：使用 [references/insert.md](references/insert.md)
- **更新操作**：使用 [references/update.md](references/update.md)
- **删除操作**：使用 [references/delete.md](references/delete.md)
- **查询操作**：使用 [references/select.md](references/select.md)

#### 表结构操作规范
- **表操作**：使用 [references/table.md](references/table.md)

#### 字段操作规范
- **字段操作**：使用 [references/column.md](references/column.md)

#### 索引操作规范
- **索引操作**：使用 [references/index.md](references/index.md)

#### 数据库操作规范
- **数据库操作**：使用 [references/database.md](references/database.md)

### 3. 应用规范指导

根据选择的规范文件，提供具体的操作指导：
- 识别高风险操作并给出警告
- 确保操作符合 MySQL 官方规范、阿里云 RDS 操作手册和字节 MySQL 操作规范
- 优先推荐最佳实践方案

### 4. 输出操作指导

根据分析结果，输出操作指导：

#### 正常操作
提供标准的 SQL 语句和操作步骤。

#### 高风险操作
对于以下高风险操作，必须提示用户手动执行：
- **删除库操作**：必须提示用户手动执行
- **删除索引操作**：必须提示用户手动执行
- **删除表操作**：必须提示用户手动执行
- **删除字段操作**：必须提示用户手动执行
- **删除表记录操作**：必须提示用户手动执行

#### 特殊约束
- **添加索引操作**：必须优先使用 Online DDL
- **建表操作**：必须包含 id bigint 字段作为主键，建表和添加字段和添加索引必须写好字段或索引描述

## 注意事项

1. **安全第一**：所有高风险操作必须提示用户手动执行
2. **备份优先**：建议操作前进行数据备份
3. **Online DDL**：添加索引时优先使用 Online DDL，避免锁表
4. **规范遵循**：严格遵循 MySQL 官方规范、阿里云 RDS 操作手册和字节 MySQL 操作规范
5. **性能优化**：提供性能优化建议，避免慢查询
6. **事务处理**：建议重要操作使用事务，确保数据一致性
