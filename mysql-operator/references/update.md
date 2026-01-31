# MySQL 更新操作规范

## 概述
本文档定义 MySQL 数据库更新操作的最佳实践和规范，包括单记录更新和批量更新。遵循 MySQL 官方规范、阿里云 RDS 操作手册和字节跳动 MySQL 操作规范。

## 安全警告

⚠️ **更新操作是高风险操作，必须谨慎执行**

- 忘记添加 WHERE 条件会导致全表更新
- 批量更新可能锁表，影响业务
- 更新前必须确认 WHERE 条件的准确性
- 生产环境更新前必须备份数据

## 单记录更新

### 基本语法
```sql
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

### 规范要求

#### 1. 必须包含 WHERE 条件
- **严禁不带 WHERE 条件的 UPDATE 语句**
- WHERE 条件必须能精确定位到单条记录
- 优先使用主键或唯一索引作为条件

```sql
-- 推荐：使用主键作为条件
UPDATE users
SET email = 'newemail@example.com', updated_at = NOW()
WHERE id = 1001;

-- 不推荐：使用非索引字段（可能更新多条）
UPDATE users
SET email = 'newemail@example.com'
WHERE username = 'zhangsan';

-- 严禁：不带 WHERE 条件
UPDATE users
SET status = 0;
```

#### 2. 使用 LIMIT 1 限制
- 即使使用主键，也建议添加 LIMIT 1
- 防止意外更新多条记录

```sql
-- 推荐：添加 LIMIT 1
UPDATE users
SET email = 'newemail@example.com', updated_at = NOW()
WHERE id = 1001
LIMIT 1;
```

#### 3. 先查询后更新
- 更新前先查询确认记录存在
- 确认更新前的数据状态
- 避免更新不存在的数据

```sql
-- 第一步：查询确认
SELECT * FROM users WHERE id = 1001;

-- 第二步：执行更新
UPDATE users
SET email = 'newemail@example.com', updated_at = NOW()
WHERE id = 1001;

-- 第三步：验证结果
SELECT * FROM users WHERE id = 1001;
```

#### 4. 更新时间戳
- 必须更新 updated_at 字段
- 使用 NOW() 或 CURRENT_TIMESTAMP
- 保持数据一致性

```sql
UPDATE users
SET email = 'newemail@example.com', updated_at = NOW()
WHERE id = 1001;
```

#### 5. 使用 REPLACE INTO 替代场景
- 当需要插入或更新时使用 ON DUPLICATE KEY UPDATE
- 避免 REPLACE INTO 导致自增 ID 变化

```sql
-- 推荐：使用 ON DUPLICATE KEY UPDATE
INSERT INTO user_settings (user_id, setting_key, setting_value)
VALUES (1001, 'theme', 'dark')
ON DUPLICATE KEY UPDATE
    setting_value = VALUES(setting_value),
    updated_at = NOW();
```

## 批量更新

### 基本语法
```sql
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

### 规范要求

#### 1. 批次大小控制
- 单条 UPDATE 语句影响行数不超过 1000 条
- 大批量更新应分批执行
- 每批提交一次，避免大事务

```sql
-- 推荐：分批更新，每批 500-1000 条
UPDATE users
SET status = 0, updated_at = NOW()
WHERE status = 1 AND created_at < '2023-01-01'
LIMIT 1000;
```

#### 2. 使用 JOIN 更新
- 多表关联更新时使用 JOIN
- 确保关联条件有索引

```sql
-- 使用 JOIN 更新
UPDATE users u
JOIN user_groups ug ON u.group_id = ug.id
SET u.group_name = ug.name, u.updated_at = NOW()
WHERE ug.status = 1;
```

#### 3. 使用 CASE WHEN 批量更新不同值
- 不同记录更新为不同值时使用 CASE WHEN
- 减少 SQL 语句数量

```sql
-- 使用 CASE WHEN 批量更新不同值
UPDATE products
SET price = CASE id
    WHEN 1 THEN 99.99
    WHEN 2 THEN 149.99
    WHEN 3 THEN 199.99
    ELSE price
END,
updated_at = NOW()
WHERE id IN (1, 2, 3);
```

#### 4. 事务控制
- 批量更新使用事务保证原子性
- 单事务不超过 10000 条记录
- 分批提交避免大事务

```sql
START TRANSACTION;

-- 第一批
UPDATE users SET status = 0 WHERE id BETWEEN 1 AND 1000;

-- 第二批
UPDATE users SET status = 0 WHERE id BETWEEN 1001 AND 2000;

COMMIT;
```

### 性能优化

#### 1. 索引优化
- 确保 WHERE 条件字段有索引
- 避免全表扫描
- 使用 EXPLAIN 分析执行计划

```sql
-- 查看执行计划
EXPLAIN UPDATE users
SET status = 0
WHERE created_at < '2023-01-01';
```

#### 2. 避免锁表
- 大批量更新会锁表
- 使用 LIMIT 分批更新
- 考虑使用 pt-online-schema-change（修改表结构时）

#### 3. 低峰期执行
- 大批量更新应在业务低峰期执行
- 避免影响线上业务
- 提前通知相关人员

### 阿里云 RDS 特殊建议

#### 1. 监控指标
- 关注 CPU 使用率
- 监控活跃连接数
- 观察锁等待情况
- 注意主从延迟

#### 2. 参数优化
```sql
-- 查看 innodb_lock_wait_timeout
SHOW VARIABLES LIKE 'innodb_lock_wait_timeout';

-- 临时调整（会话级）
SET SESSION innodb_lock_wait_timeout = 600; -- 10分钟
```

#### 3. 最佳实践
- 大批量更新前评估影响行数
- 使用 SELECT COUNT(*) 预估
- 准备回滚方案

```sql
-- 先查询预估影响行数
SELECT COUNT(*) FROM users
WHERE status = 1 AND created_at < '2023-01-01';

-- 确认后再执行更新
UPDATE users
SET status = 0, updated_at = NOW()
WHERE status = 1 AND created_at < '2023-01-01'
LIMIT 1000;
```

## 安全规范

### 1. 更新前备份
```sql
-- 创建备份表
CREATE TABLE users_backup_20240115 AS
SELECT * FROM users
WHERE status = 1 AND created_at < '2023-01-01';

-- 执行更新
UPDATE users
SET status = 0, updated_at = NOW()
WHERE status = 1 AND created_at < '2023-01-01';
```

### 2. 使用事务
```sql
START TRANSACTION;

UPDATE users
SET email = 'newemail@example.com', updated_at = NOW()
WHERE id = 1001;

-- 验证结果
SELECT * FROM users WHERE id = 1001;

-- 确认无误后提交
COMMIT;

-- 如有问题回滚
-- ROLLBACK;
```

### 3. 审计日志
- 记录所有更新操作
- 包含操作人、时间、影响行数
- 保留变更前后数据快照

## 字节跳动规范补充

### 1. 数据一致性
- 更新操作必须保证数据一致性
- 使用事务保证原子性
- 考虑缓存同步问题

```sql
-- 更新数据库后同步缓存（伪代码）
START TRANSACTION;

UPDATE users SET status = 0 WHERE id = 1001;

-- 业务逻辑：同步更新缓存
-- cache.set('user:1001:status', 0);

COMMIT;
```

### 2. 乐观锁实现
- 使用版本号实现乐观锁
- 防止并发更新冲突

```sql
-- 表结构添加 version 字段
ALTER TABLE users ADD COLUMN version INT DEFAULT 0 COMMENT '版本号';

-- 乐观锁更新
UPDATE users
SET email = 'newemail@example.com',
    version = version + 1,
    updated_at = NOW()
WHERE id = 1001 AND version = 5;

-- 检查影响行数，为 0 则表示并发冲突
```

### 3. 敏感字段更新
- 密码等敏感字段更新需要特殊处理
- 必须加密存储
- 记录安全审计日志

```sql
-- 密码更新示例
UPDATE users
SET password = SHA2('newpassword', 256),
    password_updated_at = NOW(),
    updated_at = NOW()
WHERE id = 1001;
```

## 常见问题

### 1. 锁等待超时
```sql
-- 错误：Lock wait timeout exceeded
-- 原因：事务持有锁时间过长
-- 解决：减小批次大小，分批提交

-- 查看锁等待
SHOW ENGINE INNODB STATUS;
```

### 2. 死锁
```sql
-- 错误：Deadlock found when trying to get lock
-- 原因：并发事务循环等待锁
-- 解决：按相同顺序访问表，减小事务粒度
```

### 3. 主从延迟
```sql
-- 大批量更新导致主从延迟
-- 解决：分批更新，监控延迟

-- 查看主从延迟
SHOW SLAVE STATUS\G;
```

## 总结

### 单记录更新 checklist
- [ ] 使用主键或唯一索引作为 WHERE 条件
- [ ] 添加 LIMIT 1
- [ ] 先查询确认记录存在
- [ ] 更新 updated_at 字段
- [ ] 使用事务保证原子性

### 批量更新 checklist
- [ ] 预估影响行数
- [ ] 分批执行，每批不超过 1000 条
- [ ] 确保 WHERE 条件字段有索引
- [ ] 使用 EXPLAIN 分析执行计划
- [ ] 业务低峰期执行
- [ ] 准备备份和回滚方案
- [ ] 监控数据库性能指标

### 严禁事项
- [ ] 严禁不带 WHERE 条件的 UPDATE
- [ ] 严禁在业务高峰期大批量更新
- [ ] 严禁单事务更新超过 10000 条记录
- [ ] 严禁更新前不备份重要数据
