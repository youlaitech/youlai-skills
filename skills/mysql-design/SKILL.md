---
name: mysql-design
description: MySQL 数据库设计规范。当设计数据库表结构、字段类型、索引、命名规范时使用此 skill。
---

# MySQL 数据库设计规范

## 触发条件

- 设计数据库表结构
- 定义字段类型和长度
- 创建索引
- 命名表和字段

---

## Part 1: 命名规范

### 库名命名

| 规则 | 说明 |
|------|------|
| 小写字母 | `youlai_admin` |
| 下划线分隔 | 不用驼峰 |
| 业务前缀 | `youlai_` |

### 表名命名

| 类型 | 前缀 | 示例 |
|------|------|------|
| 系统表 | `sys_` | `sys_user`, `sys_role` |
| 业务表 | `biz_` | `biz_order`, `biz_product` |
| 关联表 | `rel_` 或主表前缀 | `sys_user_role`, `rel_user_dept` |
| 配置表 | `cfg_` | `cfg_dict`, `cfg_config` |
| 日志表 | `log_` | `log_login`, `log_operation` |

### 字段命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 主键 | `id` | `id` |
| 外键 | 表名 + `_id` | `user_id`, `dept_id` |
| 名称 | `xxx_name` | `user_name`, `dept_name` |
| 状态 | `status` | `status` |
| 类型 | `xxx_type` | `menu_type`, `log_type` |
| 时间 | `xxx_time` | `create_time`, `login_time` |
| 布尔 | `is_xxx` | `is_deleted`, `is_enabled` |
| 数量 | `xxx_count` | `login_count`, `view_count` |
| 排序 | `sort` 或 `order_num` | `sort` |

### 索引命名

| 类型 | 前缀 | 示例 |
|------|------|------|
| 主键 | `pk_` | `pk_user` |
| 唯一索引 | `uk_` | `uk_username` |
| 普通索引 | `idx_` | `idx_dept_id` |
| 组合索引 | `idx_` | `idx_dept_status` |
| 全文索引 | `ft_` | `ft_content` |

---

## Part 2: 字段类型规范

### 主键

```sql
`id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键'
```

### 字符串类型

| 场景 | 类型 | 长度 |
|------|------|------|
| 用户名 | VARCHAR | 50 |
| 昵称 | VARCHAR | 50 |
| 密码（加密后） | VARCHAR | 100 |
| 手机号 | VARCHAR | 20 |
| 邮箱 | VARCHAR | 100 |
| URL | VARCHAR | 500 |
| 标题 | VARCHAR | 200 |
| 简介 | VARCHAR | 500 |
| 详细描述 | TEXT | - |
| JSON 数据 | JSON | - |

### 数值类型

| 场景 | 类型 | 说明 |
|------|------|------|
| 主键/ID | BIGINT | 20 位 |
| 外键 | BIGINT | 关联主键类型 |
| 数量/计数 | INT | - |
| 金额（分） | BIGINT | 避免浮点精度问题 |
| 排序号 | INT | - |
| 状态 | TINYINT | 0/1/2 |
| 类型 | TINYINT | 枚举值 |
| 百分比 | DECIMAL(5,2) | 0.00-100.00 |

### 时间类型

| 场景 | 类型 | 说明 |
|------|------|------|
| 创建时间 | DATETIME | `create_time` |
| 更新时间 | DATETIME | `update_time` |
| 日期 | DATE | `birth_date` |
| 时间戳 | BIGINT | 毫秒级 |

### 布尔类型

```sql
`is_deleted` TINYINT(1) NOT NULL DEFAULT 0 COMMENT '是否删除(0否 1是)'
`is_enabled` TINYINT(1) NOT NULL DEFAULT 1 COMMENT '是否启用(0否 1是)'
```

---

## Part 3: 通用字段

### 每张表必备字段

```sql
`id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键',
`create_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
`update_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
`create_by` BIGINT UNSIGNED DEFAULT NULL COMMENT '创建人',
`update_by` BIGINT UNSIGNED DEFAULT NULL COMMENT '更新人',
`deleted` TINYINT(1) NOT NULL DEFAULT 0 COMMENT '删除标志(0未删除 1已删除)',
PRIMARY KEY (`id`)
```

### 可选字段

```sql
`remark` VARCHAR(500) DEFAULT NULL COMMENT '备注',
`version` INT NOT NULL DEFAULT 0 COMMENT '乐观锁版本号',
`tenant_id` BIGINT UNSIGNED DEFAULT NULL COMMENT '租户ID',
```

---

## Part 4: 索引规范

### 索引原则

| 原则 | 说明 |
|------|------|
| 最左前缀 | 组合索引按查询顺序建立 |
| 高选择性 | 选择区分度高的列 |
| 控制数量 | 单表索引不超过 5 个 |
| 组合优先 | 组合索引优于多个单列索引 |

### 必建索引

```sql
-- 主键索引
PRIMARY KEY (`id`)

-- 外键索引
INDEX `idx_dept_id` (`dept_id`)

-- 状态索引（常用于筛选）
INDEX `idx_status` (`status`)

-- 创建时间索引（常用于排序）
INDEX `idx_create_time` (`create_time`)

-- 唯一索引（业务唯一字段）
UNIQUE INDEX `uk_username` (`username`)
```

### 组合索引

```sql
-- 组合索引：部门 + 状态
INDEX `idx_dept_status` (`dept_id`, `status`)

-- 组合索引：用户 + 创建时间
INDEX `idx_user_time` (`user_id`, `create_time`)
```

### 索引避坑

| 避免 | 原因 |
|------|------|
| `LIKE '%xxx%'` | 前缀模糊不走索引 |
| `OR` 条件 | 可能不走索引，改用 UNION |
| 函数操作 | `WHERE YEAR(create_time) = 2024` 不走索引 |
| 类型转换 | 字符串与数字比较会隐式转换 |
| `NOT IN` | 改用 `NOT EXISTS` 或 `LEFT JOIN ... IS NULL` |

---

## Part 5: 表设计规范

### 用户表示例

```sql
CREATE TABLE `sys_user` (
  `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键',
  `username` VARCHAR(50) NOT NULL COMMENT '用户名',
  `password` VARCHAR(100) NOT NULL COMMENT '密码',
  `nickname` VARCHAR(50) NOT NULL COMMENT '昵称',
  `mobile` VARCHAR(20) DEFAULT NULL COMMENT '手机号',
  `email` VARCHAR(100) DEFAULT NULL COMMENT '邮箱',
  `gender` TINYINT NOT NULL DEFAULT 0 COMMENT '性别(0未知 1男 2女)',
  `avatar` VARCHAR(500) DEFAULT NULL COMMENT '头像URL',
  `status` TINYINT NOT NULL DEFAULT 1 COMMENT '状态(1正常 0禁用)',
  `dept_id` BIGINT UNSIGNED DEFAULT NULL COMMENT '部门ID',
  `create_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `create_by` BIGINT UNSIGNED DEFAULT NULL COMMENT '创建人',
  `update_by` BIGINT UNSIGNED DEFAULT NULL COMMENT '更新人',
  `deleted` TINYINT(1) NOT NULL DEFAULT 0 COMMENT '删除标志(0未删除 1已删除)',
  PRIMARY KEY (`id`),
  UNIQUE INDEX `uk_username` (`username`),
  INDEX `idx_dept_id` (`dept_id`),
  INDEX `idx_status` (`status`),
  INDEX `idx_create_time` (`create_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='用户表';
```

### 关联表示例

```sql
CREATE TABLE `sys_user_role` (
  `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键',
  `user_id` BIGINT UNSIGNED NOT NULL COMMENT '用户ID',
  `role_id` BIGINT UNSIGNED NOT NULL COMMENT '角色ID',
  `create_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`id`),
  UNIQUE INDEX `uk_user_role` (`user_id`, `role_id`),
  INDEX `idx_role_id` (`role_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='用户角色关联表';
```

---

## Part 6: 字段默认值

| 字段类型 | 默认值 | 说明 |
|---------|--------|------|
| 字符串 | `NULL` 或 `''` | 非必填用 NULL |
| 数值 | `0` | - |
| 布尔 | `0` 或 `1` | 根据业务 |
| 时间 | `CURRENT_TIMESTAMP` | - |
| 外键 | `NULL` | 非必填关联 |

### NULL vs NOT NULL

| 场景 | 选择 |
|------|------|
| 必填字段 | `NOT NULL` |
| 可选字段 | `NULL` |
| 外键（非必填） | `NULL` |
| 状态字段 | `NOT NULL DEFAULT 1` |

---

## Part 7: 字符集与引擎

### 字符集

```sql
DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
```

| 设置 | 说明 |
|------|------|
| `utf8mb4` | 支持 4 字节字符（emoji） |
| `utf8mb4_unicode_ci` | 不区分大小写排序 |

### 存储引擎

```sql
ENGINE=InnoDB
```

| 引擎 | 适用场景 |
|------|---------|
| InnoDB | 事务、外键、行锁 |
| MyISAM | 只读、全文索引（旧版） |
| Memory | 临时表、缓存 |

---

## Part 8: SQL 编写规范

### SELECT

```sql
-- ✅ 指定字段
SELECT id, username, nickname FROM sys_user WHERE status = 1;

-- ❌ 避免 SELECT *
SELECT * FROM sys_user;

-- ✅ 分页查询
SELECT id, username FROM sys_user WHERE status = 1 LIMIT 0, 20;

-- ✅ 批量插入
INSERT INTO sys_user (username, nickname) VALUES ('user1', '用户1'), ('user2', '用户2');
```

### UPDATE

```sql
-- ✅ 带 WHERE 条件
UPDATE sys_user SET nickname = '新昵称' WHERE id = 1;

-- ✅ 乐观锁
UPDATE sys_user SET nickname = '新昵称', version = version + 1 WHERE id = 1 AND version = 1;

-- ❌ 避免全表更新
UPDATE sys_user SET status = 0;
```

### DELETE

```sql
-- ✅ 逻辑删除
UPDATE sys_user SET deleted = 1 WHERE id = 1;

-- ✅ 物理删除（慎用）
DELETE FROM sys_user_role WHERE user_id = 1;
```

---

## Part 9: 性能优化

### 查询优化

| 优化点 | 说明 |
|--------|------|
| 避免 `SELECT *` | 只查需要的字段 |
| 避免 `LIKE '%xxx%'` | 用 `LIKE 'xxx%'` 或全文索引 |
| 避免大 OFFSET | `LIMIT 10000, 20` 改用游标分页 |
| 避免 `COUNT(*)` | 大表用缓存或估算 |
| 批量操作 | 批量插入/更新代替循环单条 |

### 表设计优化

| 优化点 | 说明 |
|--------|------|
| 垂直拆分 | 大字段拆到扩展表 |
| 水平拆分 | 大表按规则分表 |
| 适当冗余 | 减少关联查询 |
| 归档历史 | 冷数据迁移 |

---

## Part 10: 设计检查清单

- [ ] 表名使用小写 + 下划线
- [ ] 字段名使用小写 + 下划线
- [ ] 主键使用 BIGINT AUTO_INCREMENT
- [ ] 包含通用字段（create_time, update_time, deleted）
- [ ] 外键字段添加索引
- [ ] 唯一业务字段添加唯一索引
- [ ] 状态字段添加索引
- [ ] 使用 utf8mb4 字符集
- [ ] 使用 InnoDB 引擎
- [ ] 每个字段添加 COMMENT
- [ ] 表添加 COMMENT
- [ ] 索引数量不超过 5 个
