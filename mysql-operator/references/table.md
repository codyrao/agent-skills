# MySQL 表操作规范

## 概述

本文档定义 MySQL 数据库表操作的最佳实践和规范，包括建表和删表。表操作是数据库结构的核心，必须严格遵守规范以确保数据完整性、性能和可维护性。

---

## 高风险操作声明

### 删除表操作必须手动执行

**根据安全规范，DROP TABLE 操作必须提示用户手动执行，禁止自动执行。**

原因：
- DROP TABLE 会永久删除表结构和所有数据
- 误删表可能导致灾难性业务损失
- 需要人工确认表名和备份状态
- 需要确认无业务依赖

---

## 建表规范

### 基本语法

```sql
CREATE TABLE table_name (
    column1 datatype constraints,
    column2 datatype constraints,
    ...
    table_constraints
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='表描述';
```

### 强制要求

#### 1. 必须包含 id bigint 主键

**所有表必须包含 id bigint 字段作为主键，这是强制要求！**

```sql
CREATE TABLE users (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    -- 其他字段...
    PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

主键规范：
- 类型：BIGINT UNSIGNED
- 属性：NOT NULL AUTO_INCREMENT
- 命名：id
- 注释：必须添加中文注释说明

#### 2. 必须写好字段描述

**所有字段必须添加 COMMENT 注释，说明字段用途！**

```sql
CREATE TABLE users (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    username VARCHAR(50) NOT NULL COMMENT '用户名',
    email VARCHAR(100) NOT NULL COMMENT '邮箱地址',
    phone VARCHAR(20) DEFAULT NULL COMMENT '手机号',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-禁用，1-启用',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_email (email) COMMENT '邮箱唯一索引'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

#### 3. 必须写好索引描述

**所有索引必须添加 COMMENT 注释，说明索引用途！**

```sql
CREATE TABLE orders (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    user_id BIGINT UNSIGNED NOT NULL COMMENT '用户ID',
    order_no VARCHAR(50) NOT NULL COMMENT '订单号',
    status TINYINT NOT NULL DEFAULT 0 COMMENT '订单状态：0-待支付，1-已支付，2-已发货，3-已完成',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_order_no (order_no) COMMENT '订单号唯一索引',
    KEY idx_user_id (user_id) COMMENT '用户ID索引，用于查询用户订单',
    KEY idx_status_created (status, created_at) COMMENT '状态+创建时间联合索引，用于订单列表查询'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='订单表';
```

#### 4. 必须包含时间戳字段

**所有表必须包含 created_at 和 updated_at 字段！**

```sql
CREATE TABLE products (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    -- 业务字段...
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='产品表';
```

### 完整建表示例

```sql
CREATE TABLE users (
    -- 主键
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    
    -- 业务字段
    username VARCHAR(50) NOT NULL COMMENT '用户名',
    email VARCHAR(100) NOT NULL COMMENT '邮箱地址',
    phone VARCHAR(20) DEFAULT NULL COMMENT '手机号',
    password VARCHAR(255) NOT NULL COMMENT '密码（加密存储）',
    avatar VARCHAR(255) DEFAULT NULL COMMENT '头像URL',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-禁用，1-启用，2-已注销',
    
    -- 时间戳
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    
    -- 主键约束
    PRIMARY KEY (id),
    
    -- 唯一索引
    UNIQUE KEY uk_username (username) COMMENT '用户名唯一索引',
    UNIQUE KEY uk_email (email) COMMENT '邮箱唯一索引',
    
    -- 普通索引
    KEY idx_status (status) COMMENT '状态索引，用于用户状态筛选',
    KEY idx_created_at (created_at) COMMENT '创建时间索引，用于按时间排序',
    
    -- 联合索引
    KEY idx_status_created (status, created_at) COMMENT '状态+创建时间联合索引，用于用户列表查询'
    
) ENGINE=InnoDB 
  DEFAULT CHARSET=utf8mb4 
  COLLATE=utf8mb4_unicode_ci
  COMMENT='用户表：存储用户基本信息';
```

### 字段类型规范

#### 数值类型

| 类型 | 用途 | 示例 |
|------|------|------|
| TINYINT | 小整数（-128~127） | status, is_deleted |
| SMALLINT | 较小整数（-32768~32767） | 短编码 |
| INT | 标准整数 | 普通ID |
| BIGINT | 大整数 | 主键ID，大数量 |
| DECIMAL | 精确小数 | 金额 |
| FLOAT/DOUBLE | 浮点数 | 科学计算（不推荐用于金额） |

```sql
-- 金额字段使用 DECIMAL
amount DECIMAL(10, 2) NOT NULL DEFAULT 0.00 COMMENT '订单金额',

-- 状态字段使用 TINYINT
status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-禁用，1-启用',

-- 主键使用 BIGINT
id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键ID',
```

#### 字符串类型

| 类型 | 用途 | 最大长度 |
|------|------|---------|
| CHAR | 固定长度 | 255 |
| VARCHAR | 可变长度 | 65535 |
| TEXT | 长文本 | 65535 |
| MEDIUMTEXT | 中等文本 | 16M |
| LONGTEXT | 长文本 | 4G |

```sql
-- 固定长度使用 CHAR
gender CHAR(1) COMMENT '性别：M-男，F-女',

-- 可变长度使用 VARCHAR
username VARCHAR(50) NOT NULL COMMENT '用户名',
email VARCHAR(100) NOT NULL COMMENT '邮箱',

-- 长文本使用 TEXT
description TEXT COMMENT '商品描述',
content MEDIUMTEXT COMMENT '文章内容',
```

#### 时间类型

| 类型 | 用途 | 格式 |
|------|------|------|
| DATE | 日期 | YYYY-MM-DD |
| TIME | 时间 | HH:MM:SS |
| DATETIME | 日期时间 | YYYY-MM-DD HH:MM:SS |
| TIMESTAMP | 时间戳 | YYYY-MM-DD HH:MM:SS |
| YEAR | 年份 | YYYY |

```sql
-- 创建时间使用 TIMESTAMP
created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',

-- 更新时间使用 TIMESTAMP
updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',

-- 生日等日期使用 DATE
birthday DATE DEFAULT NULL COMMENT '生日',

-- 需要精确时间使用 DATETIME
login_time DATETIME DEFAULT NULL COMMENT '登录时间',
```

### 引擎选择

**统一使用 InnoDB 引擎！**

```sql
-- 推荐：使用 InnoDB
CREATE TABLE users (...) ENGINE=InnoDB;

-- 不推荐：使用 MyISAM
CREATE TABLE users (...) ENGINE=MyISAM;
```

InnoDB 优势：
- 支持事务
- 支持行级锁
- 支持外键
- 崩溃恢复
- MVCC 并发控制

### 字符集规范

**统一使用 utf8mb4 字符集！**

```sql
-- 推荐：使用 utf8mb4
CREATE TABLE users (...) 
ENGINE=InnoDB 
DEFAULT CHARSET=utf8mb4 
COLLATE=utf8mb4_unicode_ci;
```

utf8mb4 优势：
- 支持完整的 Unicode 字符
- 支持 emoji 表情
- 兼容 utf8

### 表注释规范

**表必须有 COMMENT 注释，说明表用途！**

```sql
-- 推荐：详细的表注释
CREATE TABLE users (...) COMMENT='用户表：存储用户基本信息，包括用户名、邮箱、手机号等';

-- 不推荐：无注释或简单注释
CREATE TABLE users (...) COMMENT='用户表';
```

### 软删除设计

**所有业务表必须实现软删除！**

```sql
CREATE TABLE users (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    -- 业务字段...
    is_deleted TINYINT NOT NULL DEFAULT 0 COMMENT '是否删除：0-未删除，1-已删除',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted_at TIMESTAMP NULL DEFAULT NULL COMMENT '删除时间',
    PRIMARY KEY (id),
    KEY idx_is_deleted (is_deleted) COMMENT '软删除标记索引'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

---

## 删表规范

### 基本语法

```sql
DROP TABLE [IF EXISTS] table_name;
```

### 高风险警告

⚠️ **DROP TABLE 是极高风险操作，必须严格遵守以下规范！**

### 手动执行警告模板

```
⚠️⚠️⚠️ 极高风险操作警告 ⚠️⚠️⚠️

【操作类型】：删除表（DROP TABLE）
【风险等级】：极高
【影响范围】：表结构和所有数据永久删除

【必须确认事项】：
1. ✅ 已完整备份数据（逻辑备份 + 物理备份）
2. ✅ 确认表名：users
3. ✅ 确认数据库：production_db
4. ✅ 确认无业务依赖（无外键关联、无应用引用）
5. ✅ 了解业务影响（已评估）
6. ✅ 有数据恢复方案
7. ✅ 已通知相关团队成员
8. ✅ 获得上级授权

【建议操作】：
此操作必须手动执行，请勿自动执行。

请在 MySQL 客户端中手动执行以下步骤：

步骤 1 - 确认表信息：
SHOW CREATE TABLE users;
SELECT COUNT(*) FROM users;

步骤 2 - 备份数据：
-- 方式 1：导出数据
mysqldump -u username -p production_db users > users_backup_20240115.sql

-- 方式 2：创建备份表
CREATE TABLE users_backup_20240115 AS SELECT * FROM users;

步骤 3 - 确认无依赖：
-- 检查外键依赖
SELECT 
    TABLE_NAME,
    COLUMN_NAME,
    CONSTRAINT_NAME,
    REFERENCED_TABLE_NAME,
    REFERENCED_COLUMN_NAME
FROM
    INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE
    REFERENCED_TABLE_NAME = 'users';

-- 检查触发器
SHOW TRIGGERS LIKE 'users%';

步骤 4 - 执行删除（谨慎执行）：
DROP TABLE IF EXISTS users;

步骤 5 - 验证删除：
SHOW TABLES LIKE 'users%';

执行前请再次确认以上所有事项！
此操作不可逆，一旦执行无法恢复！
```

### 删表前检查清单

#### 1. 数据备份

```sql
-- 方式 1：逻辑备份（mysqldump）
-- mysqldump -u root -p db_name table_name > table_name_backup.sql

-- 方式 2：物理备份表
CREATE TABLE users_backup_20240115 AS SELECT * FROM users;

-- 方式 3：重命名表（快速回滚）
RENAME TABLE users TO users_to_be_dropped_20240115;
-- 如需要恢复：
-- RENAME TABLE users_to_be_dropped_20240115 TO users;
```

#### 2. 检查外键依赖

```sql
-- 检查其他表是否依赖此表
SELECT 
    TABLE_SCHEMA,
    TABLE_NAME,
    COLUMN_NAME,
    CONSTRAINT_NAME
FROM
    INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE
    REFERENCED_TABLE_NAME = 'users'
    AND REFERENCED_TABLE_SCHEMA = 'production_db';
```

#### 3. 检查触发器

```sql
-- 查看表相关触发器
SHOW TRIGGERS LIKE 'users%';

-- 查看所有触发器
SELECT 
    TRIGGER_NAME,
    EVENT_MANIPULATION,
    ACTION_STATEMENT
FROM
    INFORMATION_SCHEMA.TRIGGERS
WHERE
    EVENT_OBJECT_TABLE = 'users';
```

#### 4. 检查存储过程和函数

```sql
-- 检查存储过程是否引用此表
SELECT 
    ROUTINE_NAME,
    ROUTINE_TYPE,
    ROUTINE_DEFINITION
FROM
    INFORMATION_SCHEMA.ROUTINES
WHERE
    ROUTINE_DEFINITION LIKE '%users%'
    AND ROUTINE_SCHEMA = 'production_db';
```

#### 5. 检查视图

```sql
-- 检查视图是否引用此表
SELECT 
    TABLE_NAME,
    VIEW_DEFINITION
FROM
    INFORMATION_SCHEMA.VIEWS
WHERE
    VIEW_DEFINITION LIKE '%users%'
    AND TABLE_SCHEMA = 'production_db';
```

### 安全删表流程

```sql
-- 步骤 1：重命名表（而非直接删除）
RENAME TABLE users TO users_deprecated_20240115;

-- 步骤 2：观察一段时间（如 7 天），确认无问题

-- 步骤 3：确认无误后，删除备份表
DROP TABLE IF EXISTS users_deprecated_20240115;
```

---

## 表维护

### 1. 查看表结构

```sql
-- 查看建表语句
SHOW CREATE TABLE users;

-- 查看表字段
DESCRIBE users;
-- 或
SHOW COLUMNS FROM users;

-- 查看表索引
SHOW INDEX FROM users;
```

### 2. 修改表结构

```sql
-- 修改表注释
ALTER TABLE users COMMENT='用户表（已更新）';

-- 修改表引擎
ALTER TABLE users ENGINE=InnoDB;

-- 修改表字符集
ALTER TABLE users CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### 3. 表分析优化

```sql
-- 分析表
ANALYZE TABLE users;

-- 优化表
OPTIMIZE TABLE users;

-- 检查表
CHECK TABLE users;

-- 修复表
REPAIR TABLE users;
```

### 4. 查看表状态

```sql
-- 查看表统计信息
SHOW TABLE STATUS LIKE 'users';

-- 查看表大小
SELECT 
    table_name,
    ROUND((data_length + index_length) / 1024 / 1024, 2) AS 'size_mb'
FROM
    information_schema.tables
WHERE
    table_schema = 'production_db'
    AND table_name = 'users';
```

---

## 阿里云 RDS 特殊建议

### 1. 大表处理

- 大表（超过 1GB）避免在高峰期修改结构
- 使用 pt-online-schema-change 进行在线 DDL
- 提前评估修改时间和影响

### 2. 监控指标

- 监控表大小增长
- 关注碎片率
- 观察行数变化

### 3. 备份策略

- 定期备份重要表
- 使用 RDS 自动备份功能
- 重要操作前手动创建快照

---

## 字节跳动规范补充

### 1. 表命名规范

- 使用小写字母
- 单词间用下划线分隔
- 表名使用名词复数形式
- 避免使用 MySQL 保留字

```sql
-- 推荐
CREATE TABLE user_profiles (...);
CREATE TABLE order_items (...);

-- 不推荐
CREATE TABLE UserProfiles (...);
CREATE TABLE orderItems (...);
CREATE TABLE user (...);  -- 单数
CREATE TABLE select (...);  -- 保留字
```

### 2. 分表设计

- 大数据量表提前规划分表
- 使用一致性哈希或范围分片
- 分表键选择要均匀

```sql
-- 分表示例：users 表按 user_id % 100 分表
CREATE TABLE users_00 (...);
CREATE TABLE users_01 (...);
-- ...
CREATE TABLE users_99 (...);
```

### 3. 归档策略

- 历史数据定期归档
- 归档表命名规范：原表名_archive_年月
- 归档后数据迁移到冷存储

```sql
-- 创建归档表
CREATE TABLE users_archive_202401 LIKE users;

-- 迁移历史数据
INSERT INTO users_archive_202401
SELECT * FROM users
WHERE created_at < '2023-01-01';

-- 删除已归档数据
DELETE FROM users
WHERE created_at < '2023-01-01';
```

---

## 总结

### 建表 checklist

- [ ] 包含 id bigint 主键
- [ ] 所有字段添加 COMMENT 注释
- [ ] 所有索引添加 COMMENT 注释
- [ ] 包含 created_at 和 updated_at 字段
- [ ] 使用 InnoDB 引擎
- [ ] 使用 utf8mb4 字符集
- [ ] 表添加 COMMENT 注释
- [ ] 实现软删除（is_deleted + deleted_at）
- [ ] 字段类型选择合理
- [ ] 必要字段添加索引

### 删表 checklist

- [ ] 完整备份数据
- [ ] 检查外键依赖
- [ ] 检查触发器
- [ ] 检查存储过程和函数
- [ ] 检查视图
- [ ] 确认无业务依赖
- [ ] 获得上级授权
- [ ] 通知相关团队
- [ ] 手动执行，禁止自动执行
- [ ] 优先使用重命名而非直接删除

### 严禁事项

- [ ] 严禁自动执行 DROP TABLE
- [ ] 严禁无备份删表
- [ ] 严禁未确认依赖就删表
- [ ] 严禁建表不包含 id bigint 主键
- [ ] 严禁字段和索引无注释
- [ ] 严禁使用 MyISAM 引擎
- [ ] 严禁使用 utf8 字符集
