---
name: rust
description: Rust 后端开发规范. Use this skill when developing Rust web projects, implementing REST APIs with Axum + SeaORM, JWT/Redis token authentication, or the youlai-rust admin backend (PostgreSQL).
---

# Rust 后端开发规范（Axum + SeaORM）

> 本 Skill 配合设计文档 `docs/youlai-rust-2026.md` 使用。AI 助手依据这两份文档可从 0 到 1 创建与 youlai-boot-postgres 功能等价的 Rust 后端。

## 触发条件

- 开发 Rust Web 项目
- 使用 Axum 框架实现 REST API
- 使用 SeaORM 异步 ORM
- 使用 serde / validator 做数据验证
- 实现 JWT / Redis Token 认证
- 实现基于角色的权限控制（RBAC）
- 开发 youlai-rust 管理后台模块
- 依据 `docs/youlai-rust-2026.md` 创建 youlai-rust 项目

---

## Part 1: 技术栈

| 层 | 选型 | 说明 |
|---|------|------|
| 异步运行时 | **tokio** 1.x | Rust 异步事实标准 |
| Web 框架 | **axum** 0.8+ | Tokio 官方，Tower 中间件生态 |
| ORM | **sea-orm** 1.1+ | ActiveRecord + 查询构建器，对标 MyBatis-Plus |
| 迁移工具 | **sea-orm-migration** 1.1+ | SeaORM 内置 |
| 数据验证 | **validator** 0.18+ | `#[validate]` 派生，对标 Bean Validation |
| 数据库 | **PostgreSQL** 16+ | 复用 `youlai-boot-postgres/sql/youlai_admin.sql` |
| 数据库驱动 | **sqlx-postgres** 0.8+ | SeaORM 底层，异步原生 |
| 缓存 | **Redis** 7.x | `redis` crate 异步客户端 |
| 本地缓存 | **moka** 0.12+ | 高性能并发缓存 |
| 序列化 | **serde + serde_json** 1.x | Rust 序列化事实标准 |
| 配置管理 | **figment** 0.10+ | 多源合并（TOML + ENV） |
| 日志 | **tracing + tracing-subscriber** 0.1+ | 结构化日志、span 链路追踪 |
| API 文档 | **utoipa + utoipa-swagger-ui** 5.x / 8.x | 宏派生 OpenAPI |
| 认证 | **jsonwebtoken** 9.x + **bcrypt** 0.16+ | JWT 签发/验签、密码哈希 |
| 限流 | **tower-governor** 0.4+ | Tower 兼容 |
| Excel | **calamine**(读) + **rust_xlsxwriter**(写) | 纯 Rust |
| 对象存储 | **aws-sdk-s3** 1.x | MinIO/S3 兼容 |
| 邮件 | **lettre** 0.11+ | 异步 SMTP |
| 任务调度 | **tokio-cron-scheduler** 0.13+ | 进程内 cron |
| 包管理 | **cargo** | Rust 官方 |
| 代码质量 | **clippy + rustfmt** | 官方 linter + 格式化 |
| 测试 | **#[test] + reqwest + testcontainers** | testcontainers 起真实依赖 |

> **不含多租户**：youlai-rust 不实现多租户，数据库为 PostgreSQL。

---

## Part 2: 目录结构

```
youlai-rust/
├── src/
│   ├── main.rs                       # 程序入口：装配路由/中间件/状态/优雅关停
│   ├── lib.rs                        # 库入口（供集成测试）
│   ├── core/                         # 核心基础设施
│   │   ├── config.rs                 # figment 读取 config.toml + .env，强类型 Settings
│   │   ├── database.rs               # SeaORM DatabaseConnection + 连接池
│   │   ├── redis.rs                  # async Redis 连接池
│   │   └── state.rs                  # AppState（db / redis / token_manager）
│   ├── common/                       # 公共常量/枚举/工具
│   │   ├── constants.rs              # SecurityConstants / RedisConstants
│   │   ├── enums.rs                  # ResultCode 枚举
│   │   └── pagination.rs             # PageQuery / PageData
│   ├── entity/                       # SeaORM Entity（sea-orm-cli 生成）
│   │   ├── mod.rs
│   │   ├── sys_user.rs
│   │   ├── sys_role.rs
│   │   └── ...
│   ├── framework/                    # 框架层（不依赖 modules/）
│   │   ├── web/                      # 统一响应与异常处理
│   │   │   ├── response.rs           # Result<T> + PageResult<T> + PageData<T>
│   │   │   ├── error.rs              # AppError + AppResult<T>
│   │   │   └── exception.rs          # 全局异常处理
│   │   ├── security/                 # 安全模块
│   │   │   ├── token.rs              # TokenManager trait + JwtTokenManager + RedisTokenManager
│   │   │   ├── extractor.rs          # current_user 提取器
│   │   │   ├── layer.rs              # Token 认证 tower Layer
│   │   │   ├── permission.rs         # require_perm 权限 guard
│   │   │   └── model.rs             # SecurityUserDetails / AuthenticationToken
│   │   ├── captcha/                  # 图形验证码
│   │   │   └── service.rs            # captcha crate + Redis 存储
│   │   ├── cache/                    # 缓存
│   │   │   └── redis_cache.rs        # Redis 缓存工具
│   │   ├── sse/                      # SSE 实时推送
│   │   │   └── manager.rs            # SseManager + connect / broadcast
│   │   ├── middleware/               # 中间件
│   │   │   ├── cors.rs
│   │   │   ├── log.rs                # 操作日志中间件
│   │   │   └── rate_limit.rs         # 限流（tower-governor）
│   │   ├── integration/              # 第三方集成
│   │   │   ├── mail/                 # 邮件（lettre + tera）
│   │   │   ├── sms/                  # 短信
│   │   │   └── wxma/                 # 微信小程序
│   │   ├── apidoc.rs                 # utoipa OpenAPI + Swagger UI
│   │   └── job/                      # 定时任务
│   └── modules/                      # 业务模块
│       ├── auth/                     # 认证（router + service + dto）
│       ├── system/                   # 系统管理
│       │   ├── user/                 # 用户
│       │   ├── role/                 # 角色
│       │   ├── menu/                 # 菜单
│       │   ├── dept/                 # 部门
│       │   ├── dict/                 # 字典
│       │   ├── config/               # 配置
│       │   ├── notice/               # 通知
│       │   └── log/                  # 日志
│       ├── message/                  # 消息/SSE
│       ├── file/                     # 文件
│       └── codegen/                  # 代码生成
├── migration/                        # sea-orm-migration 迁移
├── templates/                        # tera 模板（邮件/代码生成）
├── tests/                            # 集成测试
├── config/                           # 配置文件
│   ├── config.toml
│   └── config-dev.toml
├── docker/                           # Docker 部署
├── Cargo.toml
├── .env.example
├── rust-toolchain.toml
└── README.md
```

**每个业务模块（如 user）内部结构**：

```
src/modules/system/user/
├── mod.rs        # 路由注册 routes() + handler 函数
├── dto.rs        # 请求/响应 DTO（UserCreate / UserUpdate / UserVO / UserQuery）
└── service.rs    # UserService 业务逻辑
```

**设计原则**：
- `entity/` 是纯 SeaORM Entity，由 `sea-orm-cli` 自动生成，不依赖任何业务层
- `framework/` 是框架层，不依赖 `modules/`
- `modules/` 是业务层，依赖 `framework/` 和 `entity/`
- `common/` 是常量/枚举/工具，被所有层共享

---

## Part 3: 命名规范

### 3.1 文件命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 模块目录 | 小写下划线（snake_case）名词 | `user/`, `dict/`, `dict_item/` |
| 模块入口 | `mod.rs` | `src/modules/system/user/mod.rs` |
| DTO 文件 | `dto.rs` | `src/modules/system/user/dto.rs` |
| Service 文件 | `service.rs` | `src/modules/auth/service.rs` |
| Entity 文件 | 表名 | `src/entity/sys_user.rs` |
| 配置文件 | `config.rs` | `src/core/config.rs` |
| 常量文件 | `constants.rs` | `src/common/constants.rs` |
| 枚举文件 | `enums.rs` | `src/common/enums.rs` |
| 测试文件 | `<模块>_api.rs` | `tests/auth_api.rs` |

### 3.2 类型命名

| 类型 | 规范 | 示例 |
|------|------|------|
| SeaORM Entity | 表名转 PascalCase | `SysUser`, `SysRole`, `SysMenu` |
| DTO 结构体 | 功能 + 后缀 | `UserCreate`, `UserUpdate`, `UserVO`, `UserQuery` |
| Service 结构体 | 模块名 + `Service` | `UserService`, `AuthService`, `RoleService` |
| 配置结构体 | `Settings` / `XxxConfig` | `Settings`, `SecurityConfig`, `DatabaseConfig` |
| 枚举 | 功能名（无 Enum 后缀） | `ResultCode`, `Status`(i16 别名) |
| 错误类型 | `AppError` | 统一使用 `AppError` |
| Trait | 能力名 | `TokenManager`, `TenantQuery`(无) |
| Handler 函数 | snake_case 动词 | `list_users`, `create_user`, `get_user_form` |

**DTO 后缀规范**：

| 后缀 | 用途 | 示例 |
|------|------|------|
| `Create` | 创建表单 | `UserCreate`, `RoleCreate` |
| `Update` | 更新表单 | `UserUpdate`, `MenuUpdate` |
| `VO` | 视图对象（返回） | `UserVO`, `RoleVO`, `DictVO` |
| `Query` | 分页查询参数 | `UserQuery`, `RoleQuery`, `LogQuery` |
| `Form` | 特定操作表单 | `UserPasswordForm`, `RoleMenuForm` |
| `Result` | 响应结果 | `LoginResult`, `ExcelResult` |

### 3.3 方法命名

| 操作 | 方法名 | 示例 |
|------|--------|------|
| 分页查询 | `get_page` | `pub async fn get_page(&self, query: UserQuery) -> AppResult<PageData<UserVO>>` |
| 单条查询 | `get_by_id` | `pub async fn get_by_id(&self, id: i64) -> AppResult<UserVO>` |
| 树形查询 | `get_tree` | `pub async fn get_tree(&self, keywords: &str) -> AppResult<Vec<MenuVO>>` |
| 下拉选项 | `get_options` | `pub async fn get_options(&self) -> AppResult<Vec<OptionVO>>` |
| 创建 | `create` | `pub async fn create(&self, form: UserCreate) -> AppResult<UserVO>` |
| 更新 | `update` | `pub async fn update(&self, form: UserUpdate) -> AppResult<UserVO>` |
| 删除（批量） | `delete` | `pub async fn delete(&self, ids: &str) -> AppResult<u64>` |
| 状态变更 | `update_status` | `pub async fn update_status(&self, id: i64, status: i16)` |
| 导入 | `import_<资源>` | `pub async fn import_users(&self, bytes: Vec<u8>) -> AppResult<ExcelResult>` |
| 导出 | `export_<资源>` | `pub async fn export_users(&self, query: UserQuery) -> AppResult<Vec<u8>>` |

### 3.4 变量命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 变量 | snake_case | `user_list`, `role_codes` |
| 常量 | UPPER_SNAKE_CASE | `MAX_PAGE_SIZE`, `ROOT_ROLE_CODE` |
| 私有方法/字段 | 无特殊前缀（Rust 用可见性控制） | `fn to_vo(...)` |
| 布尔变量 | `is_`/`has_` 前缀 | `is_root`, `is_deleted`, `has_perm` |
| 数据库字段 | snake_case | `dept_id`, `create_time` |
| JSON/API 字段 | camelCase（serde rename） | `deptId`, `createTime`, `pageSize` |

> **关键约定**：Entity 字段用 snake_case（对应 PostgreSQL 列名），DTO/API 字段用 camelCase（对应前端 JSON 键名），通过 `#[serde(rename_all = "camelCase")]` 或 `#[serde(rename = "camelCase")]` 转换。

---

## Part 4: 路由与 API 规范

### 4.1 路由前缀

```rust
// src/modules/system/user/mod.rs
use axum::Router;
pub fn routes() -> Router<AppState> {
    Router::new()
        .route("/api/v1/users", get(list_users).post(create_user))
        // ...
}
```

| 模块 | 前缀 |
|------|------|
| 认证 | `/api/v1/auth` |
| 用户 | `/api/v1/users` |
| 角色 | `/api/v1/roles` |
| 菜单 | `/api/v1/menus` |
| 部门 | `/api/v1/depts` |
| 字典 | `/api/v1/dicts`、`/api/v1/dict-items` |
| 配置 | `/api/v1/configs` |
| 通知 | `/api/v1/notices` |
| 日志 | `/api/v1/logs` |
| 文件 | `/api/v1/files` |
| SSE | `/api/v1/sse` |
| 代码生成 | `/api/v1/gen` |

### 4.2 标准 CRUD 路径

| 操作 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 分页列表 | `GET` | `/api/v1/users` | Query 参数：`pageNum`/`pageSize`/`keywords`/... |
| 表单数据 | `GET` | `/api/v1/users/:id/form` | 路径参数 `id: i64` |
| 新增 | `POST` | `/api/v1/users` | Body: `UserCreate` |
| 更新 | `PUT` | `/api/v1/users/:id` | Body: `UserUpdate` |
| 批量删除 | `DELETE` | `/api/v1/users/:ids` | `ids` 逗号分隔 |
| 修改状态 | `PATCH` | `/api/v1/users/:id/status` | Body: `UserStatusForm` |
| 重置密码 | `PUT` | `/api/v1/users/:id/password/reset` | |
| 导出 | `GET` | `/api/v1/users/export` | 返回 xlsx 字节流 |
| 导入 | `POST` | `/api/v1/users/import` | multipart xlsx |
| 导入模板 | `GET` | `/api/v1/users/template` | 下载模板 xlsx |
| 下拉选项 | `GET` | `/api/v1/roles/options` | 返回 `[{value, label}]` |

### 4.3 路由注册顺序（重要！）

**固定路径端点必须定义在路径参数端点之前**，否则 axum 路由会冲突报错或被错误匹配：

```rust
// ✅ 正确顺序（axum 需显式注册每条路由）
Router::new()
    .route("/api/v1/users", get(list_users))           // 固定
    .route("/api/v1/users/template", get(download_template))  // 固定 —— 在 /:id 前
    .route("/api/v1/users/export", get(export_users))  // 固定
    .route("/api/v1/users/import", post(import_users)) // 固定
    .route("/api/v1/users/options", get(list_user_options))   // 固定
    .route("/api/v1/users/me", get(get_current_user))  // 固定
    .route("/api/v1/users/:id/form", get(get_user_form))      // 路径参数
    .route("/api/v1/users/:id", put(update_user))      // 路径参数
    .route("/api/v1/users/:ids", delete(delete_users)) // 路径参数
```

### 4.4 Handler 函数模板

```rust
// src/modules/system/user/mod.rs — 完整模板
use axum::{
    extract::{Path, Query, State},
    routing::{get, post, put, delete, patch},
    Json, Router,
};
use validator::Validate;
use crate::core::state::AppState;
use crate::framework::security::{extractor::current_user, permission::require_perm};
use crate::framework::web::response::{Result, PageResult};
use crate::modules::system::user::dto::{UserCreate, UserUpdate, UserQuery};
use crate::modules::system::user::service::UserService;

pub fn routes() -> Router<AppState> {
    Router::new()
        .route("/api/v1/users", get(list_users).post(create_user))
        .route("/api/v1/users/:id/form", get(get_user_form))
        .route("/api/v1/users/:id", put(update_user))
        .route("/api/v1/users/:ids", delete(delete_users))
}

/// 分页查询用户列表
#[utoipa::path(
    get, path = "/api/v1/users",
    tag = "用户管理",
    params(
        ("pageNum" = i64, Query, description = "当前页码"),
        ("pageSize" = i64, Query, description = "每页条数"),
        ("keywords" = Option<String>, Query, description = "搜索关键词"),
    ),
    responses((status = 200, body = PageResult<UserVO>))
)]
pub async fn list_users(
    State(state): State<AppState>,
    Query(query): Query<UserQuery>,
) -> AppResult<Json<PageResult<UserVO>>> {
    let page = UserService::new(&state.db).get_page(query).await?;
    Ok(Json(PageResult::success(page.list, page.total)))
}

/// 创建用户
#[utoipa::path(
    post, path = "/api/v1/users",
    tag = "用户管理",
    request_body = UserCreate,
    responses((status = 200, body = Result<UserVO>))
)]
pub async fn create_user(
    State(state): State<AppState>,
    Json(form): Json<UserCreate>,
) -> AppResult<Json<Result<UserVO>>> {
    form.validate()?;  // validator 校验
    let vo = UserService::new(&state.db).create(form).await?;
    Ok(Json(Result::success(vo)))
}
```

---

## Part 5: HTTP 状态码与业务错误码

### 5.1 设计理念

对齐 youlai-boot-postgres：**HTTP 状态码恒为 200**，业务状态码通过 body 的 `code` 字段传递。前端 axios 拦截器统一从 `response.data.code` 读取业务码。

### 5.2 业务错误码（ResultCode）

完整枚举见 `docs/youlai-rust-2026.md` 第 3.4 节。核心码段：

| 错误码 | 含义 | 说明 |
|--------|------|------|
| `00000` | 成功 | 正常返回 |
| `A0210` | 用户名或密码错误 | 登录失败 |
| `A0230` | 访问令牌无效或过期 | 触发前端 token 刷新 |
| `A0231` | 刷新令牌无效或过期 | 跳转登录页 |
| `A0301` | 访问未授权 | 权限不足 |
| `A0400` | 参数校验失败 | validator 校验不通过 |
| `B0001` | 系统执行出错 | 全局兜底 |

首位编码规则：
- `0` 成功（`00000`）
- `A` 用户端错误（认证/权限/参数）
- `B` 系统端错误
- `C` 第三方服务错误

### 5.3 AppError 与全局异常处理

```rust
// src/framework/web/error.rs
use axum::{http::StatusCode, response::{IntoResponse, Response}, Json};
use crate::common::enums::ResultCode;
use crate::framework::web::response::Result;

#[derive(Debug, thiserror::Error)]
pub enum AppError {
    #[error("{1:?}")]
    Business(ResultCode, Option<String>),
    #[error("参数校验失败: {0}")]
    Validation(String),
    #[error("数据库错误: {0}")]
    Database(String),
    #[error(transparent)]
    Other(#[from] anyhow::Error),
}

// validator::ValidationErrors → AppError
impl From<validator::ValidationErrors> for AppError {
    fn from(e: validator::ValidationErrors) -> Self {
        AppError::Validation(e.to_string())
    }
}

impl IntoResponse for AppError {
    fn into_response(self) -> Response {
        // 关键：业务错误统一返回 HTTP 200，code 通过 body 传递
        let result: Result<()> = match self {
            AppError::Business(code, msg) => Result::failed(code, msg),
            AppError::Validation(msg) => Result::failed(ResultCode::UserRequestParameterError, Some(msg)),
            AppError::Database(msg) => {
                tracing::error!("数据库错误: {msg}");
                Result::failed(ResultCode::DatabaseExecutionError, None)
            }
            AppError::Other(e) => {
                tracing::error!("系统异常: {e:#}");
                Result::failed(ResultCode::SystemError, None)
            }
        };
        (StatusCode::OK, Json(result)).into_response()
    }
}

pub type AppResult<T> = std::result::Result<T, AppError>;
```

### 5.4 业务异常抛出示例

```rust
// ✅ 认证失败
return Err(AppError::Business(ResultCode::UserPasswordError, None));
return Err(AppError::Business(ResultCode::AccessTokenInvalid, None));

// ✅ 权限不足
return Err(AppError::Business(ResultCode::AccessUnauthorized, Some("权限不足".into())));

// ✅ 数据不存在
return Err(AppError::Business(ResultCode::AccountNotFound, Some("用户不存在".into())));

// ✅ 参数校验（handler 中调用 form.validate()?）
form.validate()?;  // 自动转 AppError::Validation
```

---

## Part 6: 统一响应格式

### 6.1 Result 结构

```json
{
    "code": "00000",
    "msg": "成功",
    "data": { … }
}
```

```rust
// src/framework/web/response.rs
use serde::{Serialize, Deserialize};
use crate::common::enums::ResultCode;

#[derive(Debug, Serialize, Deserialize)]
#[serde(rename_all = "camelCase")]
pub struct Result<T> {
    pub code: String,
    pub msg: String,
    pub data: Option<T>,
}

impl<T> Result<T> {
    pub fn success(data: T) -> Self { /* code=00000 */ }
    pub fn success_empty() -> Self { /* data=None */ }
    pub fn failed(code: ResultCode, msg: Option<String>) -> Self { /* ... */ }
}
```

### 6.2 PageResult 结构

```json
{
    "code": "00000",
    "msg": "成功",
    "data": {
        "list": [ … ],
        "total": 100
    }
}
```

```rust
#[derive(Debug, Serialize, Deserialize)]
#[serde(rename_all = "camelCase")]
pub struct PageResult<T> {
    pub code: String,
    pub msg: String,
    pub data: PageData<T>,
}

#[derive(Debug, Serialize, Deserialize)]
#[serde(rename_all = "camelCase")]
pub struct PageData<T> {
    pub list: Vec<T>,
    pub total: u64,
}
```

### 6.3 分页查询参数

```rust
// src/common/pagination.rs
use serde::Deserialize;

#[derive(Debug, Deserialize)]
#[serde(rename_all = "camelCase")]
pub struct PageQuery {
    #[serde(default = "default_page_num")]
    pub page_num: i64,
    #[serde(default = "default_page_size")]
    pub page_size: i64,
}

fn default_page_num() -> i64 { 1 }
fn default_page_size() -> i64 { 10 }
```

> **前端约定**：分页参数命名为 `pageNum` / `pageSize`（camelCase），兼容 vue3-element-admin 的 TablePage 组件。

---

## Part 7: SeaORM Entity 规范

### 7.1 Entity 由 sea-orm-cli 自动生成

```bash
sea-orm-cli generate entity \
    --connection-string "postgres://postgres:123456@localhost:5432/youlai_admin" \
    --output-dir src/entity \
    --with-serde both
```

### 7.2 Entity 示例

```rust
// src/entity/sys_user.rs
use sea_orm::entity::prelude::*;
use chrono::NaiveDateTime;

#[derive(Clone, Debug, PartialEq, DeriveEntityModel, serde::Serialize, serde::Deserialize)]
#[sea_orm(table_name = "sys_user")]
pub struct Model {
    #[sea_orm(primary_key)]
    pub id: i64,                          // bigint
    pub username: String,                 // varchar
    pub nickname: String,
    pub password: String,
    #[sea_orm(nullable)]
    pub dept_id: Option<i64>,
    pub mobile: String,
    pub email: String,
    pub status: i16,                      // smallint → i16
    #[sea_orm(column_name = "create_time")]
    pub create_time: Option<NaiveDateTime>,  // timestamp
    #[sea_orm(column_name = "update_time")]
    pub update_time: Option<NaiveDateTime>,
    #[sea_orm(column_name = "is_deleted")]
    pub is_deleted: i16,
}

#[derive(Copy, Clone, Debug, EnumIter, DeriveRelation)]
pub enum Relation {
    #[sea_orm(has_many = "super::sys_user_role::Entity")]
    UserRole,
}

impl Related<super::sys_user_role::Entity> for Entity {
    fn to() -> RelationDef { Relation::UserRole.def() }
}

impl ActiveModelBehavior for ActiveModel {}
```

### 7.3 PostgreSQL 类型映射

| PostgreSQL 类型 | Rust 类型 | 说明 |
|----------------|-----------|------|
| `bigint` | `i64` | 主键、外键 |
| `smallint` | `i16` | 状态、标识位 |
| `integer` | `i32` | 普通整数 |
| `varchar(n)` | `String` | 变长字符串 |
| `text` | `String` | 长文本 |
| `boolean` | `bool` | 布尔 |
| `timestamp` | `chrono::NaiveDateTime` | 时间 |
| `jsonb` | `serde_json::Value` | JSON |
| `uuid` | `uuid::Uuid` | UUID |

### 7.4 字段命名对照

| SQL 列名 | Entity 字段 | DTO 字段 | 说明 |
|----------|-------------|----------|------|
| `dept_id` | `dept_id` | `deptId` | 部门ID |
| `create_time` | `create_time` | `createTime` | 创建时间 |
| `is_deleted` | `is_deleted` | — | 逻辑删除（内部使用） |
| `parent_id` | `parent_id` | `parentId` | 父节点ID |

> **转换规则**：Entity → VO 时，`to_vo` 方法负责 snake_case → camelCase 转换。

### 7.5 VO 转换模板

```rust
// src/modules/system/user/dto.rs
impl From<entity::sys_user::Model> for UserVO {
    fn from(m: entity::sys_user::Model) -> Self {
        UserVO {
            id: m.id,
            username: m.username,
            dept_id: m.dept_id,       // ← 手动映射（VO 用 #[serde(rename)]）
            create_time: m.create_time.map(|t| t.to_string()),
            // ...
        }
    }
}
```

---

## Part 8: 认证与鉴权

### 8.1 Token 模式

通过 `config.toml` 的 `security.session.type` 切换：

| 值 | 模式 | 实现 |
|----|------|------|
| `jwt`（默认） | JWT 自包含 Token | `JwtTokenManager` |
| `redis-token` | Redis 存储 Token | `RedisTokenManager` |

### 8.2 获取当前用户

```rust
use crate::framework::security::extractor::current_user;

pub async fn get_current_user_info(req: Request) -> AppResult<Json<Result<UserVO>>> {
    let user = current_user(&req).ok_or_else(|| 
        AppError::Business(ResultCode::AccessTokenInvalid, None))?;
    Ok(Json(Result::success(user.into())))
}
```

### 8.3 权限校验

```rust
use crate::framework::security::permission::require_perm;

// 精确权限（通过 route_layer 挂载中间件）
pub fn routes() -> Router<AppState> {
    Router::new()
        .route("/api/v1/users", get(list_users)
            .route_layer(middleware::from_fn(require_perm("sys:user:query"))))
        .route("/api/v1/users", post(create_user)
            .route_layer(middleware::from_fn(require_perm("sys:user:create"))))
}
```

### 8.4 权限标识命名规范

```
模块:实体:操作
sys:user:create
sys:user:edit
sys:user:delete
sys:role:query
sys:menu:query
sys:dept:query
```

| 操作 | 说明 |
|------|------|
| `query` | 列表/查询 |
| `create` | 新增 |
| `edit` | 编辑 |
| `delete` | 删除 |
| `import` | 导入 |
| `export` | 导出 |
| `reset_pwd` | 重置密码 |

---

## Part 9: 数据库操作规范

### 9.1 SeaORM 查询

```rust
// ✅ 单条查询
let user = entity::sys_user::Entity::find_by_id(user_id)
    .filter(entity::sys_user::Column::IsDeleted.eq(0))
    .one(&self.db)
    .await?
    .ok_or_else(|| AppError::Business(ResultCode::AccountNotFound, Some("用户不存在".into())))?;

// ✅ 分页查询
let paginator = entity::sys_user::Entity::find()
    .filter(entity::sys_user::Column::IsDeleted.eq(0))
    .filter(entity::sys_user::Column::Status.eq(1))
    .filter(entity::sys_user::Column::Username.like(format!("%{keywords}%")))
    .order_by_desc(entity::sys_user::Column::CreateTime)
    .paginate(&self.db, page_size as usize);

let total = paginator.num_items().await?;
let users = paginator.fetch_page((page_num - 1) as usize).await?;
```

### 9.2 事务管理

```rust
// ✅ 显式事务（多步写操作）
use sea_orm::TransactionTrait;

self.db.transaction::<_, _, AppError>(|txn| {
    Box::pin(async move {
        entity::sys_user::ActiveModel { ... }.insert(txn).await?;
        entity::sys_user_role::ActiveModel { ... }.insert(txn).await?;
        Ok(())
    })
}).await?;
```

### 9.3 批量操作

```rust
// ✅ 批量删除（逻辑删除）
let id_list: Vec<i64> = ids.split(',').filter_map(|s| s.parse().ok()).collect();
entity::sys_user::Entity::update_many()
    .col_expr(entity::sys_user::Column::IsDeleted, Expr::value(1))
    .filter(entity::sys_user::Column::Id.is_in(id_list))
    .exec(&self.db)
    .await?;

// ✅ 批量插入
let models: Vec<entity::sys_config::ActiveModel> = batch.into_iter().map(Into::into).collect();
entity::sys_config::Entity::insert_many(models).exec(&self.db).await?;
```

---

## Part 10: 添加新模块 — 完整教程

以添加 **"字典项"** 模块为例，展示从零到完整的步骤。

### Step 1: 生成 Entity

确认 `src/entity/sys_dict_item.rs` 已由 `sea-orm-cli generate entity` 生成。若无，重新执行生成命令。

### Step 2: 创建模块目录

```bash
mkdir src/modules/system/dict_item
```

### Step 3: 创建 `dto.rs`

```rust
// src/modules/system/dict_item/dto.rs
use serde::{Deserialize, Serialize};
use validator::Validate;

#[derive(Debug, Deserialize, Validate)]
#[serde(rename_all = "camelCase")]
pub struct DictItemCreate {
    #[validate(length(min = 1, max = 50))]
    pub dict_code: String,
    #[validate(length(min = 1, max = 50))]
    pub value: String,
    #[validate(length(min = 1, max = 100))]
    pub label: String,
    pub tag_type: Option<String>,
    pub status: Option<i16>,
    pub sort: Option<i16>,
}

#[derive(Debug, Deserialize, Validate)]
#[serde(rename_all = "camelCase")]
pub struct DictItemUpdate {
    pub id: i64,
    #[validate(length(min = 1, max = 50))]
    pub dict_code: String,
    pub value: String,
    pub label: String,
    pub tag_type: Option<String>,
    pub status: Option<i16>,
    pub sort: Option<i16>,
}

#[derive(Debug, Serialize)]
#[serde(rename_all = "camelCase")]
pub struct DictItemVO {
    pub id: i64,
    pub dict_code: String,
    pub value: String,
    pub label: String,
    pub tag_type: Option<String>,
    pub status: i16,
    pub sort: i16,
}

impl From<entity::sys_dict_item::Model> for DictItemVO {
    fn from(m: entity::sys_dict_item::Model) -> Self {
        Self {
            id: m.id,
            dict_code: m.dict_code,
            value: m.value,
            label: m.label,
            tag_type: m.tag_type,
            status: m.status.unwrap_or(1),
            sort: m.sort.unwrap_or(0),
        }
    }
}
```

### Step 4: 创建 `service.rs`

```rust
// src/modules/system/dict_item/service.rs
use sea_orm::*;
use crate::core::state::AppState;
use crate::framework::web::error::{AppError, AppResult};
use crate::common::enums::ResultCode;
use crate::modules::system::dict_item::dto::*;

pub struct UserService<'a> {
    db: &'a DatabaseConnection,
}

impl<'a> UserService<'a> {
    pub fn new(db: &'a DatabaseConnection) -> Self { Self { db } }

    pub async fn list_by_dict_code(&self, dict_code: &str) -> AppResult<Vec<DictItemVO>> {
        let items = entity::sys_dict_item::Entity::find()
            .filter(entity::sys_dict_item::Column::DictCode.eq(dict_code))
            .filter(entity::sys_dict_item::Column::IsDeleted.eq(0))
            .order_by_asc(entity::sys_dict_item::Column::Sort)
            .all(self.db).await?;
        Ok(items.into_iter().map(Into::into).collect())
    }

    pub async fn create(&self, form: DictItemCreate) -> AppResult<DictItemVO> {
        let active = entity::sys_dict_item::ActiveModel {
            dict_code: sea_orm::Set(form.dict_code),
            value: sea_orm::Set(form.value),
            label: sea_orm::Set(form.label),
            tag_type: sea_orm::Set(form.tag_type),
            status: sea_orm::Set(form.status.unwrap_or(1)),
            sort: sea_orm::Set(form.sort.unwrap_or(0)),
            ..Default::default()
        };
        let model = active.insert(self.db).await?;
        Ok(model.into())
    }

    pub async fn delete(&self, ids: &str) -> AppResult<u64> {
        let id_list: Vec<i64> = ids.split(',').filter_map(|s| s.parse().ok()).collect();
        let res = entity::sys_dict_item::Entity::update_many()
            .col_expr(entity::sys_dict_item::Column::IsDeleted, Expr::value(1))
            .filter(entity::sys_dict_item::Column::Id.is_in(id_list))
            .exec(self.db).await?;
        Ok(res.rows_affected)
    }
}
```

### Step 5: 创建 `mod.rs`（路由）

```rust
// src/modules/system/dict_item/mod.rs
pub mod dto;
pub mod service;

use axum::{extract::{Path, State}, routing::get, Json, Router};
use crate::core::state::AppState;
use crate::framework::web::response::Result;
use crate::modules::system::dict_item::dto::*;
use crate::modules::system::dict_item::service::DictItemService;

pub fn routes() -> Router<AppState> {
    Router::new()
        .route("/api/v1/dict-items", get(list_dict_items).post(create_dict_item))
        .route("/api/v1/dict-items/:id", axum::routing::put(update_dict_item))
        .route("/api/v1/dict-items/:ids", axum::routing::delete(delete_dict_items))
}

pub async fn list_dict_items(
    State(state): State<AppState>,
) -> AppResult<Json<Result<Vec<DictItemVO>>>> { /* ... */ }

pub async fn create_dict_item(
    State(state): State<AppState>,
    Json(form): Json<DictItemCreate>,
) -> AppResult<Json<Result<DictItemVO>>> { /* ... */ }
```

### Step 6: 在 `main.rs` 注册路由

```rust
// src/main.rs
let app = Router::new()
    .merge(modules::auth::routes())
    .merge(modules::system::dict_item::routes())  // ← 新增
    // ...
    .with_state(state);
```

### Step 7: 验证

```bash
cargo watch -x 'run'
# 访问 http://localhost:8000/swagger-ui 确认路由已注册
```

---

## Part 11: 注释规范

### 11.1 总则

- **doc comment 优先**：所有公共项用 `///` 文档注释
- **自解释代码**：好的命名和结构是自解释的，注释描述"为什么"
- **中文优先**：用中文把问题说清楚；专有名词保留英文（如 `HTTP`、`async`）
- **同步更新**：代码修改时 doc comment 必须同步修改

### 11.2 doc comment 格式

```rust
/// 分页查询用户列表。
///
/// 根据关键词模糊搜索用户名/昵称/手机号，支持状态筛选。
///
/// # 参数
/// - `query`: 分页查询参数（pageNum / pageSize / keywords / status）
///
/// # 返回
/// 分页结果，包含 list 和 total。
///
/// # 错误
/// - `AppError::Business(UserRequestParameterError)`: pageSize 超过 100
pub async fn get_page(&self, query: UserQuery) -> AppResult<PageData<UserVO>> {
    ...
}
```

### 11.3 各层注释要点

| 层 | 必须说明 |
|----|---------|
| 模块（`mod.rs` 顶部） | 文件职责 |
| Service struct | 职责、依赖 |
| 公共方法 | 功能、参数、返回值、错误 |
| Handler | 配合 `#[utoipa::path]` 注解 API 文档 |

### 11.4 TODO / FIXME 标记规范

| 标记 | 用途 | 格式 |
|------|------|------|
| `TODO` | 待实现 | `// TODO(标记人, 日期, 说明)` |
| `FIXME` | 已知 bug | `// FIXME(标记人, 日期, 说明)` |

---

## Part 12: 代码质量检查清单

### 12.1 通用检查

- [ ] 所有公共项有 `///` doc comment
- [ ] 用 `serde` 的 `#[derive(Serialize, Deserialize)]` 而非手动实现
- [ ] DTO 用 `#[serde(rename_all = "camelCase")]` 保证 JSON 驼峰
- [ ] import 按标准库 → 第三方 → 内部顺序排列（用 `cargo fmt` 自动处理）
- [ ] 无 `println!()` 调试代码，统一用 `tracing::info!`/`debug!`
- [ ] 日志不输出密码、密钥等敏感信息
- [ ] 配置通过 `figment` + `config.toml` + `.env` 管理，不硬编码
- [ ] 所有 `async fn` 内部使用 `await`
- [ ] `cargo clippy -- -D warnings` 无警告

### 12.2 数据库相关

- [ ] Entity 字段名使用 `snake_case`，与 PostgreSQL 列名一致
- [ ] 所有查询加上 `is_deleted == 0` 过滤
- [ ] 使用 SeaORM 的 `find()` / `filter()` 而非裸 SQL（复杂联表除外）
- [ ] 多步写操作使用 `transaction()`
- [ ] `smallint` 映射 `i16`，`bigint` 映射 `i64`

### 12.3 API 相关

- [ ] 路由 prefix 统一 `/api/v1/模块名`
- [ ] 固定路径端点定义在路径参数端点之前
- [ ] 分页参数命名为 `pageNum` / `pageSize`
- [ ] JSON 字段使用 camelCase（`deptId`、`createTime`）
- [ ] 统一返回 `Result<T>` / `PageResult<T>` 包装
- [ ] HTTP 状态码恒为 200，业务码通过 body 的 `code` 传递
- [ ] 错误返回使用 `ResultCode` 枚举，不硬编码字符串
- [ ] 敏感接口添加 `require_perm("xxx")` 权限中间件
- [ ] 每个 handler 添加 `#[utoipa::path]` 注解

### 12.4 Service 层

- [ ] Service 构造函数接受 `&DatabaseConnection`
- [ ] 数据不存在时返回 `AppError::Business(ResultCode::AccountNotFound/...)`
- [ ] 批量操作前检查参数非空
- [ ] `to_vo` / `from` 等转换方法用 `impl From<Model> for XxxVO`

---

## Part 13: 常见反模式

| 反模式 | 问题 | 正确做法 |
|--------|------|----------|
| `println!("debug")` | 无日志级别、无时间戳 | `tracing::info!()` / `tracing::debug!()` |
| 硬编码配置 | 不便运维 | 用 `figment` + `config.toml` + `.env` |
| `unwrap()` 在生产代码 | panic 风险 | 用 `?` + `AppError` 处理 |
| SQL 查询缺 `is_deleted = 0` | 查到已删除数据 | 所有查询加逻辑删除条件 |
| 固定路径在 `/:id` 之后 | axum 路由冲突 | 固定路径注册在前 |
| `code="A0230"` 硬编码字符串 | 易出错 | 用 `ResultCode::AccessTokenInvalid.code()` |
| 未加 `#[serde(rename)]` | JSON 字段非驼峰 | DTO 用 `#[serde(rename_all = "camelCase")]` |
| 所有错误 `unwrap_or_default()` | 隐藏错误 | 用 `?` 传播，由全局异常处理 |
| `sea-orm` features 用 `sqlx-mysql` | 连不上 PostgreSQL | 用 `sqlx-postgres` |
| Entity 手写 | 易与表结构脱节 | 用 `sea-orm-cli generate entity` 自动生成 |

---

## Part 14: Docker 部署

```yaml
# docker/docker-compose.yml
services:
  api:
    build: ..
    ports: ["8000:8000"]
    depends_on: [postgres, redis, minio]
    environment:
      YOULAI_DATABASE__URL: postgres://postgres:123456@postgres:5432/youlai_admin
      YOULAI_REDIS__URL: redis://redis:6379/0
      YOULAI_MINIO__ENDPOINT: http://minio:9000

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: youlai_admin
      POSTGRES_PASSWORD: "123456"
    ports: ["5432:5432"]
    volumes:
      - pg_data:/var/lib/postgresql/data
      - ../youlai-boot-postgres/sql/youlai_admin.sql:/docker-entrypoint-initdb.d/init.sql

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]

  minio:
    image: minio/minio
    ports: ["9000:9000", "9001:9001"]
    command: server /data --console-address ":9001"
    volumes: ["minio_data:/data"]

volumes:
  pg_data:
  minio_data:
```

---

## Part 15: 从 0 到 1 创建项目指引（AI 专用）

> 当 AI 助手被要求"依据文档创建 youlai-rust 项目"时，按以下步骤执行。

### 必读文档

1. **设计文档**：`docs/youlai-rust-2026.md`（完整方案、API 端点表、代码骨架、配置）
2. **本 Skill**：`youlai-skills/skills/rust/SKILL.md`（命名规范、开发约定）

### 创建步骤

1. **初始化项目**：`cargo new youlai-rust`，写入 `Cargo.toml`（见设计文档 3.1 节）
2. **按目录结构创建骨架**（见 Part 2）
3. **准备数据库**：导入 `youlai-boot-postgres/sql/youlai_admin.sql` 到 PostgreSQL
4. **生成 Entity**：执行 `sea-orm-cli generate entity`（见设计文档 9.2 节）
5. **写入配置**：`config/config.toml`（见设计文档第六章）
6. **按 P0-P8 阶段实现**（见设计文档 9.4 / 第十一章）：
   - P0：core（config/database/redis/state）+ framework/web（response/error）
   - P1：framework/security（token/extractor/layer/permission）+ framework/captcha
   - P2：modules/auth + framework/sse
   - P3：modules/system（user/role/menu/dept）
   - P4：modules/system（dict/config/notice/log）+ Excel + SSE 字典同步
   - P5：个人中心 + 短信登录
   - P6：modules/file + modules/codegen
   - P7：Docker + 邮件 + 定时任务
   - P8：微信小程序 + 异步任务
7. **每个模块遵循 Part 10 的"添加新模块"流程**
8. **验证**：`cargo clippy -- -D warnings` + `cargo test` + Swagger UI 手动测试

### 关键约束

- 数据库为 **PostgreSQL**（非 MySQL），SeaORM features 用 `sqlx-postgres`
- **不含多租户**，不要创建 tenant 相关的模块/中间件/表
- API 路径、响应格式、错误码 **100% 对齐 youlai-boot-postgres**（见设计文档第四章）
- HTTP 状态码恒为 200，业务码通过 `code` 字段传递
- JSON 键名驼峰（`#[serde(rename_all = "camelCase")]`）
- 默认账号：`admin` / `123456`
