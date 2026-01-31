# MySQL 字段操作规范

## 概述

本文档定义 MySQL 数据库字段操作的最佳实践和规范，包括添加字段和删除字段。字段操作会影响表结构和现有数据，必须严格遵守规范。

---

## 高风险操作声明

### 删除字段操作必须手动执行

**根据安全规范，DROP COLUMN 操作必须提示用户手动执行，禁止自动执行。**

原因：
- DROP COLUMN 会永久删除字段和所有数据
- 误删字段可能导致数据丢失和业务异常
- 需要人工确认字段名和业务影响
- 需要确认无业务依赖

---

## 添加字段规范

### 基本语法

```sql
-- 添加单个字段
ALTER TABLE table_name
ADD COLUMN column_name datatype constraints COMMENT '字段描述';

-- 添加字段到指定位置
ALTER TABLE table_name
ADD COLUMN column_name datatype constraints COMMENT '字段描述' AFTER existing_column;

-- 添加多个字段
ALTER TABLE table_name
ADD COLUMN column1 datatype constraints COMMENT '描述1',
ADD COLUMN column2 datatype constraints COMMENT '描述2';
```

### 强制要求

#### 1. 必须写好字段描述

**所有添加的字段必须添加 COMMENT 注释，说明字段用途！**

```sql
-- 推荐：添加带注释的字段
ALTER TABLE users
ADD COLUMN phone VARCHAR(20) DEFAULT NULL COMMENT '手机号';

-- 不推荐：无注释
ALTER TABLE users
ADD COLUMN phone VARCHAR(20) DEFAULT NULL;
```

#### 2. 设置合理的默认值

- 必须为字段设置默认值
- 避免现有数据出现 NULL（除非业务需要）
- 考虑业务兼容性

```sql
-- 推荐：设置默认值
ALTER TABLE users
ADD COLUMN status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-禁用，1-启用';

-- 添加字段并更新现有数据
ALTER TABLE users
ADD COLUMN level INT DEFAULT NULL COMMENT '用户等级';

-- 更新现有数据
UPDATE users SET level = 1 WHERE level IS NULL;

-- 修改为非空（如果需要）
ALTER TABLE users
MODIFY COLUMN level INT NOT NULL DEFAULT 1 COMMENT '用户等级';
```

#### 3. 选择合适的位置

- 新字段建议添加到表末尾
- 或者添加到相关业务字段之后
- 避免频繁修改字段顺序

```sql
-- 推荐：添加到表末尾（默认）
ALTER TABLE users
ADD COLUMN nickname VARCHAR(50) DEFAULT NULL COMMENT '昵称';

-- 推荐：添加到指定字段之后
ALTER TABLE users
ADD COLUMN nickname VARCHAR(50) DEFAULT NULL COMMENT '昵称' AFTER username;

-- 不推荐：添加到第一列（影响表结构）
ALTER TABLE users
ADD COLUMN nickname VARCHAR(50) DEFAULT NULL COMMENT '昵称' FIRST;
```

### 在线 DDL（重要）

**添加字段时使用 Online DDL，减少对业务的影响！**

```sql
-- 推荐：使用 Online DDL
ALTER TABLE users
ADD COLUMN nickname VARCHAR(50) DEFAULT NULL COMMENT '昵称',
ALGORITHM=INPLACE,
LOCK=NONE;

-- 不推荐：传统方式会锁表
ALTER TABLE users
ADD COLUMN nickname VARCHAR(50) DEFAULT NULL COMMENT '昵称';
```

Online DDL 参数说明：
- `ALGORITHM=INPLACE`：原地算法，不复制表
- `LOCK=NONE`：无锁，允许并发读写
- `LOCK=SHARED`：共享锁，允许读不允许写
- `LOCK=EXCLUSIVE`：排他锁，不允许读写

### 添加字段流程

```sql
-- 步骤 1：检查表当前结构
DESCRIBE users;

-- 步骤 2：评估影响（大表需要谨慎）
SELECT COUNT(*) FROM users;
SHOW TABLE STATUS LIKE 'users';

-- 步骤 3：添加字段（使用 Online DDL）
ALTER TABLE users
ADD COLUMN nickname VARCHAR(50) DEFAULT NULL COMMENT '昵称' AFTER username,
ALGORITHM=INPLACE,
LOCK=NONE;

-- 步骤 4：验证添加结果
DESCRIBE users;
SHOW CREATE TABLE users;

-- 步骤 5：更新现有数据（如果需要）
UPDATE users SET nickname = username WHERE nickname IS NULL;
```

### 批量添加字段

```sql
-- 一次添加多个字段（减少 ALTER 次数）
ALTER TABLE users
ADD COLUMN nickname VARCHAR(50) DEFAULT NULL COMMENT '昵称' AFTER username,
ADD COLUMN avatar VARCHAR(255) DEFAULT NULL COMMENT '头像URL' AFTER nickname,
ADD COLUMN gender TINYINT DEFAULT NULL COMMENT '性别：0-女，1-男，2-保密' AFTER avatar,
ALGORITHM=INPLACE,
LOCK=NONE;
```

---

## 删除字段规范

### 基本语法

```sql
-- 删除单个字段
ALTER TABLE table_name
DROP COLUMN column_name;

-- 删除多个字段
ALTER TABLE table_name
DROP COLUMN column1,
DROP COLUMN column2;
```

### 高风险警告

⚠️ **DROP COLUMN 是极高风险操作，必须严格遵守以下规范！**

### 手动执行警告模板

```
⚠️⚠️⚠️ 极高风险操作警告 ⚠️⚠️⚠️

【操作类型】：删除字段（DROP COLUMN）
【风险等级】：极高
【影响范围】：字段和所有数据永久删除

【必须确认事项】：
1. ✅ 已完整备份表数据
2. ✅ 确认字段名：phone
3. ✅ 确认表名：users
4. ✅ 确认数据库：production_db
5. ✅ 确认无业务依赖（无代码引用、无索引依赖）
6. ✅ 确认无外键依赖
7. ✅ 了解业务影响（已评估）
8. ✅ 有数据恢复方案
9. ✅ 已通知相关团队成员
10. ✅ 获得上级授权

【建议操作】：
此操作必须手动执行，请勿自动执行。

请在 MySQL 客户端中手动执行以下步骤：

步骤 1 - 确认字段信息：
DESCRIBE users;
SHOW CREATE TABLE users;

步骤 2 - 检查字段数据：
SELECT phone, COUNT(*) as count
FROM users
WHERE phone IS NOT NULL
GROUP BY phone
LIMIT 10;

步骤 3 - 检查依赖：
-- 检查是否有索引依赖此字段
SHOW INDEX FROM users WHERE Column_name = 'phone';

-- 检查是否有外键依赖此字段
SELECT * FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE REFERENCED_COLUMN_NAME = 'phone'
AND REFERENCED_TABLE_NAME = 'users';

-- 检查存储过程和函数
SELECT * FROM INFORMATION_SCHEMA.ROUTINES
WHERE ROUTINE_DEFINITION LIKE '%phone%';

步骤 4 - 备份数据：
-- 方式 1：导出表结构+数据
mysqldump -u username -p production_db users > users_backup_20240115.sql

-- 方式 2：备份字段数据到临时表
CREATE TABLE users_phone_backup AS
SELECT id, phone FROM users WHERE phone IS NOT NULL;

步骤 5 - 执行删除（谨慎执行）：
ALTER TABLE users DROP COLUMN phone;

步骤 6 - 验证删除：
DESCRIBE users;

执行前请再次确认以上所有事项！
此操作不可逆，一旦执行无法恢复！
```

### 删除字段前检查清单

#### 1. 检查字段数据

```sql
-- 查看字段统计信息
SELECT 
    COUNT(*) as total_count,
    COUNT(phone) as non_null_count,
    COUNT(DISTINCT phone) as unique_count
FROM users;

-- 查看字段样本数据
SELECT id, phone FROM users WHERE phone IS NOT NULL LIMIT 10;
```

#### 2. 检查索引依赖

```sql
-- 检查字段是否有索引
SHOW INDEX FROM users WHERE Column_name = 'phone';

-- 检查字段是否是主键或唯一索引的一部分
SHOW INDEX FROM users;
```

#### 3. 检查外键依赖

```sql
-- 检查其他表是否依赖此字段
SELECT 
    TABLE_SCHEMA,
    TABLE_NAME,
    COLUMN_NAME,
    CONSTRAINT_NAME
FROM
    INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE
    REFERENCED_TABLE_NAME = 'users'
    AND REFERENCED_COLUMN_NAME = 'phone';
```

#### 4. 检查存储过程和函数

```sql
-- 检查存储过程是否引用此字段
SELECT 
    ROUTINE_NAME,
    ROUTINE_TYPE
FROM
    INFORMATION_SCHEMA.ROUTINES
WHERE
    ROUTINE_DEFINITION LIKE '%phone%'
    AND ROUTINE_SCHEMA = 'production_db';
```

#### 5. 检查视图

```sql
-- 检查视图是否引用此字段
SELECT 
    TABLE_NAME,
    VIEW_DEFINITION
FROM
    INFORMATION_SCHEMA.VIEWS
WHERE
    VIEW_DEFINITION LIKE '%phone%'
    AND TABLE_SCHEMA = 'production_db';
```

#### 6. 检查触发器

```sql
-- 检查触发器是否引用此字段
SELECT 
    TRIGGER_NAME,
    ACTION_STATEMENT
FROM
    INFORMATION_SCHEMA.TRIGGERS
WHERE
    ACTION_STATEMENT LIKE '%phone%'
    AND EVENT_OBJECT_TABLE = 'users';
```

### 安全删除流程

```sql
-- 步骤 1：重命名字段（而非直接删除）
ALTER TABLE users
CHANGE COLUMN phone phone_deprecated VARCHAR(20) DEFAULT NULL COMMENT '已废弃：手机号';

-- 步骤 2：观察一段时间（如 7 天），确认业务正常

-- 步骤 3：确认无误后，删除字段
ALTER TABLE users DROP COLUMN phone_deprecated;
```

### 在线 DDL 删除字段

```sql
-- 使用 Online DDL 删除字段
ALTER TABLE users
DROP COLUMN phone,
ALGORITHM=INPLACE,
LOCK=NONE;
```

---

## 修改字段规范

### 基本语法

```sql
-- 修改字段类型
ALTER TABLE table_name
MODIFY COLUMN column_name new_datatype constraints COMMENT '新描述';

-- 修改字段名和类型
ALTER TABLE table_name
CHANGE COLUMN old_name new_name datatype constraints COMMENT '新描述';
```

### 修改字段注意事项

#### 1. 数据类型转换风险

```sql
-- 危险：可能导致数据截断或丢失
ALTER TABLE users
MODIFY COLUMN age VARCHAR(10) COMMENT '年龄';

-- 安全：扩大字段长度
ALTER TABLE users
MODIFY COLUMN username VARCHAR(100) COMMENT '用户名';
```

#### 2. 使用 Online DDL

```sql
-- 推荐：使用 Online DDL 修改字段
ALTER TABLE users
MODIFY COLUMN username VARCHAR(100) NOT NULL COMMENT '用户名',
ALGORITHM=INPLACE,
LOCK=NONE;
```

#### 3. 修改字段名

```sql
-- 修改字段名（谨慎操作）
ALTER TABLE users
CHANGE COLUMN phone mobile VARCHAR(20) DEFAULT NULL COMMENT '手机号';
```

---

## 字段设计最佳实践

### 1. 字段命名规范

- 使用小写字母
- 单词间用下划线分隔
- 字段名应具有描述性
- 避免使用 MySQL 保留字

```sql
-- 推荐
ALTER TABLE users ADD COLUMN user_name VARCHAR(50) COMMENT '用户名';
ALTER TABLE users ADD COLUMN created_time TIMESTAMP COMMENT '创建时间';

-- 不推荐
ALTER TABLE users ADD COLUMN userName VARCHAR(50);  -- 驼峰命名
ALTER TABLE users ADD COLUMN select VARCHAR(50);    -- 保留字
```

### 2. 字段类型选择

| 场景 | 推荐类型 | 说明 |
|------|---------|------|
| 主键 | BIGINT UNSIGNED | 自增ID |
| 状态 | TINYINT | 0-255范围 |
| 金额 | DECIMAL(10,2) | 精确计算 |
| 短文本 | VARCHAR(255) | 可变长度 |
| 长文本 | TEXT | 大文本 |
| 时间戳 | TIMESTAMP | 自动更新 |
| 日期 | DATE | 年月日 |
| 布尔值 | TINYINT(1) | 0或1 |

### 3. 默认值设置

```sql
-- 状态字段默认启用
ALTER TABLE users ADD COLUMN status TINYINT NOT NULL DEFAULT 1 COMMENT '状态';

-- 时间字段默认当前时间
ALTER TABLE users ADD COLUMN created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间';

-- 可空字段默认NULL
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMP NULL DEFAULT NULL COMMENT '删除时间';
```

### 4. 字段顺序建议

```sql
-- 推荐字段顺序：
-- 1. 主键 id
-- 2. 业务字段（按业务逻辑分组）
-- 3. 状态字段
-- 4. 时间戳字段（created_at, updated_at）
-- 5. 软删除字段（is_deleted, deleted_at）

CREATE TABLE example (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    
    -- 业务字段
    name VARCHAR(50) NOT NULL COMMENT '名称',
    description TEXT COMMENT '描述',
    
    -- 状态字段
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态',
    
    -- 时间戳
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    
    -- 软删除
    is_deleted TINYINT NOT NULL DEFAULT 0 COMMENT '是否删除',
    deleted_at TIMESTAMP NULL DEFAULT NULL COMMENT '删除时间',
    
    PRIMARY KEY (id)
) ENGINE=InnoDB COMMENT='示例表';
```

---

## 阿里云 RDS 特殊建议

### 1. 大表加字段

- 大表（超过 1GB）添加字段需要谨慎
- 使用 Online DDL 减少锁表时间
- 在业务低峰期执行

```sql
-- 大表添加字段
ALTER TABLE large_table
ADD COLUMN new_field VARCHAR(50) DEFAULT NULL COMMENT '新字段',
ALGORITHM=INPLACE,
LOCK=NONE;
```

### 2. 监控 DDL 进度

```sql
-- 查看正在执行的 DDL
SHOW PROCESSLIST;

-- 查看 InnoDB 事务状态
SHOW ENGINE INNODB STATUS;
```

### 3. 参数优化

```sql
-- 临时调整锁等待超时（大表 DDL）
SET SESSION lock_wait_timeout = 3600;
SET SESSION innodb_lock_wait_timeout = 3600;
```

---

## 字节跳动规范补充

### 1. 字段变更审批

- 生产环境字段变更需要审批
- 重要字段变更需要 DBA 审核
- 变更前需要在测试环境验证

### 2. 灰度发布

- 新字段先灰度发布
- 观察一段时间后再全量
- 监控业务指标

```sql
-- 步骤 1：添加字段（可空）
ALTER TABLE users
ADD COLUMN new_feature_flag TINYINT DEFAULT NULL COMMENT '新功能标记';

-- 步骤 2：灰度更新数据
UPDATE users SET new_feature_flag = 1 WHERE id IN (1, 2, 3);

-- 步骤 3：观察业务

-- 步骤 4：全量更新
UPDATE users SET new_feature_flag = 0 WHERE new_feature_flag IS NULL;

-- 步骤 5：修改为 NOT NULL
ALTER TABLE users
MODIFY COLUMN new_feature_flag TINYINT NOT NULL DEFAULT 0 COMMENT '新功能标记';
```

### 3. 字段废弃流程

```sql
-- 步骤 1：标记字段废弃（添加注释）
ALTER TABLE users
MODIFY COLUMN old_field VARCHAR(50) DEFAULT NULL COMMENT '已废弃：原字段，请勿使用';

-- 步骤 2：代码中移除引用

-- 步骤 3：观察一段时间

-- 步骤 4：删除字段
ALTER TABLE users DROP COLUMN old_field;
```

---

## 总结

### 添加字段 checklist

- [ ] 写好字段 COMMENT 描述
- [ ] 设置合理的默认值
- [ ] 选择合适的字段位置
- [ ] 使用 Online DDL（ALGORITHM=INPLACE, LOCK=NONE）
- [ ] 大表在业务低峰期执行
- [ ] 更新现有数据（如果需要）
- [ ] 验证添加结果

### 删除字段 checklist

- [ ] 完整备份表数据
- [ ] 检查字段数据内容
- [ ] 检查索引依赖
- [ ] 检查外键依赖
- [ ] 检查存储过程和函数
- [ ] 检查视图
- [ ] 检查触发器
- [ ] 确认无业务依赖
- [ ] 获得上级授权
- [ ] 手动执行，禁止自动执行
- [ ] 优先使用重命名字段而非直接删除

### 修改字段 checklist

- [ ] 评估数据类型转换风险
- [ ] 检查数据截断可能性
- [ ] 使用 Online DDL
- [ ] 备份数据
- [ ] 在测试环境验证

### 严禁事项

- [ ] 严禁自动执行 DROP COLUMN
- [ ] 严禁无备份删除字段
- [ ] 严禁未确认依赖就删除字段
- [ ] 严禁添加字段不写 COMMENT
- [ ] 严禁大表高峰期执行 DDL
- [ ] 严禁不设置默认值（除非业务需要 NULL）
