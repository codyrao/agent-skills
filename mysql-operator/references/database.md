# MySQL 数据库操作规范

## 概述

本文档定义 MySQL 数据库级别的操作最佳实践和规范，包括创建数据库和删除数据库。数据库操作是最高级别的操作，直接影响整个数据库实例。

---

## 高风险操作声明

### 删除数据库操作必须手动执行

**根据安全规范，DROP DATABASE 操作必须提示用户手动执行，禁止自动执行。**

原因：
- DROP DATABASE 会永久删除整个数据库及其所有表和数据
- 误删数据库可能导致灾难性业务损失
- 需要人工确认数据库名和备份状态
- 需要确认无业务依赖
- 需要最高级别的授权

---

## 创建数据库规范

### 基本语法

```sql
CREATE DATABASE [IF NOT EXISTS] database_name
CHARACTER SET charset_name
COLLATE collation_name;
```

### 规范要求

#### 1. 使用 utf8mb4 字符集

**所有数据库必须统一使用 utf8mb4 字符集！**

```sql
-- 推荐：使用 utf8mb4
CREATE DATABASE IF NOT EXISTS production_db
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

-- 不推荐：使用 utf8 或其他字符集
CREATE DATABASE IF NOT EXISTS production_db
CHARACTER SET utf8;
```

#### 2. 数据库命名规范

- 使用小写字母
- 单词间用下划线分隔
- 环境标识：开发（dev）、测试（test）、生产（prod）
- 避免使用 MySQL 保留字

```sql
-- 推荐
CREATE DATABASE IF NOT EXISTS myapp_prod_db CHARACTER SET utf8mb4;
CREATE DATABASE IF NOT EXISTS myapp_test_db CHARACTER SET utf8mb4;
CREATE DATABASE IF NOT EXISTS myapp_dev_db CHARACTER SET utf8mb4;

-- 不推荐
CREATE DATABASE IF NOT EXISTS MyAppDB;  -- 驼峰命名
CREATE DATABASE IF NOT EXISTS myapp;    -- 无环境标识
CREATE DATABASE IF NOT EXISTS database; -- 保留字
```

#### 3. 创建前检查

```sql
-- 检查数据库是否已存在
SHOW DATABASES LIKE 'myapp_prod_db';

-- 查看创建语句
SHOW CREATE DATABASE myapp_prod_db;
```

### 完整创建流程

```sql
-- 步骤 1：检查数据库是否存在
SHOW DATABASES LIKE 'myapp_prod_db';

-- 步骤 2：创建数据库
CREATE DATABASE IF NOT EXISTS myapp_prod_db
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

-- 步骤 3：验证创建
SHOW CREATE DATABASE myapp_prod_db;

-- 步骤 4：使用数据库
USE myapp_prod_db;

-- 步骤 5：查看当前数据库
SELECT DATABASE();
```

### 数据库权限设置

```sql
-- 创建专用用户
CREATE USER 'myapp_user'@'%' IDENTIFIED BY 'strong_password';

-- 授予数据库权限
GRANT SELECT, INSERT, UPDATE, DELETE ON myapp_prod_db.* TO 'myapp_user'@'%';

-- 刷新权限
FLUSH PRIVILEGES;

-- 查看用户权限
SHOW GRANTS FOR 'myapp_user'@'%';
```

### 生产环境建库 checklist

- [ ] 确认数据库命名符合规范
- [ ] 使用 utf8mb4 字符集
- [ ] 创建专用数据库用户
- [ ] 设置最小权限原则
- [ ] 记录数据库信息
- [ ] 配置备份策略
- [ ] 配置监控告警

---

## 删除数据库规范

### 基本语法

```sql
DROP DATABASE [IF EXISTS] database_name;
```

### 极高风险警告

⚠️ **DROP DATABASE 是最高风险操作，必须严格遵守以下规范！**

⚠️ **此操作会永久删除整个数据库及其所有内容，无法恢复！**

### 手动执行警告模板

```
⚠️⚠️⚠️ 灾难级风险操作警告 ⚠️⚠️⚠️

【操作类型】：删除数据库（DROP DATABASE）
【风险等级】：灾难级
【影响范围】：整个数据库及其所有表、数据、索引、视图、存储过程等全部永久删除

【必须确认事项】：
1. ✅ 已完整备份整个数据库（逻辑备份 + 物理备份）
2. ✅ 确认数据库名：myapp_prod_db
3. ✅ 确认数据库实例：production-mysql-01
4. ✅ 确认无业务依赖（所有应用已停止连接）
5. ✅ 确认无其他数据库依赖（无外键跨库关联）
6. ✅ 了解业务影响（已评估，业务已下线或迁移）
7. ✅ 有完整数据恢复方案（已测试可恢复）
8. ✅ 已通知所有相关团队成员
9. ✅ 已获得 CTO/技术VP 书面授权
10. ✅ 已安排在维护窗口期执行
11. ✅ 已准备回滚方案

【建议操作】：
此操作必须手动执行，绝对禁止自动执行！

请在 MySQL 客户端中手动执行以下步骤：

步骤 1 - 确认数据库信息：
SHOW DATABASES LIKE 'myapp_prod_db';
SHOW CREATE DATABASE myapp_prod_db;

步骤 2 - 统计数据库信息：
USE myapp_prod_db;
SHOW TABLES;
SELECT COUNT(*) FROM information_schema.tables 
WHERE table_schema = 'myapp_prod_db';

步骤 3 - 完整备份（多种方式）：
-- 方式 1：逻辑备份（mysqldump）
mysqldump -u root -p --single-transaction --routines --triggers \
  myapp_prod_db > myapp_prod_db_backup_20240115.sql

-- 方式 2：物理备份（如果有）
-- 使用 xtrabackup 或其他物理备份工具

-- 方式 3：RDS 快照（阿里云 RDS）
-- 在控制台手动创建快照

步骤 4 - 验证备份：
-- 检查备份文件大小
ls -lh myapp_prod_db_backup_20240115.sql

-- 测试恢复（在测试环境）
mysql -u root -p -e "CREATE DATABASE myapp_prod_db_test;"
mysql -u root -p myapp_prod_db_test < myapp_prod_db_backup_20240115.sql

步骤 5 - 停止业务连接：
-- 查看当前连接
SHOW PROCESSLIST;

-- 杀掉相关连接（谨慎操作）
-- KILL connection_id;

步骤 6 - 再次确认：
-- 最后一次确认数据库名
SELECT 'myapp_prod_db' AS database_to_drop;

步骤 7 - 执行删除（极度谨慎）：
DROP DATABASE IF EXISTS myapp_prod_db;

步骤 8 - 验证删除：
SHOW DATABASES LIKE 'myapp_prod_db';

步骤 9 - 清理相关资源：
-- 删除数据库用户（如果不再需要）
-- DROP USER 'myapp_user'@'%';

执行前请再次确认以上所有事项！
此操作不可逆，一旦执行无法恢复！
请三思而后行！
```

### 删除数据库前检查清单

#### 1. 完整备份数据库

```sql
-- 方式 1：mysqldump 逻辑备份
mysqldump -u root -p \
  --single-transaction \
  --routines \
  --triggers \
  --events \
  --hex-blob \
  --skip-lock-tables \
  myapp_prod_db > myapp_prod_db_backup_$(date +%Y%m%d_%H%M%S).sql

-- 方式 2：分表备份（大数据量）
mysqldump -u root -p myapp_prod_db table1 > table1_backup.sql
mysqldump -u root -p myapp_prod_db table2 > table2_backup.sql

-- 方式 3：阿里云 RDS 快照
-- 在 RDS 控制台创建手动快照
```

#### 2. 统计数据库信息

```sql
-- 查看数据库大小
SELECT 
    table_schema AS database_name,
    ROUND(SUM(data_length + index_length) / 1024 / 1024 / 1024, 2) AS 'size_gb',
    COUNT(*) AS 'table_count'
FROM information_schema.tables
WHERE table_schema = 'myapp_prod_db'
GROUP BY table_schema;

-- 查看所有表
USE myapp_prod_db;
SHOW TABLES;

-- 查看表数量
SELECT COUNT(*) AS table_count 
FROM information_schema.tables 
WHERE table_schema = 'myapp_prod_db';
```

#### 3. 检查业务依赖

```sql
-- 查看当前连接
SHOW PROCESSLIST;

-- 查看数据库连接（按用户分组）
SELECT 
    user,
    host,
    db,
    command,
    COUNT(*) as connection_count
FROM information_schema.processlist
WHERE db = 'myapp_prod_db'
GROUP BY user, host, db, command;
```

#### 4. 检查跨库依赖

```sql
-- 检查是否有跨库外键（MySQL 不支持，但应用层可能有逻辑关联）
-- 检查是否有跨库查询的视图
SELECT 
    TABLE_NAME,
    VIEW_DEFINITION
FROM information_schema.views
WHERE VIEW_DEFINITION LIKE '%myapp_prod_db%'
AND TABLE_SCHEMA != 'myapp_prod_db';
```

#### 5. 检查存储过程和函数

```sql
-- 查看数据库存储过程
SELECT 
    ROUTINE_NAME,
    ROUTINE_TYPE,
    CREATED,
    LAST_ALTERED
FROM information_schema.routines
WHERE ROUTINE_SCHEMA = 'myapp_prod_db';

-- 查看数据库事件
SELECT 
    EVENT_NAME,
    EVENT_DEFINITION,
    STATUS
FROM information_schema.events
WHERE EVENT_SCHEMA = 'myapp_prod_db';
```

### 安全删除流程

```sql
-- 步骤 1：重命名数据库（而非直接删除）
-- MySQL 不支持直接重命名数据库，需要导出导入

-- 步骤 2：导出数据
mysqldump -u root -p myapp_prod_db > myapp_prod_db_export.sql

-- 步骤 3：创建新数据库（标记为废弃）
CREATE DATABASE myapp_prod_db_deprecated_20240115 CHARACTER SET utf8mb4;

-- 步骤 4：导入到新数据库
mysql -u root -p myapp_prod_db_deprecated_20240115 < myapp_prod_db_export.sql

-- 步骤 5：删除原数据库
DROP DATABASE IF EXISTS myapp_prod_db;

-- 步骤 6：观察一段时间（如 30 天）

-- 步骤 7：确认无误后，删除废弃数据库
DROP DATABASE IF EXISTS myapp_prod_db_deprecated_20240115;
```

---

## 数据库管理

### 1. 查看数据库信息

```sql
-- 查看所有数据库
SHOW DATABASES;

-- 查看数据库创建语句
SHOW CREATE DATABASE myapp_prod_db;

-- 查看当前数据库
SELECT DATABASE();

-- 查看数据库大小
SELECT 
    table_schema AS database_name,
    ROUND(SUM(data_length) / 1024 / 1024 / 1024, 2) AS 'data_size_gb',
    ROUND(SUM(index_length) / 1024 / 1024 / 1024, 2) AS 'index_size_gb',
    ROUND(SUM(data_length + index_length) / 1024 / 1024 / 1024, 2) AS 'total_size_gb'
FROM information_schema.tables
WHERE table_schema = 'myapp_prod_db'
GROUP BY table_schema;
```

### 2. 切换数据库

```sql
-- 使用数据库
USE myapp_prod_db;

-- 验证当前数据库
SELECT DATABASE();
```

### 3. 数据库字符集管理

```sql
-- 查看数据库字符集
SHOW VARIABLES LIKE 'character_set_database';
SHOW VARIABLES LIKE 'collation_database';

-- 修改数据库字符集
ALTER DATABASE myapp_prod_db
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;
```

### 4. 数据库权限管理

```sql
-- 创建用户
CREATE USER 'app_readonly'@'%' IDENTIFIED BY 'password';
CREATE USER 'app_readwrite'@'%' IDENTIFIED BY 'password';
CREATE USER 'app_admin'@'localhost' IDENTIFIED BY 'password';

-- 授予只读权限
GRANT SELECT ON myapp_prod_db.* TO 'app_readonly'@'%';

-- 授予读写权限
GRANT SELECT, INSERT, UPDATE, DELETE ON myapp_prod_db.* TO 'app_readwrite'@'%';

-- 授予管理权限
GRANT ALL PRIVILEGES ON myapp_prod_db.* TO 'app_admin'@'localhost';

-- 刷新权限
FLUSH PRIVILEGES;

-- 查看用户权限
SHOW GRANTS FOR 'app_readwrite'@'%';

-- 撤销权限
REVOKE DELETE ON myapp_prod_db.* FROM 'app_readwrite'@'%';

-- 删除用户
DROP USER 'app_readonly'@'%';
```

---

## 阿里云 RDS 特殊建议

### 1. 数据库创建

- 在 RDS 控制台创建数据库
- 使用参数模板统一配置
- 配置自动备份策略

### 2. 数据库删除

- 删除前创建手动快照
- 确认快照可恢复
- 在维护窗口期执行

### 3. 监控告警

- 监控数据库大小
- 监控连接数
- 监控 QPS/TPS
- 配置告警阈值

---

## 字节跳动规范补充

### 1. 环境隔离

```sql
-- 严格区分环境
myapp_prod_db    -- 生产环境
myapp_staging_db -- 预发布环境
myapp_test_db    -- 测试环境
myapp_dev_db     -- 开发环境
```

### 2. 命名空间

```sql
-- 多应用命名空间
app1_prod_db
app2_prod_db
app3_prod_db
```

### 3. 数据库审批

- 生产环境创建数据库需要审批
- 删除数据库需要 CTO 授权
- 所有操作需要审计日志

---

## 总结

### 创建数据库 checklist

- [ ] 确认数据库命名符合规范
- [ ] 使用 utf8mb4 字符集
- [ ] 创建专用数据库用户
- [ ] 设置最小权限原则
- [ ] 记录数据库信息
- [ ] 配置备份策略
- [ ] 配置监控告警

### 删除数据库 checklist

- [ ] 完整备份整个数据库（多种方式）
- [ ] 统计数据库信息（表数量、数据大小）
- [ ] 确认无业务依赖（所有应用已停止连接）
- [ ] 确认无跨库依赖
- [ ] 验证备份可恢复
- [ ] 获得 CTO/技术VP 书面授权
- [ ] 通知所有相关团队成员
- [ ] 安排在维护窗口期执行
- [ ] 准备回滚方案
- [ ] 手动执行，绝对禁止自动执行

### 严禁事项

- [ ] 严禁自动执行 DROP DATABASE
- [ ] 严禁无备份删除数据库
- [ ] 严禁未确认依赖就删除数据库
- [ ] 严禁未获授权删除生产数据库
- [ ] 严禁使用非 utf8mb4 字符集
- [ ] 严禁使用 root 用户运行应用
- [ ] 严禁授予不必要的权限
