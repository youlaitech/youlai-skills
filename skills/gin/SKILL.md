---
name: gin
description: Gin backend development standards. Use this skill when developing Go/Gin projects, implementing REST APIs, using GORM, JWT authentication, or permission control.
---

# Gin 后端开发规范

> 本规范以 youlai-gin 当前代码为准（模块名 `youlai-gin`），并融合 Uber Go Style Guide、Go CodeReviewComments 社区惯例。

## 触发条件

- Develop Gin projects
- Implement REST APIs
- Use GORM for data access
- Implement JWT authentication
- Implement permission control

---

## Part 1: 技术栈

| 层 | 选型 | 说明 |
|----|------|------|
| 运行环境 | **Go 1.25+** | 标准库健壮、并发原生 |
| Web 框架 | **Gin** | 高性能 HTTP 路由器 |
| ORM | **GORM 2.0** | 链式查询 |
| 数据库 | **MySQL 8.x** | InnoDB 引擎 |
| 缓存 | **Redis 7.x** (go-redis) | 分布式缓存 / Token 存储 |
| 认证 | **golang-jwt v5** | JWT 签发/验签 |
| 日志 | **zap** + lumberjack 滚动 | 结构化日志 |
| API 文档 | **swaggo/swag** (`swag init -o api`) | OpenAPI 自动生成 |

---

## Part 2: 目录结构

> 基础目录参考 [Standard Go Project Layout](https://github.com/golang-standards/project-layout)；`internal/` 按业务模块划分，模块内含 handler/model/service/router。

```
├── main.go                         # 程序入口（配置/日志/数据库/存储初始化 + 优雅关闭）
├── api/                            # Swagger 产物（swag init -o api 生成：docs.go/swagger.json/swagger.yaml）
├── internal/                       # 私有代码（Go 编译器强制不可被外部导入）
│   ├── auth/                       # 认证模块（账密/短信/扫码/小程序登录）
│   │   ├── handler/ model/ service/ router.go
│   ├── system/                     # 系统管理（按实体分子模块）
│   │   ├── user/ role/ menu/ dept/ dict/ config/ notice/ log/
│   │   │   └── handler/ model/ repository/ service/
│   │   └── router.go               # 模块装配（构造注入或函数注册）
│   ├── codegen/                    # 代码生成器
│   ├── file/                       # 文件模块
│   ├── message/                    # 消息模块（SSE 推送）
│   ├── common/                     # 公共基础设施（禁止依赖业务模块）
│   │   ├── auth/                   # JWT / Redis TokenManager + 认证/权限中间件
│   │   ├── config/                 # 配置加载（viper）
│   │   ├── context/                # gin 上下文取用户信息、操作人注入
│   │   ├── database/               # GORM 初始化 + 分页
│   │   ├── excel/ json/ logger/    # 导出、序列化、结构化日志
│   │   ├── permission/             # 权限服务 + 数据权限（datascope）
│   │   ├── redis/                  # Redis 客户端 + key 规范
│   │   ├── storage/                # 文件存储（local/s3/aliyun 工厂）
│   │   ├── utils/                  # 文件/密码/树/验证码工具（历史包，新代码不再往里堆）
│   │   ├── validator/              # 参数绑定与校验（BindJSON/BindQuery）
│   │   └── response.go             # 统一响应（包名 common，导入时别名 response）
│   ├── middleware/                 # 全局中间件（error_handler/operation_log/rate_limiter/requestid）
│   └── router/register.go          # 路由注册入口
├── pkg/                            # 公共库（可被外部导入，禁止依赖 internal/）
│   ├── constant/                   # 业务常量 + 错误码（阿里规范 "00000"/"A0400"…）
│   ├── enums/                      # 操作类型 / 日志模块枚举
│   ├── errs/                       # AppError 统一错误（Code/Msg/HTTPStatus/Err/Stack）
│   ├── gormx/                      # GORM 扩展（审计钩子、PatchMap）
│   ├── model/                      # BaseEntity/BaseQuery/PagedData/Option/KeyValue
│   └── types/                      # BigInt/FlexInt/LocalTime 序列化类型
├── configs/                        # dev/prod/test.yaml
├── docs/                           # 项目文档与图片（非 Swagger 产物）
├── docker/docker-compose.yml
├── sql/mysql/                      # 建库脚本
└── go.mod / go.sum
```

**设计原则**：
- `internal/` 不可被外部导入（编译器强约束）；`pkg/` 可复用，**不得 import `internal/`**
- `internal/common/` 是基础设施，**不得 import 业务模块**（`internal/system` 等）
- 业务模块分层 Handler → Service → Repository，model 放 entity/form/query/vo
- **目标结构（新模块必须遵循）**：结构体 + 构造器注入，如 `dept`：

```go
deptHandler.NewHandler(deptService.NewService(deptRepo.NewRepository(db))).RegisterRoutes(r)
```

user/role/config/notice/log 等包级函数风格为历史遗留，新代码不再模仿。

| 层 | 目录 | 职责 | 禁区 |
|----|------|------|------|
| Handler | `handler/` | 参数绑定、调用 service、写响应 | 不写业务逻辑、不碰 GORM |
| Model | `model/` | entity/form/query/vo | 无逻辑 |
| Repository | `repository/` | GORM 查询 | 不写业务判断 |
| Service | `service/` | 业务逻辑 | **不收 `*gin.Context`**，第一参数是 `context.Context` |

---

## Part 3: 命名规范

### 3.1 文件命名

| 类型 | 规范 | 示例 |
|------|------|------|
| Handler | `{name}_handler.go` | `user_handler.go` |
| Service | `{name}_service.go` | `user_service.go` |
| Repository | `{name}_repo.go` | `user_repo.go` |
| Entity | `entity.go` 在 model/ 下 | `model/entity.go` |
| 表单/查询/视图 | `form.go` / `query.go` / `vo.go` | 同上 |

### 3.2 命名原则

- **简洁不冗余**：包名已提供上下文，类型名不重复包名
- 导出大写 `GetByID`，非导出小写 `parseToken`
- `context.Context` 始终作为函数**第一个参数**
- **禁止**把 `pkg/model` 别名为 `common`（误导且与 `internal/common` 混淆）
- 避免包名撞标准库（历史遗留 `internal/common/context`，导入时别名 `appContext`）

```go
// ❌ 冗余：调用时变成 user.UserService
package user
type UserService struct{}

// ✅ 包名已提供语义
package user
type Service struct{}
```

### 3.3 方法命名（新代码必须遵循）

| 动作 | 前缀 | 示例 |
|------|------|------|
| 查询单个 | Get | `Get(ctx, id)` |
| 查询列表 | List | `List(ctx, query)` |
| 分页查询 | Page | `Page(ctx, query)` |
| 新增 | Create | `Create(ctx, form)` |
| 更新 | Update | `Update(ctx, form)` |
| 删除 | Delete | `Delete(ctx, id)` |
| 下拉选项 | Options | `Options(ctx)` |
| 存在性校验 | Exists | `ExistsByCode(ctx, code)` |

- 方法名**不带实体名**：接收者已是 `dept.Repository`，写 `List` 而非 `GetDeptList`
- 历史遗留的 `Save*`（create-or-update 合一）、`GetXxxList` 不再模仿

### 3.4 变量命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 变量 | camelCase | `userList` |
| 常量 | PascalCase | `MaxSize` |
| 私有变量 | 小写开头 | `internalState` |
| 布尔值 | Is/Has/Can 前缀 | `isDeleted`, `hasPermission` |

### 3.5 import 排序（`goimports -local youlai-gin` 强制执行）

```go
import (
    // 1. 标准库
    "context"
    "time"

    // 2. 第三方库
    "github.com/gin-gonic/gin"
    "gorm.io/gorm"

    // 3. 本项目内部
    "youlai-gin/internal/common/response"
    "youlai-gin/pkg/errs"
)
```

---

## Part 4: RESTful API 规范

### 4.1 标准 CRUD 路径（以用户为例）

| 操作 | 方法 | 路径 |
|------|------|------|
| 分页列表 | `GET` | `/api/v1/users`（pageNum/pageSize/关键词走 query） |
| 表单回显 | `GET` | `/api/v1/users/:userId/form` |
| 新增 | `POST` | `/api/v1/users` |
| 更新 | `PUT` | `/api/v1/users/:userId` |
| 删除（可批量） | `DELETE` | `/api/v1/users/:ids`（逗号分隔） |
| 状态变更 | `PATCH` | `/api/v1/users/:userId/status` |
| 下拉选项 | `GET` | `/api/v1/users/options` |
| 导入/导出 | `POST/GET` | `/api/v1/users/import` / `/api/v1/users/export` |

### 4.2 Handler 模板

```go
// Page 用户分页列表
// @Summary 用户分页列表
// @Tags 02.用户接口
// @Param query query model.UserQuery false "查询参数"
// @Success 200 {object} common.Result
// @Router /api/v1/users [get]
func (h *Handler) Page(c *gin.Context) {
    var query model.UserQuery
    if err := validator.BindQuery(c, &query); err != nil {
        c.Error(err)
        return
    }
    result, err := h.svc.Page(c.Request.Context(), &query)
    if err != nil {
        c.Error(err)
        return
    }
    response.OkPaged(c, result)
}
```

要点：绑定用 `validator.BindJSON/BindQuery`（返回已包装的 `*errs.AppError`）；错误一律 `c.Error(err) + return`；响应一律 `response.Ok/OkPaged/OkMsg`。

---

## Part 5: 响应格式与异常处理

### 5.1 统一响应（`internal/common/response.go`，包名 `common`）

```go
type Result struct {
    Code string      `json:"code"` // 阿里错误码：00000 成功 / A0400 参数错 / B0001 系统错
    Msg  string      `json:"msg"`
    Data interface{} `json:"data"`
}
```

| 函数 | 用途 |
|------|------|
| `Ok(c, data)` | 成功 + 数据 |
| `OkPage(c, list, pageNum, pageSize, total)` / `OkPaged(c, *model.PagedData)` | 分页 |
| `OkMsg(c, msg)` | 成功 + 仅提示 |
| `FromAppError(c, ae)` | 按 `AppError` 输出（真实 HTTP 状态码），错误中间件专用 |

### 5.2 错误流：`c.Error` + 全局中间件

```go
// pkg/errs
type AppError struct {
    Code       string `json:"code"`
    Msg        string `json:"msg"`
    HTTPStatus int    `json:"-"`
    Err        error  `json:"-"`  // 仅日志，不返回客户端
    Stack      string `json:"-"`
}
```

- Handler 只写 `c.Error(errs.BadRequest("具体信息"))` / 透传 service 返回的 error
- `middleware.ErrorHandler()` 统一 `FromAppError` 输出；类型判断用 `errors.As`，不用裸断言
- **禁止** handler 直接调用 `response.Fail/BadRequest`（遗留函数，待删除）
- 错误只在顶层 log 一次，中间层 `fmt.Errorf("...: %w", err)` 包装

---

## Part 6: 实体规范

```go
// pkg/model.BaseEntity：公共审计字段 + 手动软删除
type BaseEntity struct {
    CreateBy   *types.BigInt   `gorm:"column:create_by" json:"createBy,omitempty"`
    CreateTime types.LocalTime `gorm:"column:create_time;autoCreateTime" json:"createTime,omitempty"`
    UpdateBy   *types.BigInt   `gorm:"column:update_by" json:"updateBy,omitempty"`
    UpdateTime types.LocalTime `gorm:"column:update_time;autoUpdateTime" json:"updateTime,omitempty"`
    IsDeleted  int             `gorm:"column:is_deleted;default:0" json:"-"`
}

// 业务实体：主键用 types.BigInt（解决 JS 大数精度），显式 column 标签
type User struct {
    ID       types.BigInt `gorm:"primaryKey;autoIncrement" json:"id"`
    Username string       `gorm:"column:username;not null" json:"username"`
    Password string       `gorm:"column:password;not null" json:"-"`
    DeptID   types.BigInt `gorm:"column:dept_id" json:"deptId"`
    common.BaseEntity
}

func (User) TableName() string { return "sys_user" }
```

- 软删除走 `is_deleted` 标志位（**不是** `gorm.DeletedAt`），查询需带条件
- `create_by/update_by` 由 `gormx` 审计钩子从 ctx 填充，**同一张表只允许一个实体定义**

---

## Part 7: Service 规范

```go
// Repository 接口定义在消费方（service 包），便于替换与测试
type Repository interface {
    Page(ctx context.Context, query *model.UserQuery) ([]model.User, int64, error)
    Create(ctx context.Context, user *model.User) error
}

type Service struct {
    repo Repository
}

func NewService(repo Repository) *Service { return &Service{repo: repo} }

func (s *Service) Create(ctx context.Context, form *model.UserForm) error {
    exists, err := s.repo.ExistsByUsername(ctx, form.Username)
    if err != nil {
        return err
    }
    if exists {
        return errs.BadRequest("用户名已存在")
    }
    user := &model.User{Username: form.Username, Password: utils.HashPassword(form.Password)}
    return s.repo.Create(ctx, user)
}
```

- **禁止** service/repo 方法接收 `*gin.Context`；需要操作人时由 handler 传入 `appContext.OperatorCtx(c)`（已注入 operator，供审计钩子使用）
- repo 方法统一 `ctx` 第一参数 + `r.db.WithContext(ctx)`
- 业务失败返回 `errs.*`，**不用 panic**

---

## Part 8: 认证规范

### 8.1 TokenManager 与中间件

```go
// internal/common/auth：TokenManager 接口（JWT/Redis 双实现），由 config.Security 决定
tokenManager, _ := auth.CreateTokenManager(&config.Cfg.Security)
authorized.Use(pkgAuth.Middleware(tokenManager))     // 验签 + 写入用户上下文
r.GET("...", auth.RequirePermission("sys:user:list"), handler) // 权限点
```

### 8.2 权限标识格式

```
模块:实体:操作
sys:user:create / sys:user:update / sys:user:delete / sys:role:list
```

### 8.3 取当前用户

- `appContext.GetCurrentUserID(c)` / `GetCurrentUser(c)`：返回 `(v, error)`，错误即 `errs.Unauthorized`，直接 `c.Error`
- 需要写操作人：`appContext.OperatorCtx(c)` 转 `context.Context` 传给 service

---

## Part 9: 注释规范

- **导出符号必须有 godoc 注释**（中文可，信号字保留英文）
- Swagger 注解：`@Summary` / `@Tags` / `@Param` / `@Success` / `@Router`；Tags 用 `01.认证接口` 这类带序号分组
- 注释写 **Why** 不写 What；不写"用于 XXX 流程"这类会腐烂的上下文
- 生成物：`swag init -o api`（输出到 `api/`，勿手改）

```go
// Page 分页查询用户列表。
//
// 根据关键词模糊搜索用户名/昵称/手机号，支持状态筛选。
//
// @Summary 用户分页列表
// @Tags 02.用户接口
// @Param query query model.UserQuery false "查询参数"
// @Success 200 {object} common.Result
// @Router /api/v1/users [get]
func (h *Handler) Page(c *gin.Context) { ... }
```

---

## Part 10: 代码质量检查清单

- [ ] 遵循 RESTful 路径规范（Part 4.1）
- [ ] 错误走 `c.Error` + ErrorHandler，禁止 handler 直接 `response.Fail`
- [ ] service/repo 不出现 `*gin.Context`，`ctx` 为第一参数
- [ ] `strconv.ParseInt/Atoi` 必须检查错误
- [ ] 文件上传校验大小和类型
- [ ] 实体内嵌 `pkg/model.BaseEntity`，软删用 `is_deleted`
- [ ] 新 API 有 Swaggo 注解，`swag init -o api` 重新生成
- [ ] `goimports -w -local youlai-gin` + `go vet` 通过，CI 加 `golangci-lint`
- [ ] import 三组有序（标准库 → 第三方 → 项目内）
- [ ] 导出符号有 godoc 注释
- [ ] 错误判断用 `errors.Is/As`，不裸类型断言

### 常见反模式

| 反模式 | 正确做法 |
|--------|----------|
| `fmt.Printf("debug: %v", data)` | `slog/zap` 结构化日志 |
| `panic(err)` 处理业务错误 | 返回 `errs.*`，handler `c.Error` |
| `strconv.ParseInt(s, 10, 64)` 忽略 err | 必须检查错误 |
| Context 塞进结构体字段 | Context 作为函数第一个参数 |
| service 收 `*gin.Context` | handler 传 `OperatorCtx(c)` / `c.Request.Context()` |
| 层层 `log.Error + return err` | 只在顶层 log 一次，中间 `%w` 包装 |
| HTTP 调用无超时 | `context.WithTimeout(ctx, 30*time.Second)` |
| 堆栈返回给调用方 | 错误信息脱敏，堆栈只写日志 |
| 新建 `utils`/`common` 杂物包 | 按域命名（如 `excel`、`storage`），工具函数放进使用它的包 |
| 把 `pkg/model` 别名为 `common` | 直接用包名 `model` |
| 同一张表多个实体定义 | 实体唯一定义，跨层复用 |
| 悬空的 DI 图 / 无人调用的导出函数 | 删掉或补全，死代码不过夜 |
