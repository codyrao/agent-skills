# MySQL 删除操作规范

## 概述

本文档定义 MySQL 数据库删除操作的最佳实践和规范，包括单记录删除和多记录删除。删除操作是**极高风险操作**，必须严格遵守本规范。

⚠️ **警告：删除操作不可逆，一旦执行无法恢复！**

---

## 高风险操作声明

### 删除表记录操作必须手动执行

**根据安全规范，所有 DELETE 操作必须提示用户手动执行，禁止自动执行。**

原因：
- DELETE 操作会永久删除数据
- 误删数据可能导致严重业务损失
- 需要人工确认 WHERE 条件的准确性
- 需要确认已备份数据

---

## 单记录删除

### 基本语法

```sql
DELETE FROM table_name
WHERE condition;
```

### 规范要求

#### 1. 必须包含精确的 WHERE 条件

- **严禁不带 WHERE 条件的 DELETE 语句**
- 必须使用主键或唯一索引精确定位单条记录
- WHERE 条件必须能确保只删除一条记录

```sql
-- 推荐：使用主键删除单条记录
DELETE FROM users
WHERE id = 1001;

-- 不推荐：使用非唯一条件（可能删除多条）
DELETE FROM users
WHERE username = 'zhangsan';

-- 严禁：不带 WHERE 条件
DELETE FROM users;
```

#### 2. 必须使用 LIMIT 1

- 即使使用主键，也必须添加 LIMIT 1
- 防止意外删除多条记录
- 作为最后一道安全防线

```sql
-- 推荐：添加 LIMIT 1
DELETE FROM users
WHERE id = 1001
LIMIT 1;
```

#### 3. 先查询确认

- 删除前必须先查询确认记录内容
- 确认是要删除的目标记录
- 记录删除前的数据状态

```sql
-- 第一步：查询确认
SELECT * FROM users WHERE id = 1001;

-- 第二步：确认无误后执行删除
DELETE FROM users
WHERE id = 1001
LIMIT 1;

-- 第三步：验证删除结果
SELECT * FROM users WHERE id = 1001;
```

#### 4. 使用软删除替代硬删除

**强烈推荐使用软删除，避免物理删除数据。**

```sql
-- 推荐：软删除（更新状态）
UPDATE users
SET is_deleted = 1,
    deleted_at = NOW(),
    updated_at = NOW()
WHERE id = 1001;

-- 不推荐：硬删除
DELETE FROM users
WHERE id = 1001;
```

软删除优势：
- 数据可恢复
- 保留历史数据
- 便于审计追踪
- 避免误删风险

---

## 多记录删除

### 基本语法

```sql
DELETE FROM table_name
WHERE condition;
```

### 规范要求

#### 1. 必须预估影响行数

- 删除前必须使用 SELECT COUNT(*) 预估影响行数
- 确认影响范围符合预期
- 大批量删除需要分批执行

```sql
-- 第一步：预估影响行数
SELECT COUNT(*) FROM users
WHERE status = 0 AND last_login < '2023-01-01';

-- 第二步：查看样本数据
SELECT * FROM users
WHERE status = 0 AND last_login < '2023-01-01'
LIMIT 10;

-- 第三步：确认无误后分批删除
DELETE FROM users
WHERE status = 0 AND last_login < '2023-01-01'
LIMIT 1000;
```

#### 2. 批次大小控制

- 单条 DELETE 语句最多删除 1000 条记录
- 大批量删除必须分批执行
- 每批删除后确认结果再继续

```sql
-- 分批删除示例
-- 第一批
DELETE FROM users
WHERE status = 0 AND last_login < '2023-01-01'
LIMIT 1000;

-- 确认删除结果
SELECT COUNT(*) FROM users
WHERE status = 0 AND last_login < '2023-01-01';

-- 第二批
DELETE FROM users
WHERE status = 0 AND last_login < '2023-01-01'
LIMIT 1000;
```

#### 3. 使用 JOIN 删除

- 多表关联删除时使用 JOIN
- 确保关联条件正确
- 先 SELECT 确认再 DELETE

```sql
-- 先查询确认
SELECT u.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL AND u.created_at < '2023-01-01';

-- 确认无误后删除
DELETE u
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL AND u.created_at < '2023-01-01'
LIMIT 1000;
```

#### 4. 事务控制

- 批量删除使用事务
- 分批提交，避免大事务
- 单事务不超过 10000 条记录

```sql
START TRANSACTION;

-- 第一批
DELETE FROM users
WHERE status = 0
LIMIT 1000;

-- 确认结果
SELECT COUNT(*) FROM users WHERE status = 0;

-- 第二批
DELETE FROM users
WHERE status = 0
LIMIT 1000;

COMMIT;
```

---

## 手动执行警告模板

### 删除单条记录

```
⚠️⚠️⚠️ 高风险操作警告 ⚠️⚠️⚠️

【操作类型】：删除单条记录
【风险等级】：高
【影响范围】：单条记录永久删除

【必须确认事项】：
1. ✅ 已备份数据
2. ✅ 确认删除对象：表名 = users, 主键 = 1001
3. ✅ 已查询确认记录内容
4. ✅ 了解业务影响
5. ✅ 有回滚方案（软删除）

【建议操作】：
此操作必须手动执行，请勿自动执行。

请在 MySQL 客户端中手动执行以下步骤：

步骤 1 - 查询确认：
SELECT * FROM users WHERE id = 1001;

步骤 2 - 执行删除（推荐使用软删除）：
-- 方案 A：软删除（推荐）
UPDATE users
SET is_deleted = 1, deleted_at = NOW(), updated_at = NOW()
WHERE id = 1001;

-- 方案 B：硬删除（谨慎使用）
DELETE FROM users
WHERE id = 1001
LIMIT 1;

步骤 3 - 验证结果：
SELECT * FROM users WHERE id = 1001;

执行前请再次确认以上事项。
```

### 删除多条记录

```
⚠️⚠️⚠️ 高风险操作警告 ⚠️⚠️⚠️

【操作类型】：删除多条记录
【风险等级】：极高
【影响范围】：多条记录永久删除

【必须确认事项】：
1. ✅ 已完整备份数据
2. ✅ 预估影响行数：约 5000 条
3. ✅ 已查询样本数据确认
4. ✅ 了解业务影响
5. ✅ 有回滚方案

【建议操作】：
此操作必须手动执行，请勿自动执行。

请在 MySQL 客户端中手动执行以下步骤：

步骤 1 - 预估影响：
SELECT COUNT(*) FROM users
WHERE status = 0 AND last_login < '2023-01-01';

步骤 2 - 查看样本：
SELECT * FROM users
WHERE status = 0 AND last_login < '2023-01-01'
LIMIT 10;

步骤 3 - 分批删除（每批 1000 条）：
-- 第一批
DELETE FROM users
WHERE status = 0 AND last_login < '2023-01-01'
LIMIT 1000;

-- 检查剩余数量
SELECT COUNT(*) FROM users
WHERE status = 0 AND last_login < '2023-01-01';

-- 第二批
DELETE FROM users
WHERE status = 0 AND last_login < '2023-01-01'
LIMIT 1000;

-- 重复直到删除完成

执行前请再次确认以上事项。
```

---

## 数据备份规范

### 删除前必须备份

```sql
-- 方法 1：创建备份表
CREATE TABLE users_backup_20240115 AS
SELECT * FROM users
WHERE status = 0 AND last_login < '2023-01-01';

-- 添加备份标记
ALTER TABLE users_backup_20240115
ADD COLUMN backup_time TIMESTAMP DEFAULT NOW(),
ADD COLUMN backup_reason VARCHAR(255) DEFAULT '批量删除前备份';

-- 执行删除
DELETE FROM users
WHERE status = 0 AND last_login < '2023-01-01'
LIMIT 1000;
```

```sql
-- 方法 2：导出数据
SELECT * FROM users
WHERE status = 0 AND last_login < '2023-01-01'
INTO OUTFILE '/backup/users_to_delete_20240115.csv'
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n';
```

---

## 阿里云 RDS 特殊建议

### 1. 参数优化

```sql
-- 查看当前超时设置
SHOW VARIABLES LIKE 'innodb_lock_wait_timeout';

-- 大批量删除时临时调整
SET SESSION innodb_lock_wait_timeout = 600;
```

### 2. 监控指标

- 关注 CPU 使用率
- 监控活跃连接数
- 观察锁等待情况
- 注意主从延迟

### 3. 最佳实践

- 大批量删除应在业务低峰期执行
- 删除前通知 DBA
- 准备数据恢复方案
- 记录删除操作日志

---

## 字节跳动规范补充

### 1. 强制软删除策略

**所有业务表必须实现软删除，禁止物理删除。**

```sql
-- 标准软删除表结构
CREATE TABLE users (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    -- 其他业务字段...
    is_deleted TINYINT NOT NULL DEFAULT 0 COMMENT '是否删除：0-未删除，1-已删除',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted_at TIMESTAMP NULL DEFAULT NULL COMMENT '删除时间',
    PRIMARY KEY (id),
    KEY idx_is_deleted (is_deleted),
    KEY idx_deleted_at (deleted_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';

-- 软删除操作
UPDATE users
SET is_deleted = 1,
    deleted_at = NOW(),
    updated_at = NOW()
WHERE id = 1001;

-- 查询时过滤已删除数据
SELECT * FROM users
WHERE is_deleted = 0 AND id = 1001;
```

### 2. 数据归档策略

对于历史数据，采用归档而非删除：

```sql
-- 创建归档表
CREATE TABLE users_archive LIKE users;

-- 迁移历史数据到归档表
INSERT INTO users_archive
SELECT * FROM users
WHERE created_at < '2022-01-01';

-- 确认迁移成功后删除原表数据
DELETE FROM users
WHERE created_at < '2022-01-01'
LIMIT 1000;
```

### 3. 审计日志

所有删除操作必须记录审计日志：

```sql
-- 审计日志表
CREATE TABLE audit_log (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    operation_type VARCHAR(50) NOT NULL COMMENT '操作类型：DELETE/UPDATE/INSERT',
    table_name VARCHAR(100) NOT NULL COMMENT '表名',
    primary_key_value VARCHAR(255) COMMENT '主键值',
    old_data JSON COMMENT '变更前数据',
    new_data JSON COMMENT '变更后数据',
    operator VARCHAR(100) NOT NULL COMMENT '操作人',
    operation_time TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    KEY idx_table_name (table_name),
    KEY idx_operation_time (operation_time)
) ENGINE=InnoDB COMMENT='审计日志表';

-- 记录删除日志（触发器实现）
DELIMITER //
CREATE TRIGGER trg_users_delete
BEFORE DELETE ON users
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (operation_type, table_name, primary_key_value, old_data, operator)
    VALUES ('DELETE', 'users', OLD.id, JSON_OBJECT('id', OLD.id, 'username', OLD.username), CURRENT_USER());
END//
DELIMITER ;
```

---

## 常见问题

### 1. 误删数据恢复

```sql
-- 如果有备份表
INSERT INTO users
SELECT * FROM users_backup_20240115
WHERE id = 1001;

-- 如果是软删除
UPDATE users
SET is_deleted = 0,
    deleted_at = NULL,
    updated_at = NOW()
WHERE id = 1001;
```

### 2. 删除速度慢

```sql
-- 原因 1：没有索引
-- 解决：为 WHERE 条件字段添加索引
ALTER TABLE users ADD INDEX idx_status_created_at (status, created_at);

-- 原因 2：大批量删除
-- 解决：分批删除，每批 1000 条
```

### 3. 锁等待超时

```sql
-- 错误：Lock wait timeout exceeded
-- 解决：减小批次大小，分批提交

-- 查看锁情况
SHOW ENGINE INNODB STATUS;
```

---

## 总结

### 删除操作 checklist

- [ ] 优先考虑软删除，避免物理删除
- [ ] 删除前必须完整备份数据
- [ ] 使用主键或唯一索引作为 WHERE 条件
- [ ] 必须添加 LIMIT 限制
- [ ] 先查询确认再删除
- [ ] 预估影响行数
- [ ] 大批量删除分批执行（每批 ≤1000 条）
- [ ] 业务低峰期执行
- [ ] 监控数据库性能指标
- [ ] 记录审计日志

### 严禁事项

- [ ] 严禁不带 WHERE 条件的 DELETE
- [ ] 严禁自动执行 DELETE 操作
- [ ] 严禁在业务高峰期大批量删除
- [ ] 严禁单事务删除超过 10000 条记录
- [ ] 严禁删除前不备份
- [ ] 严禁不使用 LIMIT 限制

### 手动执行确认清单

执行 DELETE 操作前，必须确认：
1. ✅ 数据已备份
2. ✅ 影响范围已确认
3. ✅ WHERE 条件已验证
4. ✅ 业务影响已评估
5. ✅ 回滚方案已准备
6. ✅ 执行时间已选择（低峰期）
7. ✅ 相关人员已通知
