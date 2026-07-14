---
name: gin
description: Gin backend development standards. Use this skill when developing Go/Gin projects, implementing REST APIs, using GORM, JWT authentication, or permission control.
---

# Gin 后端开发规范

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
| 运行环境 | **Go 1.22+** | 标准库健壮、并发原生 |
| Web 框架 | **Gin** | 高性能 HTTP 路由器 |
| ORM | **GORM 2.0** | 链式查询、自动迁移 |
| 数据库 | **MySQL 8.x** | InnoDB 引擎 |
| 缓存 | **Redis 7.x** (go-redis) | 分布式缓存 |
| 认证 | **golang-jwt** | JWT 签发/验签 |
| 日志 | **zap / slog** | 结构化日志 |
| API 文档 | **swaggo/swag** (swag init) | OpenAPI 自动生成 |

---

## Part 2: 目录结构

> 基础目录以 [Standard Go Project Layout](https://github.com/golang-standards/project-layout) 为准；`internal/` 在此基础上按业务模块（`auth`、`system` 等）划分，模块内含 handler/model/service/router。

```
├── main.go                         # 程序入口
├── internal/                       # 私有代码（Go 编译器强制不可被外部导入）
│   ├── auth/                       # 认证模块
│   │   ├── handler/{auth_handler, wxma_handler}.go
│   │   ├── model/{request, captcha, wxma_model}.go
│   │   ├── service/{auth_service, wxma_service}.go
│   │   └── router.go
│   ├── system/                     # 系统管理（按实体分子模块）
│   │   ├── user/                   # handler/user_handler.go + model/{entity,form,query,vo}.go + repository + service
│   │   ├── role/                   # 同上 + role_perms_repo, cache_service
│   │   ├── menu/ dept/ dict/ config/ notice/ log/
│   │   └── router.go
│   ├── codegen/                    # 代码生成器
│   ├── file/                       # 文件模块
│   ├── message/                    # 消息模块（SSE 推送）
│   ├── common/                     # 公共组件
│   │   ├── auth/                   # JWT / Redis Token 管理 + Middleware
│   │   ├── config/                 # 全局配置加载
│   │   ├── context/                # 上下文（用户信息）
│   │   ├── database/               # GORM + 分页
│   │   ├── permission/             # 权限服务 + 数据权限
│   │   ├── redis/                  # Redis 客户端
│   │   ├── storage/                # 文件存储（本地/阿里云）
│   │   ├── validator/              # 自定义校验器
│   │   └── response.go             # 统一响应
│   ├── middleware/                  # 全局中间件（error_handler, permission, rate_limiter, requestid）
│   └── router/register.go          # 路由注册入口
├── pkg/                            # 公共库（可被外部项目导入）
│   ├── constant/{business, error_code}.go
│   ├── enums/{action_type, log_module}.go
│   ├── errs/errs.go                # 统一错误定义
│   ├── model/{entity, option, pagination}.go
│   └── types/{bigint, time}.go
├── configs/                        # 配置文件（dev/prod/test.yaml）
├── docs/                           # Swagger 文档（swag init -o docs 生成）
├── docker/docker-compose.yml
├── go.mod / go.sum
```

**设计原则**：
- `internal/` 不可被外部项目导入（Go 编译器强约束）
- `pkg/` 可被外部项目复用
- 每业务模块按 Handler → Service → Repository 四层架构，model 放 DTO/Entity/VO
- `internal/common/` 是内部公共基础设施，不依赖业务模块

| 层 | 目录 | 职责 |
|----|------|------|
| Handler | `handler/` | HTTP 请求处理 |
| Model | `model/` | 实体、DTO、VO、查询 |
| Repository | `repository/` | 数据访问（GORM 查询） |
| Service | `service/` | 业务逻辑 |

---

## Part 3: 命名规范

### 3.1 文件命名

| 类型 | 规范 | 示例 |
|------|------|------|
| Handler | `{name}_handler.go` | `user_handler.go` |
| Service | `{name}_service.go` | `user_service.go` |
| Repository | `{name}_repo.go` | `user_repo.go` |
| Entity | `{name}.go` | `user.go` |
| DTO/VO | `{name}.go` 在 model/ 下 | `user_form.go`, `user_vo.go` |

### 3.2 命名原则

- **简洁不冗余**：包名已提供上下文，类型名不要重复包名
- 导出（大写）`GetByID`，非导出（小写）`parseToken`
- `context.Context` 始终作为函数第一个参数

```go
// ❌ 冗余：包名 user 已表明上下文
package user
type UserService struct{}  // → 调用时变成 user.UserService

// ✅ 简洁
package user
type Service struct{}      // → 调用时 user.Service，包名已提供语义
```

### 3.3 方法命名

| 动作 | 前缀 | 示例 |
|------|------|------|
| 查询单个 | Get | `GetByID(ctx, id)` |
| 查询列表 | List | `List(ctx, query)` |
| 分页查询 | Page | `Page(ctx, query)` |
| 新增 | Create | `Create(ctx, form)` |
| 更新 | Update | `Update(ctx, form)` |
| 删除 | Delete | `Delete(ctx, id)` |

### 3.4 变量命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 变量 | camelCase | `userList` |
| 常量 | PascalCase | `MaxSize` |
| 私有变量 | 小写开头 | `internalState` |
| 布尔值 | Is/Has/Can 前缀 | `isDeleted`, `hasPermission` |

### 3.5 import 排序

```go
import (
    // 1. 标准库
    "context"
    "time"

    // 2. 第三方库
    "github.com/gin-gonic/gin"
    "gorm.io/gorm"

    // 3. 本项目内部
    "youlai/gin/internal/common/response"
    "youlai/gin/pkg/errs"
)
```

---

## Part 4: RESTful API 规范

### 4.1 标准 CRUD 路径

| 操作 | 方法 | 路径 |
|------|------|------|
| 分页列表 | `GET` | `/api/v1/users/page` |
| 详情 | `GET` | `/api/v1/users/:id` |
| 新增 | `POST` | `/api/v1/users` |
| 更新 | `PUT` | `/api/v1/users` |
| 删除 | `DELETE` | `/api/v1/users/:id` |
| 批量删除 | `DELETE` | `/api/v1/users/batch` |
| 下拉选项 | `GET` | `/api/v1/users/options` |

### 4.2 Handler 模板

```go
// Page 用户分页列表
// @Summary 用户分页列表
// @Tags 用户管理
// @Accept json
// @Produce json
// @Param query query model.UserPageQuery false "查询参数"
// @Success 200 {object} response.PageResult
// @Router /api/v1/users/page [get]
func (h *UserHandler) Page(c *gin.Context) {
    var query model.UserPageQuery
    if err := c.ShouldBindQuery(&query); err != nil {
        c.Error(errs.BadRequest(err.Error()))
        return
    }
    result := h.service.Page(c.Request.Context(), &query)
    response.Success(c, result)
}

// Create 新增用户
// @Summary 新增用户
// @Tags 用户管理
// @Param form body model.UserForm true "用户表单"
// @Security Bearer
// @Router /api/v1/users [post]
func (h *UserHandler) Create(c *gin.Context) {
    var form model.UserForm
    if err := c.ShouldBindJSON(&form); err != nil {
        c.Error(errs.BadRequest(err.Error()))
        return
    }
    id := h.service.Create(c.Request.Context(), &form)
    response.Success(c, id)
}
```

---

## Part 5: 响应格式与异常处理

### 5.1 统一响应

```go
type Result struct {
    Code int         `json:"code"`
    Msg  string      `json:"msg"`
    Data interface{} `json:"data"`
}

type PageResult struct {
    Code int `json:"code"`
    Msg  string `json:"msg"`
    Data struct {
        List  interface{} `json:"list"`
        Total int64       `json:"total"`
    } `json:"data"`
}

func Success(c *gin.Context, data interface{}) {
    c.JSON(200, Result{Code: 200, Msg: "操作成功", Data: data})
}

func PageSuccess(c *gin.Context, list interface{}, total int64) {
    r := PageResult{Code: 200, Msg: "操作成功"}
    r.Data.List, r.Data.Total = list, total
    c.JSON(200, r)
}
```

### 5.2 统一错误处理

使用 `c.Error(err)` 配合全局错误处理中间件，统一返回错误响应。

```go
type AppError struct {
    Code       string `json:"code"`
    Msg        string `json:"msg"`
    HTTPStatus int    `json:"-"`  // 不序列化到响应
}

func BadRequest(msg string) *AppError       { return &AppError{Code: "A0400", Msg: msg, HTTPStatus: 400} }
func Unauthorized(msg string) *AppError     { return &AppError{Code: "A0230", Msg: msg, HTTPStatus: 401} }
func Forbidden(msg string) *AppError        { return &AppError{Code: "A0301", Msg: msg, HTTPStatus: 403} }
func SystemError(msg string) *AppError      { return &AppError{Code: "B0001", Msg: msg, HTTPStatus: 500} }
```

### 5.3 错误处理中间件

```go
func ErrorHandler() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Next()
        if len(c.Errors) > 0 {
            if appErr, ok := c.Errors.Last().Err.(*errs.AppError); ok {
                response.FromAppError(c, appErr)
            } else {
                response.Fail(c, "系统错误")
            }
        }
    }
}
```

### 5.4 Handler 错误处理规范

> **禁止**直接使用 `response.Fail(c, "参数错误")`，应使用 `c.Error(errs.BadRequest("具体错误信息"))`。

```go
// ✅ 正确
func GetUserForm(c *gin.Context) {
    userId, err := strconv.ParseInt(c.Param("userId"), 10, 64)
    if err != nil {
        c.Error(errs.BadRequest("无效的用户ID"))
        return
    }
}

// ❌ 错误：忽略错误
func GetUserForm(c *gin.Context) {
    userId, _ := strconv.ParseInt(c.Param("userId"), 10, 64)
}
```

---

## Part 6: 实体规范

```go
type BaseEntity struct {
    ID         uint           `gorm:"primaryKey;autoIncrement" json:"id"`
    CreateTime time.Time      `gorm:"autoCreateTime" json:"createTime"`
    UpdateTime time.Time      `gorm:"autoUpdateTime" json:"updateTime"`
    CreateBy   uint           `json:"createBy"`
    UpdateBy   uint           `json:"updateBy"`
    Deleted    gorm.DeletedAt `gorm:"index" json:"-"`
}

type SysUser struct {
    BaseEntity
    Username string `gorm:"size:50;uniqueIndex;not null" json:"username"`
    Password string `gorm:"size:100;not null" json:"-"`
    Nickname string `gorm:"size:50;not null" json:"nickname"`
    Mobile   string `gorm:"size:20" json:"mobile"`
    Email    string `gorm:"size:100" json:"email"`
    Status   int    `gorm:"default:1" json:"status"`
    DeptID   uint   `json:"deptId"`
}

func (SysUser) TableName() string { return "sys_user" }
```

---

## Part 7: Service 规范

```go
type UserService struct {
    repo *repository.UserRepo
}

func NewUserService(repo *repository.UserRepo) *UserService {
    return &UserService{repo: repo}
}

func (s *UserService) Page(ctx context.Context, query *model.UserPageQuery) *model.PageResult {
    users, total := s.repo.Page(ctx, query)
    list := make([]model.UserVO, len(users))
    for i, u := range users {
        list[i] = model.UserVO{ID: u.ID, Username: u.Username, Nickname: u.Nickname, Status: u.Status}
    }
    return &model.PageResult{List: list, Total: total}
}

func (s *UserService) Create(ctx context.Context, form *model.UserForm) uint {
    if s.repo.ExistsByUsername(ctx, form.Username) {
        panic(errs.BadRequest("用户名已存在"))
    }
    user := &model.SysUser{Username: form.Username, Password: bcrypt.Hash(form.Password), Status: form.Status}
    s.repo.Create(ctx, user)
    return user.ID
}
```

---

## Part 8: 认证规范

### 8.1 JWT 中间件

```go
func JWTAuth() gin.HandlerFunc {
    return func(c *gin.Context) {
        token := strings.TrimPrefix(c.GetHeader("Authorization"), "Bearer ")
        if token == "" { response.Unauthorized(c); c.Abort(); return }
        claims, err := auth.ParseToken(token)
        if err != nil { response.Unauthorized(c); c.Abort(); return }
        c.Set("userId", claims.UserID)
        c.Set("username", claims.Username)
        c.Next()
    }
}
```

### 8.2 权限中间件

```go
func Permission(required ...string) gin.HandlerFunc {
    return func(c *gin.Context) {
        permissions := c.GetStringSlice("permissions")
        for _, p := range required {
            for _, perm := range permissions {
                if perm == p { c.Next(); return }
            }
        }
        response.Forbidden(c)
        c.Abort()
    }
}
```

### 8.3 权限标识格式

```
模块:实体:操作
sys:user:create / sys:user:edit / sys:user:delete / sys:role:list
```

---

## Part 9: 注释规范

- **导出函数必须有 godoc 注释**
- Swagger 使用 `@Summary` / `@Tags` / `@Param` / `@Success` 注解
- Chinese OK：核心解释用中文，信号字（godoc 关键字）保留英文

```go
// UserHandler 用户管理 HTTP 处理器。
type UserHandler struct {
    service *service.UserService
}

// Page 分页查询用户列表。
//
// 根据关键词模糊搜索用户名/昵称/手机号，支持状态筛选。
//
// @Summary 用户分页列表
// @Tags 用户管理
// @Param query query model.UserPageQuery false "查询参数"
// @Success 200 {object} response.PageResult
// @Router /api/v1/users/page [get]
func (h *UserHandler) Page(c *gin.Context) { ... }
```

---

## Part 10: 代码质量检查清单

- [ ] 遵循 RESTful API 路径规范
- [ ] 使用统一响应格式 + 错误处理中间件
- [ ] 使用 `c.Error(errs.BadRequest(...))` 而非 `response.Fail`
- [ ] `strconv.ParseInt/Atoi` 必须检查错误
- [ ] 文件上传校验大小和类型
- [ ] 实体继承 `BaseEntity`，GORM 软删除 `DeletedAt`
- [ ] API 添加 Swaggo 注释（`@Summary/@Tags/@Param`）
- [ ] 使用 `gofmt` 格式化 + CI `golangci-lint` 检查
- [ ] import 分组有序（标准库 → 第三方 → 内部）
- [ ] `context.Context` 作为函数第一个参数
- [ ] 导出函数有 godoc 注释

### 常见反模式

| 反模式 | 正确做法 |
|--------|----------|
| `fmt.Printf("debug: %v", data)` | 用 `slog/zap` 结构化日志 |
| `panic(err)` 直接 panic | 用 `c.Error(errs.BadRequest(...))` 走中间件 |
| `strconv.ParseInt(s, 10, 64)` 忽略 err | 必须检查错误 |
| Context 塞进结构体字段 | Context 作为函数第一个参数传入 |
| `if err != nil { log.Error(); return err }` 层层 log | 只在顶层 log 一次，中间用 `fmt.Errorf("...: %w", err)` 包装 |
| HTTP 调用无超时 | `context.WithTimeout(ctx, 30*time.Second)` |
| 堆栈信息返回给 API 调用方 | 错误信息脱敏，堆栈只写日志 |
