# MySQL 索引操作规范

## 概述

本文档定义 MySQL 数据库索引操作的最佳实践和规范，包括添加索引和删除索引。索引对数据库性能至关重要，不当的索引操作可能导致性能下降或锁表。

---

## 高风险操作声明

### 添加索引必须优先使用 Online DDL

**根据性能规范，添加索引时必须优先使用 Online DDL（ALGORITHM=INPLACE, LOCK=NONE），减少对业务的影响。**

### 删除索引操作必须手动执行

**根据安全规范，DROP INDEX 操作必须提示用户手动执行，禁止自动执行。**

原因：
- 删除索引可能导致查询性能急剧下降
- 误删索引可能影响业务稳定性
- 需要人工确认索引名和业务影响
- 需要确认无查询依赖

---

## 添加索引规范

### 基本语法

```sql
-- 添加普通索引
ALTER TABLE table_name
ADD INDEX index_name (column_name) COMMENT '索引描述';

-- 添加唯一索引
ALTER TABLE table_name
ADD UNIQUE INDEX index_name (column_name) COMMENT '索引描述';

-- 添加联合索引
ALTER TABLE table_name
ADD INDEX index_name (column1, column2, column3) COMMENT '索引描述';

-- 添加前缀索引
ALTER TABLE table_name
ADD INDEX index_name (column_name(length)) COMMENT '索引描述';
```

### 强制要求

#### 1. 必须优先使用 Online DDL

**添加索引时必须使用 Online DDL，这是强制要求！**

```sql
-- 推荐：使用 Online DDL
ALTER TABLE users
ADD INDEX idx_username (username) COMMENT '用户名索引',
ALGORITHM=INPLACE,
LOCK=NONE;

-- 不推荐：传统方式会锁表
ALTER TABLE users
ADD INDEX idx_username (username) COMMENT '用户名索引';
```

#### 2. 必须写好索引描述

**所有索引必须添加 COMMENT 注释，说明索引用途！**

```sql
-- 推荐：带描述的索引
ALTER TABLE users
ADD INDEX idx_status_created (status, created_at) COMMENT '状态+创建时间联合索引，用于用户列表查询';

-- 不推荐：无描述
ALTER TABLE users
ADD INDEX idx_status_created (status, created_at);
```

#### 3. 索引命名规范

- 普通索引：`idx_字段名` 或 `idx_字段1_字段2`
- 唯一索引：`uk_字段名` 或 `uk_字段1_字段2`
- 主键索引：`PRIMARY`（固定）
- 全文索引：`ft_字段名`

```sql
-- 普通索引
ALTER TABLE users ADD INDEX idx_username (username) COMMENT '用户名索引';

-- 唯一索引
ALTER TABLE users ADD UNIQUE INDEX uk_email (email) COMMENT '邮箱唯一索引';

-- 联合索引
ALTER TABLE users ADD INDEX idx_status_created (status, created_at) COMMENT '状态+创建时间联合索引';
```

### Online DDL 详解

#### Online DDL 算法

```sql
-- 方式 1：INPLACE 算法（推荐）
ALTER TABLE users
ADD INDEX idx_username (username) COMMENT '用户名索引',
ALGORITHM=INPLACE,
LOCK=NONE;

-- 方式 2：COPY 算法（需要复制表，不推荐）
ALTER TABLE users
ADD INDEX idx_username (username) COMMENT '用户名索引',
ALGORITHM=COPY,
LOCK=SHARED;
```

算法选择：
- `ALGORITHM=INPLACE`：原地构建索引，不复制表，性能最好
- `ALGORITHM=COPY`：复制表数据，需要额外磁盘空间，锁表时间长

#### Online DDL 锁级别

```sql
-- LOCK=NONE：无锁，允许并发读写（推荐）
ALTER TABLE users
ADD INDEX idx_username (username),
ALGORITHM=INPLACE,
LOCK=NONE;

-- LOCK=SHARED：共享锁，允许读不允许写
ALTER TABLE users
ADD INDEX idx_username (username),
ALGORITHM=INPLACE,
LOCK=SHARED;

-- LOCK=EXCLUSIVE：排他锁，不允许读写
ALTER TABLE users
ADD INDEX idx_username (username),
ALGORITHM=INPLACE,
LOCK=EXCLUSIVE;
```

锁级别选择：
- `LOCK=NONE`：首选，对业务影响最小
- `LOCK=SHARED`：允许查询，不允许写入
- `LOCK=EXCLUSIVE`：完全锁表，仅维护时使用

### 索引类型选择

#### 1. 普通索引（INDEX）

```sql
-- 用于加速查询
ALTER TABLE users
ADD INDEX idx_username (username) COMMENT '用户名索引，用于登录查询';
```

#### 2. 唯一索引（UNIQUE INDEX）

```sql
-- 用于保证数据唯一性
ALTER TABLE users
ADD UNIQUE INDEX uk_email (email) COMMENT '邮箱唯一索引';

-- 联合唯一索引
ALTER TABLE user_roles
ADD UNIQUE INDEX uk_user_role (user_id, role_id) COMMENT '用户角色联合唯一索引';
```

#### 3. 主键索引（PRIMARY KEY）

```sql
-- 建表时定义
CREATE TABLE users (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    -- 其他字段...
    PRIMARY KEY (id)
) ENGINE=InnoDB;

-- 或者后续添加
ALTER TABLE users
ADD PRIMARY KEY (id);
```

#### 4. 前缀索引

```sql
-- 长字符串字段使用前缀索引
ALTER TABLE users
ADD INDEX idx_email_prefix (email(10)) COMMENT '邮箱前缀索引';

-- 查看选择性
SELECT 
    COUNT(DISTINCT email) / COUNT(*) as full_selectivity,
    COUNT(DISTINCT LEFT(email, 10)) / COUNT(*) as prefix_selectivity
FROM users;
```

### 联合索引设计

#### 最左前缀原则

```sql
-- 联合索引 (a, b, c) 可以支持以下查询：
-- WHERE a = ?
-- WHERE a = ? AND b = ?
-- WHERE a = ? AND b = ? AND c = ?
-- WHERE a = ? AND c = ?（部分使用）
-- 但不支持：
-- WHERE b = ?
-- WHERE c = ?
-- WHERE b = ? AND c = ?

ALTER TABLE users
ADD INDEX idx_status_type_created (status, type, created_at) COMMENT '状态+类型+创建时间联合索引';
```

#### 字段顺序原则

```sql
-- 原则 1：等值查询字段放在前面
-- 原则 2：区分度高的字段放在前面
-- 原则 3：范围查询字段放在后面

-- 推荐：status（等值）+ type（等值）+ created_at（范围）
ALTER TABLE orders
ADD INDEX idx_status_type_created (status, type, created_at) COMMENT '订单查询索引';

-- 查询示例
SELECT * FROM orders
WHERE status = 1 AND type = 'A' AND created_at > '2024-01-01';
```

### 添加索引流程

```sql
-- 步骤 1：分析查询需求
-- 确定需要优化的查询语句

-- 步骤 2：查看现有索引
SHOW INDEX FROM users;

-- 步骤 3：使用 EXPLAIN 分析查询
EXPLAIN SELECT * FROM users WHERE username = 'zhangsan';

-- 步骤 4：评估索引大小（大表需要谨慎）
SELECT 
    table_name,
    ROUND((data_length + index_length) / 1024 / 1024, 2) AS 'size_mb'
FROM information_schema.tables
WHERE table_name = 'users';

-- 步骤 5：添加索引（使用 Online DDL）
ALTER TABLE users
ADD INDEX idx_username (username) COMMENT '用户名索引',
ALGORITHM=INPLACE,
LOCK=NONE;

-- 步骤 6：验证索引
SHOW INDEX FROM users;

-- 步骤 7：再次使用 EXPLAIN 验证
EXPLAIN SELECT * FROM users WHERE username = 'zhangsan';
```

---

## 删除索引规范

### 基本语法

```sql
-- 删除索引
ALTER TABLE table_name DROP INDEX index_name;

-- 或者
DROP INDEX index_name ON table_name;
```

### 高风险警告

⚠️ **DROP INDEX 是高风险操作，必须严格遵守以下规范！**

### 手动执行警告模板

```
⚠️⚠️⚠️ 高风险操作警告 ⚠️⚠️⚠️

【操作类型】：删除索引（DROP INDEX）
【风险等级】：高
【影响范围】：查询性能可能急剧下降

【必须确认事项】：
1. ✅ 确认索引名：idx_old_field
2. ✅ 确认表名：users
3. ✅ 确认数据库：production_db
4. ✅ 确认索引无查询依赖（已检查慢查询日志）
5. ✅ 确认索引无唯一性约束（非唯一索引）
6. ✅ 了解性能影响（已评估）
7. ✅ 有回滚方案（可重新创建索引）
8. ✅ 已通知相关团队成员
9. ✅ 获得上级授权

【建议操作】：
此操作必须手动执行，请勿自动执行。

请在 MySQL 客户端中手动执行以下步骤：

步骤 1 - 确认索引信息：
SHOW INDEX FROM users;
SHOW CREATE TABLE users;

步骤 2 - 检查索引使用：
-- 查看索引大小
SELECT 
    INDEX_NAME,
    ROUND(INDEX_LENGTH / 1024 / 1024, 2) AS 'index_size_mb'
FROM information_schema.statistics
WHERE TABLE_NAME = 'users' AND INDEX_NAME = 'idx_old_field';

-- 查看慢查询日志（确认无此索引的依赖查询）
-- 需要开启慢查询日志并分析

步骤 3 - 备份索引信息：
-- 记录索引定义，便于回滚
SHOW CREATE TABLE users;

步骤 4 - 执行删除（谨慎执行）：
ALTER TABLE users DROP INDEX idx_old_field;
-- 或者
DROP INDEX idx_old_field ON users;

步骤 5 - 验证删除：
SHOW INDEX FROM users;

步骤 6 - 监控性能：
-- 观察业务指标和慢查询
-- 如有问题立即回滚（重新创建索引）

执行前请再次确认以上所有事项！
```

### 删除索引前检查清单

#### 1. 检查索引信息

```sql
-- 查看索引详情
SHOW INDEX FROM users;

-- 查看索引大小
SELECT 
    INDEX_NAME,
    SEQ_IN_INDEX,
    COLUMN_NAME,
    ROUND(INDEX_LENGTH / 1024 / 1024, 2) AS 'index_size_mb'
FROM information_schema.statistics
WHERE TABLE_NAME = 'users';
```

#### 2. 检查索引使用（MySQL 8.0）

```sql
-- 查看索引使用统计（MySQL 8.0）
SELECT 
    OBJECT_SCHEMA,
    OBJECT_NAME,
    INDEX_NAME,
    COUNT_FETCH,
    COUNT_INSERT,
    COUNT_UPDATE,
    COUNT_DELETE
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE OBJECT_NAME = 'users';
```

#### 3. 检查慢查询日志

```sql
-- 确保慢查询日志开启
SHOW VARIABLES LIKE 'slow_query_log%';
SHOW VARIABLES LIKE 'long_query_time';

-- 分析慢查询（使用 pt-query-digest 工具）
-- pt-query-digest /var/log/mysql/slow.log
```

#### 4. 检查唯一性约束

```sql
-- 确认不是唯一索引
SHOW INDEX FROM users WHERE INDEX_NAME = 'idx_old_field';

-- 如果是唯一索引，需要确认数据唯一性
SELECT column_name, COUNT(*) as count
FROM users
GROUP BY column_name
HAVING count > 1;
```

### 安全删除流程

```sql
-- 步骤 1：标记索引废弃（添加注释）
-- MySQL 不支持直接修改索引注释，需要重建

-- 步骤 2：观察一段时间（如 7 天），监控性能

-- 步骤 3：确认无误后，删除索引
ALTER TABLE users DROP INDEX idx_old_field;

-- 步骤 4：持续监控性能
```

---

## 索引优化建议

### 1. 索引设计原则

#### 应该创建索引的场景

```sql
-- 1. 主键自动创建索引
PRIMARY KEY (id)

-- 2. 外键字段应该创建索引
ALTER TABLE orders ADD INDEX idx_user_id (user_id) COMMENT '用户ID索引';

-- 3. 频繁查询的字段
ALTER TABLE users ADD INDEX idx_username (username) COMMENT '用户名索引';

-- 4. WHERE 条件字段
ALTER TABLE users ADD INDEX idx_status (status) COMMENT '状态索引';

-- 5. ORDER BY 字段
ALTER TABLE users ADD INDEX idx_created_at (created_at) COMMENT '创建时间索引';

-- 6. GROUP BY 字段
ALTER TABLE users ADD INDEX idx_status (status) COMMENT '状态分组索引';
```

#### 不应该创建索引的场景

```sql
-- 1. 数据量小的表（< 1000 行）
-- 2. 频繁更新的字段（索引维护成本高）
-- 3. 区分度低的字段（如性别、状态只有0和1）
-- 4. 很少查询的字段
-- 5. 大字段（TEXT、BLOB）
```

### 2. 索引维护

```sql
-- 查看索引统计信息
SHOW INDEX FROM users;

-- 分析表（更新索引统计）
ANALYZE TABLE users;

-- 优化表（整理索引碎片）
OPTIMIZE TABLE users;

-- 查看索引碎片率
SELECT 
    table_name,
    index_name,
    ROUND(data_free / 1024 / 1024, 2) AS 'fragment_mb'
FROM information_schema.tables
WHERE table_name = 'users';
```

### 3. 索引监控

```sql
-- 查看未使用的索引（MySQL 8.0）
SELECT 
    OBJECT_SCHEMA,
    OBJECT_NAME,
    INDEX_NAME
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE INDEX_NAME IS NOT NULL
AND COUNT_FETCH = 0
AND OBJECT_SCHEMA NOT IN ('mysql', 'performance_schema', 'sys');

-- 查看重复索引
-- 使用 pt-duplicate-key-checker 工具
-- pt-duplicate-key-checker -u root -p -h localhost -d database_name
```

---

## 阿里云 RDS 特殊建议

### 1. 大表加索引

- 大表（超过 1GB）添加索引需要谨慎
- 使用 Online DDL 减少锁表时间
- 在业务低峰期执行
- 提前评估执行时间

```sql
-- 预估加索引时间（大表）
-- 使用 pt-online-schema-change 工具
-- pt-online-schema-change --alter "ADD INDEX idx_name (column)" D=database,t=table --execute
```

### 2. 索引大小监控

```sql
-- 查看索引占用空间
SELECT 
    table_name,
    index_name,
    ROUND(SUM(index_length) / 1024 / 1024, 2) AS 'index_size_mb'
FROM information_schema.statistics
WHERE table_schema = 'production_db'
GROUP BY table_name, index_name
ORDER BY index_size_mb DESC;
```

### 3. 参数优化

```sql
-- 查看 InnoDB 缓冲池大小
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';

-- 临时调整排序缓冲区（大索引创建）
SET SESSION sort_buffer_size = 16777216;  -- 16MB
SET SESSION read_buffer_size = 8388608;   -- 8MB
```

---

## 字节跳动规范补充

### 1. 索引命名规范

```sql
-- 统一命名格式
-- 单字段：idx_字段名
-- 多字段：idx_字段1_字段2_字段3
-- 唯一索引：uk_字段名
-- 主键：PRIMARY

-- 示例
ALTER TABLE users ADD INDEX idx_username (username) COMMENT '用户名索引';
ALTER TABLE users ADD INDEX idx_status_created (status, created_at) COMMENT '状态+创建时间索引';
ALTER TABLE users ADD UNIQUE INDEX uk_email (email) COMMENT '邮箱唯一索引';
```

### 2. 索引审批流程

- 生产环境添加索引需要审批
- 大表加索引需要 DBA 审核
- 索引添加前需要在测试环境验证

### 3. 索引性能测试

```sql
-- 步骤 1：记录当前性能
EXPLAIN ANALYZE SELECT * FROM users WHERE username = 'test';

-- 步骤 2：添加索引
ALTER TABLE users ADD INDEX idx_username (username);

-- 步骤 3：对比性能
EXPLAIN ANALYZE SELECT * FROM users WHERE username = 'test';

-- 步骤 4：如无改善，考虑删除索引
```

---

## 总结

### 添加索引 checklist

- [ ] 优先使用 Online DDL（ALGORITHM=INPLACE, LOCK=NONE）
- [ ] 写好索引 COMMENT 描述
- [ ] 遵循索引命名规范
- [ ] 使用 EXPLAIN 分析查询需求
- [ ] 评估索引大小（大表谨慎）
- [ ] 在业务低峰期执行
- [ ] 验证索引效果
- [ ] 监控性能指标

### 删除索引 checklist

- [ ] 确认索引无查询依赖
- [ ] 检查慢查询日志
- [ ] 确认索引大小
- [ ] 备份索引定义
- [ ] 获得上级授权
- [ ] 手动执行，禁止自动执行
- [ ] 删除后持续监控性能

### 严禁事项

- [ ] 严禁不使用 Online DDL 添加索引（大表）
- [ ] 严禁自动执行 DROP INDEX
- [ ] 严禁未检查依赖就删除索引
- [ ] 严禁添加索引不写 COMMENT
- [ ] 严禁大表高峰期执行 DDL
- [ ] 严禁删除唯一索引不检查数据唯一性
