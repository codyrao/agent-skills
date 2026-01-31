# MySQL 插入操作规范

## 概述
本文档定义 MySQL 数据库插入操作的最佳实践和规范，包括单记录插入和批量插入。遵循 MySQL 官方规范、阿里云 RDS 操作手册和字节跳动 MySQL 操作规范。

## 单记录插入

### 基本语法
```sql
INSERT INTO table_name (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);
```

### 规范要求

#### 1. 必须指定列名
- 禁止省略列名，必须显式指定要插入的列
- 避免表结构变更导致插入失败
- 提高代码可读性和可维护性

```sql
-- 推荐：显式指定列名
INSERT INTO users (username, email, created_at)
VALUES ('zhangsan', 'zhangsan@example.com', NOW());

-- 不推荐：省略列名
INSERT INTO users
VALUES (NULL, 'zhangsan', 'zhangsan@example.com', NOW());
```

#### 2. 使用 REPLACE INTO 注意事项
- REPLACE INTO 会先删除再插入，可能导致自增 ID 变化
- 谨慎使用，优先考虑 INSERT ... ON DUPLICATE KEY UPDATE

```sql
-- 推荐：使用 ON DUPLICATE KEY UPDATE
INSERT INTO users (id, username, email)
VALUES (1, 'zhangsan', 'zhangsan@example.com')
ON DUPLICATE KEY UPDATE
    username = VALUES(username),
    email = VALUES(email),
    updated_at = NOW();

-- 谨慎使用：REPLACE INTO
REPLACE INTO users (id, username, email)
VALUES (1, 'zhangsan', 'zhangsan@example.com');
```

#### 3. 默认值处理
- 显式设置默认值，不依赖数据库默认值
- 对于 created_at 等时间字段，显式使用 NOW() 或 CURRENT_TIMESTAMP

```sql
-- 推荐：显式设置所有值
INSERT INTO users (username, email, status, created_at)
VALUES ('zhangsan', 'zhangsan@example.com', 1, NOW());
```

#### 4. 数据类型检查
- 确保插入值的数据类型与字段定义匹配
- 字符串类型使用单引号包裹
- 数值类型不使用引号
- 日期时间类型使用标准格式

```sql
-- 正确示例
INSERT INTO orders (user_id, amount, order_date, status)
VALUES (1001, 99.99, '2024-01-15 10:30:00', 'pending');
```

### 性能优化

#### 1. 避免频繁单条插入
- 单条插入会产生大量日志和磁盘 I/O
- 频繁单条插入会触发频繁的刷盘操作
- 建议合并为批量插入

#### 2. 使用连接池
- 使用数据库连接池管理连接
- 避免频繁创建和关闭连接

#### 3. 事务控制
- 单条插入不需要显式事务
- 多条相关插入应使用事务保证原子性

```sql
START TRANSACTION;

INSERT INTO orders (user_id, amount, status)
VALUES (1001, 99.99, 'pending');

SET @order_id = LAST_INSERT_ID();

INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES (@order_id, 101, 2, 49.99);

COMMIT;
```

## 批量插入

### 基本语法
```sql
INSERT INTO table_name (column1, column2, column3, ...)
VALUES 
    (value1, value2, value3, ...),
    (value4, value5, value6, ...),
    (value7, value8, value9, ...);
```

### 规范要求

#### 1. 批次大小控制
- 单条 INSERT 语句最多 1000 条记录
- 单条 INSERT 语句数据包大小不超过 max_allowed_packet（默认 4MB）
- 建议批次大小：100-500 条

```sql
-- 推荐：每批 100-500 条
INSERT INTO users (username, email, created_at)
VALUES 
    ('user1', 'user1@example.com', NOW()),
    ('user2', 'user2@example.com', NOW()),
    ...
    ('user100', 'user100@example.com', NOW());
```

#### 2. 使用 LOAD DATA INFILE
- 大量数据插入时，优先使用 LOAD DATA INFILE
- 性能比 INSERT 快 10-20 倍
- 需要 FILE 权限

```sql
-- 从文件批量加载数据
LOAD DATA INFILE '/path/to/data.csv'
INTO TABLE users
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(username, email, created_at);
```

#### 3. 批量插入优化参数
```sql
-- 临时调整参数优化批量插入
SET autocommit = 0;
SET unique_checks = 0;
SET foreign_key_checks = 0;

-- 执行批量插入
INSERT INTO table_name ...
INSERT INTO table_name ...
INSERT INTO table_name ...

COMMIT;

-- 恢复参数
SET unique_checks = 1;
SET foreign_key_checks = 1;
SET autocommit = 1;
```

**注意事项：**
- 仅在空表或维护期间使用上述优化
- 生产环境有数据时谨慎使用 unique_checks = 0
- 操作完成后必须恢复参数

#### 4. 事务控制
- 大批量插入应分批提交，避免大事务
- 单事务建议不超过 10000 条记录
- 大事务会导致主从延迟、锁等待等问题

```sql
-- 分批提交示例
SET autocommit = 0;

-- 第一批
INSERT INTO users (username, email) VALUES ...; -- 500条
COMMIT;

-- 第二批
INSERT INTO users (username, email) VALUES ...; -- 500条
COMMIT;

-- 第三批
INSERT INTO users (username, email) VALUES ...; -- 500条
COMMIT;

SET autocommit = 1;
```

### 性能对比

| 插入方式 | 性能 | 适用场景 |
|---------|------|---------|
| 单条 INSERT | 基准 | 单条记录插入 |
| 批量 INSERT (100条) | 5-10x | 小批量数据插入 |
| 批量 INSERT (1000条) | 10-20x | 中批量数据插入 |
| LOAD DATA INFILE | 20-50x | 大批量数据导入 |

### 阿里云 RDS 特殊建议

#### 1. 参数优化
```sql
-- 查看当前 max_allowed_packet
SHOW VARIABLES LIKE 'max_allowed_packet';

-- 临时调整（会话级）
SET SESSION max_allowed_packet = 16777216; -- 16MB
```

#### 2. 监控指标
- 关注 CPU 使用率
- 监控磁盘 I/O
- 观察主从延迟
- 注意连接数

#### 3. 最佳实践
- 避免在业务高峰期执行大批量插入
- 大批量插入前通知 DBA
- 准备回滚方案

### 字节跳动规范补充

#### 1. 数据校验
- 插入前进行数据合法性校验
- 必填字段不能为空
- 字符串长度不能超过字段定义
- 数值范围符合业务逻辑

#### 2. 幂等性设计
- 重要数据插入考虑幂等性
- 使用唯一索引防止重复插入
- 业务层做去重处理

```sql
-- 使用唯一索引保证幂等性
INSERT INTO user_actions (user_id, action_type, action_date, count)
VALUES (1001, 'login', '2024-01-15', 1)
ON DUPLICATE KEY UPDATE
    count = count + 1,
    updated_at = NOW();
```

#### 3. 敏感数据处理
- 密码等敏感信息必须加密存储
- 使用参数化查询防止 SQL 注入
- 日志中不能输出敏感数据

```sql
-- 使用参数化查询（示例为伪代码）
-- 不推荐：直接拼接 SQL
-- INSERT INTO users (username, password) VALUES ('admin', '123456');

-- 推荐：使用参数化查询
INSERT INTO users (username, password) VALUES (?, ?);
-- 参数: ['admin', '加密后的密码']
```

## 常见问题

### 1. 主键冲突
```sql
-- 错误：Duplicate entry 'xxx' for key 'PRIMARY'
-- 解决：使用 INSERT IGNORE 或 ON DUPLICATE KEY UPDATE

INSERT IGNORE INTO users (id, username) VALUES (1, 'test');
-- 或
INSERT INTO users (id, username) VALUES (1, 'test')
ON DUPLICATE KEY UPDATE username = VALUES(username);
```

### 2. 数据包过大
```sql
-- 错误：Packet too large
-- 解决：减小批次大小或增大 max_allowed_packet

-- 查看当前限制
SHOW VARIABLES LIKE 'max_allowed_packet';

-- 临时调整
SET GLOBAL max_allowed_packet = 16777216;
```

### 3. 锁等待超时
```sql
-- 错误：Lock wait timeout exceeded
-- 解决：分批插入，减小事务大小
-- 检查并优化锁竞争
```

## 总结

### 单记录插入 checklist
- [ ] 显式指定列名
- [ ] 数据类型匹配
- [ ] 使用参数化查询
- [ ] 敏感数据加密

### 批量插入 checklist
- [ ] 批次大小 100-500 条
- [ ] 单事务不超过 10000 条
- [ ] 大批量使用 LOAD DATA INFILE
- [ ] 避免业务高峰期执行
- [ ] 监控数据库性能指标
