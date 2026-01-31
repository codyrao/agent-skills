# MySQL 查询操作规范

## 概述

本文档定义 MySQL 数据库查询操作的最佳实践和规范，包括单记录查询和批量查询。查询操作虽然相对安全，但不规范的查询可能导致性能问题和数据库负载过高。

## 单记录查询

### 基本语法

```sql
SELECT column1, column2, ...
FROM table_name
WHERE condition;
```

### 规范要求

#### 1. 使用主键或唯一索引查询

- 单记录查询必须使用主键或唯一索引
- 确保查询性能最优
- 避免全表扫描

```sql
-- 推荐：使用主键查询
SELECT id, username, email
FROM users
WHERE id = 1001;

-- 推荐：使用唯一索引查询
SELECT id, username, email
FROM users
WHERE email = 'user@example.com';

-- 不推荐：使用非索引字段
SELECT id, username, email
FROM users
WHERE username = 'zhangsan';
```

#### 2. 只查询需要的字段

- 禁止使用 `SELECT *`
- 只查询业务需要的字段
- 减少网络传输和内存消耗

```sql
-- 推荐：只查询需要的字段
SELECT id, username, email
FROM users
WHERE id = 1001;

-- 不推荐：查询所有字段
SELECT *
FROM users
WHERE id = 1001;
```

#### 3. 使用 LIMIT 1

- 单记录查询必须添加 LIMIT 1
- 明确表达查询意图
- 防止意外返回多条记录

```sql
-- 推荐：添加 LIMIT 1
SELECT id, username, email
FROM users
WHERE email = 'user@example.com'
LIMIT 1;
```

#### 4. 处理空结果

- 代码层必须处理查询结果为空的情况
- 区分"记录不存在"和"查询错误"
- 避免空指针异常

```sql
-- 查询示例
SELECT id, username, email
FROM users
WHERE id = 1001;

-- 代码层处理（伪代码）
-- if (result == null) {
--     // 记录不存在，友好提示
-- } else {
--     // 处理查询结果
-- }
```

## 批量查询

### 基本语法

```sql
SELECT column1, column2, ...
FROM table_name
WHERE condition
LIMIT offset, count;
```

### 规范要求

#### 1. 必须限制返回行数

- 批量查询必须添加 LIMIT 限制
- 单次查询最多返回 1000 条记录
- 大数据量查询使用分页

```sql
-- 推荐：使用 LIMIT 限制
SELECT id, username, email
FROM users
WHERE status = 1
LIMIT 100;

-- 推荐：分页查询
SELECT id, username, email
FROM users
WHERE status = 1
LIMIT 0, 100;  -- 第一页，100条

SELECT id, username, email
FROM users
WHERE status = 1
LIMIT 100, 100;  -- 第二页，100条
```

#### 2. 分页优化

- 大数据量分页使用游标分页
- 避免深分页性能问题
- 推荐使用基于主键的分页

```sql
-- 不推荐：深分页（越往后越慢）
SELECT id, username, email
FROM users
WHERE status = 1
LIMIT 100000, 100;

-- 推荐：基于主键的游标分页
SELECT id, username, email
FROM users
WHERE status = 1 AND id > 100000
ORDER BY id
LIMIT 100;
```

#### 3. 使用覆盖索引

- 查询字段尽量在索引中
- 避免回表查询
- 提高查询性能

```sql
-- 假设有索引 idx_status_username (status, username)
-- 推荐：覆盖索引查询
SELECT status, username
FROM users
WHERE status = 1;

-- 不推荐：需要回表查询
SELECT id, username, email, phone
FROM users
WHERE status = 1;
```

#### 4. 避免大偏移量

- OFFSET 过大时性能急剧下降
- 使用 WHERE 条件替代大 OFFSET
- 或者使用搜索引擎（ES）处理大数据量查询

```sql
-- 不推荐：大偏移量
SELECT * FROM users LIMIT 1000000, 100;

-- 推荐：使用 WHERE 条件
SELECT * FROM users
WHERE id > 1000000
LIMIT 100;
```

## 关联查询

### 规范要求

#### 1. 控制 JOIN 表数量

- 单条 SQL 最多 JOIN 3 个表
- 过多 JOIN 影响性能和可读性
- 复杂查询拆分为多条简单查询

```sql
-- 推荐：最多 3 个表 JOIN
SELECT u.id, u.username, o.order_no, p.product_name
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN products p ON o.product_id = p.id
WHERE u.id = 1001;

-- 不推荐：过多表 JOIN
SELECT *
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN products p ON o.product_id = p.id
JOIN categories c ON p.category_id = c.id
JOIN suppliers s ON p.supplier_id = s.id
-- ... 更多 JOIN
```

#### 2. 确保关联字段有索引

- JOIN 条件字段必须有索引
- 外键字段自动创建索引
- 定期检查执行计划

```sql
-- 查看执行计划
EXPLAIN SELECT u.id, u.username, o.order_no
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.id = 1001;
```

#### 3. 避免笛卡尔积

- 确保 JOIN 条件正确
- 忘记 JOIN 条件会导致笛卡尔积
- 结果集数量爆炸式增长

```sql
-- 错误：忘记 JOIN 条件（笛卡尔积）
SELECT * FROM users u JOIN orders o;

-- 正确：添加 JOIN 条件
SELECT * FROM users u JOIN orders o ON u.id = o.user_id;
```

## 排序和分组

### 规范要求

#### 1. 排序优化

- 排序字段必须有索引
- 避免大数据量排序
- 优先使用索引排序

```sql
-- 推荐：使用索引字段排序
SELECT id, username, created_at
FROM users
WHERE status = 1
ORDER BY created_at DESC
LIMIT 100;

-- 不推荐：无索引字段排序（filesort）
SELECT id, username, last_login
FROM users
WHERE status = 1
ORDER BY last_login DESC
LIMIT 100;
```

#### 2. 分组优化

- 分组字段必须有索引
- 避免大数据量分组
- 使用临时表优化复杂分组

```sql
-- 推荐：使用索引字段分组
SELECT status, COUNT(*) as count
FROM users
GROUP BY status;

-- 查看执行计划
EXPLAIN SELECT status, COUNT(*) as count
FROM users
GROUP BY status;
```

#### 3. 避免 SELECT * + GROUP BY

- GROUP BY 查询只查询需要的字段
- SELECT * + GROUP BY 可能导致错误或非预期结果

```sql
-- 不推荐：SELECT * + GROUP BY
SELECT * FROM users GROUP BY status;

-- 推荐：只查询分组字段和聚合函数
SELECT status, COUNT(*) as count, MAX(created_at) as latest
FROM users
GROUP BY status;
```

## 性能优化

### 1. 使用 EXPLAIN 分析

- 所有查询必须使用 EXPLAIN 分析执行计划
- 关注 type、key、rows、Extra 字段
- 优化目标：type 达到 range 及以上

```sql
EXPLAIN SELECT id, username
FROM users
WHERE status = 1 AND created_at > '2024-01-01';

-- 关注字段：
-- type: 访问类型（system > const > eq_ref > ref > range > index > ALL）
-- key: 使用的索引
-- rows: 扫描行数
-- Extra: 额外信息（Using index 表示覆盖索引）
```

### 2. 索引优化

- WHERE 条件字段建立索引
- ORDER BY 字段建立索引
- JOIN 条件字段建立索引
- 定期分析和优化索引

```sql
-- 查看表索引
SHOW INDEX FROM users;

-- 分析表
ANALYZE TABLE users;

-- 优化表
OPTIMIZE TABLE users;
```

### 3. 查询缓存（MySQL 8.0 已移除）

- MySQL 8.0 已移除查询缓存
- 使用应用层缓存（Redis）
- 使用数据库连接池

### 4. 慢查询优化

```sql
-- 开启慢查询日志
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;

-- 查看慢查询日志
SHOW VARIABLES LIKE 'slow_query_log%';

-- 分析慢查询
-- 使用 pt-query-digest 工具分析
```

## 阿里云 RDS 特殊建议

### 1. 只读实例

- 查询操作优先使用只读实例
- 分担主库压力
- 提高查询性能

```sql
-- 应用层配置读写分离
-- 写操作 -> 主库
-- 读操作 -> 只读实例
```

### 2. 连接池

- 使用数据库连接池
- 控制连接数
- 避免连接泄漏

```sql
-- 查看当前连接数
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Max_used_connections';
```

### 3. 监控指标

- 关注 QPS（每秒查询数）
- 监控慢查询数量
- 观察 CPU 使用率
- 注意内存使用情况

## 字节跳动规范补充

### 1. 强制索引使用

- 重要查询强制使用指定索引
- 避免优化器选择错误的索引

```sql
-- 强制使用索引
SELECT id, username
FROM users FORCE INDEX (idx_status)
WHERE status = 1;

-- 忽略索引
SELECT id, username
FROM users IGNORE INDEX (idx_status)
WHERE status = 1;
```

### 2. 查询超时设置

- 设置查询超时时间
- 避免慢查询影响整体性能

```sql
-- 设置会话级超时（秒）
SET SESSION max_execution_time = 10000;  -- 10秒

-- 或者在 SQL 中指定（MySQL 8.0）
SELECT /*+ MAX_EXECUTION_TIME(10000) */ *
FROM users
WHERE status = 1;
```

### 3. 数据脱敏

- 敏感字段查询需要脱敏
- 日志中不能输出敏感数据
- 应用层实现脱敏逻辑

```sql
-- 不推荐：直接查询敏感字段
SELECT id, username, phone, id_card
FROM users
WHERE id = 1001;

-- 推荐：应用层脱敏处理
-- phone: 138****8888
-- id_card: 110101********1234
```

## 常见问题

### 1. 查询慢

```sql
-- 可能原因：
-- 1. 没有索引
-- 2. 索引未生效
-- 3. 数据量过大
-- 4. 查询语句复杂

-- 解决方案：
-- 1. 添加索引
ALTER TABLE users ADD INDEX idx_status (status);

-- 2. 分析执行计划
EXPLAIN SELECT * FROM users WHERE status = 1;

-- 3. 优化查询语句
-- 只查询需要的字段，添加 LIMIT 限制
```

### 2. 死锁

```sql
-- 查询导致死锁
-- 原因：并发事务循环等待锁

-- 查看死锁日志
SHOW ENGINE INNODB STATUS;

-- 解决方案：
-- 1. 按相同顺序访问表
-- 2. 减小事务粒度
-- 3. 使用乐观锁
```

### 3. 主从延迟

```sql
-- 写入后立即读取可能读到旧数据
-- 解决方案：
-- 1. 强制读主库
-- 2. 延迟读取
-- 3. 使用缓存
```

## 总结

### 单记录查询 checklist

- [ ] 使用主键或唯一索引查询
- [ ] 只查询需要的字段（禁止 SELECT *）
- [ ] 添加 LIMIT 1
- [ ] 代码层处理空结果
- [ ] 使用 EXPLAIN 分析执行计划

### 批量查询 checklist

- [ ] 添加 LIMIT 限制（单次 ≤1000 条）
- [ ] 使用分页查询
- [ ] 深分页使用游标分页
- [ ] 优先使用覆盖索引
- [ ] 避免大偏移量

### 关联查询 checklist

- [ ] 最多 JOIN 3 个表
- [ ] 确保关联字段有索引
- [ ] 避免笛卡尔积
- [ ] 使用 EXPLAIN 分析执行计划

### 排序和分组 checklist

- [ ] 排序字段有索引
- [ ] 分组字段有索引
- [ ] 避免大数据量排序/分组
- [ ] 禁止 SELECT * + GROUP BY

### 性能优化 checklist

- [ ] 使用 EXPLAIN 分析所有查询
- [ ] 确保 WHERE 条件字段有索引
- [ ] 定期分析和优化索引
- [ ] 开启慢查询日志
- [ ] 使用只读实例分担查询压力
- [ ] 使用连接池管理连接
