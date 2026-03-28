---
name: gin
description: Gin 后端开发规范。当开发 Gin 项目、实现 REST API、GORM 数据访问、JWT 认证、权限控制时使用此 skill。
---

# Gin 后端开发规范

## 触发条件

- 开发 Gin 项目
- 实现 REST API
- 使用 GORM 数据访问
- 实现 JWT 认证
- 实现权限控制

---

## Part 1: 技术栈

- **Go** - 运行环境
- **Gin** - Web 框架
- **GORM** - ORM 框架
- **MySQL** - 数据库
- **Redis** - 缓存
- **JWT** - Token 认证
- **Swagger** - API 文档
- **zap/logrus** - 结构化日志

---

## Part 2: 目录结构

遵循 [Standard Go Project Layout](https://github.com/golang-standards/project-layout) 社区标准。

> **简单单体项目**：`main.go` 可直接放根目录，无需 `cmd/` 目录。`cmd/` 适用于多入口项目（如同时有 API 服务、CLI 工具、定时任务）。

```
├── main.go                     # 程序入口（简单单体项目放根目录）
├── internal/                   # 私有代码（Go 编译器强制不可被外部导入）
│   ├── app/                    # 应用层
│   │   └── myapp/
│   │       ├── handler/        # HTTP 处理器（Controller）
│   │       │   ├── user.go
│   │       │   └── auth.go
│   │       └── router/         # 路由注册
│   │           └── router.go
│   ├── domain/                 # 领域层
│   │   ├── entity/             # 实体
│   │   │   ├── base.go
│   │   │   └── user.go
│   │   ├── dto/                # 数据传输对象
│   │   │   ├── user_dto.go
│   │   │   └── page_dto.go
│   │   └── vo/                 # 视图对象
│   │       └── user_vo.go
│   ├── infra/                  # 基础设施层
│   │   ├── repository/         # 数据访问
│   │   │   └── user_repo.go
│   │   ├── middleware/         # 中间件
│   │   │   ├── jwt.go
│   │   │   ├── permission.go
│   │   │   └── cors.go
│   │   └── persistence/        # 持久化配置
│   │       └── mysql.go
│   └── service/                # 业务逻辑层
│       └── user_service.go
├── pkg/                        # 公共库（可被外部项目导入）
│   ├── response/               # 统一响应
│   │   └── result.go
│   ├── auth/                   # 认证工具
│   │   └── jwt.go
│   ├── cache/                  # 缓存封装
│   │   └── redis.go
│   ├── errs/                   # 错误定义
│   │   └── errs.go
│   └── utils/                  # 通用工具
│       └── bcrypt.go
├── api/                        # Swagger 文档（swag init -o api 生成）
│   ├── docs.go
│   ├── swagger.json
│   └── swagger.yaml
├── configs/                    # 配置文件模板或默认配置
│   └── config.yaml
├── docker/                     # Docker 配置
│   └── docker-compose.yml
├── sql/                        # 数据库脚本
│   └── mysql/
│       └── schema.sql
├── go.mod
└── go.sum
```

### 目录说明

| 目录           | 用途                                  | 规则                         |
| -------------- | ------------------------------------- | ---------------------------- |
| `/cmd`         | 主干程序入口                          | 每个应用一个子目录，代码精简 |
| `/internal`    | 私有代码                              | Go 编译器强制不可被外部导入  |
| `/pkg`         | 公共库                                | 可被外部项目导入使用         |
| `/api`         | API 定义（OpenAPI/Swagger、协议定义） | 存放接口规范文件             |
| `/configs`     | 配置文件模板或默认配置                | 不含敏感信息                 |
| `/scripts`     | 构建、安装、分析脚本                  | 保持 Makefile 简洁           |
| `/build`       | 打包和 CI 配置                        | Docker、OS 包配置            |
| `/deployments` | 部署配置                              | IaaS/PaaS/K8s 配置           |
| `/test`        | 额外测试应用和测试数据                | 大项目可设 data 子目录       |
| `/docs`        | 设计和用户文档                        | 除 godoc 外的文档            |
| `/tools`       | 项目支持工具                          | 可导入 internal/pkg          |

### 禁止使用的目录

| 目录   | 原因                       |
| ------ | -------------------------- |
| `/src` | Java 风格，Go 项目不应使用 |

---

## Part 3: 命名规范

### 文件命名

| 类型   | 规范             | 示例              |
| ------ | ---------------- | ----------------- |
| 控制器 | 小写 + \_        | `user.go`         |
| 服务   | 小写 + \_service | `user_service.go` |
| 仓库   | 小写 + \_repo    | `user_repo.go`    |
| 实体   | 小写             | `user.go`         |
| DTO    | 小写 + \_dto     | `user_dto.go`     |
| VO     | 小写 + \_vo      | `user_vo.go`      |

### 结构体命名

| 类型       | 规范           | 示例                        |
| ---------- | -------------- | --------------------------- |
| 实体       | PascalCase     | `User` 或 `SysUser`         |
| DTO        | 功能 + DTO     | `UserForm`, `UserPageQuery` |
| VO         | 功能 + VO      | `UserVO`                    |
| Service    | 实体 + Service | `UserService`               |
| Repository | 实体 + Repo    | `UserRepo`                  |

### 方法命名

| 动作     | 前缀   | 示例           |
| -------- | ------ | -------------- |
| 查询单个 | Get    | `GetByID()`    |
| 查询列表 | List   | `ListUsers()`  |
| 分页查询 | Page   | `PageUsers()`  |
| 新增     | Create | `CreateUser()` |
| 更新     | Update | `UpdateUser()` |
| 删除     | Delete | `DeleteByID()` |

### 变量命名

| 类型     | 规范                           | 示例                         |
| -------- | ------------------------------ | ---------------------------- |
| 变量     | camelCase                      | `userList`                   |
| 常量     | PascalCase 或 UPPER_SNAKE_CASE | `MaxSize` 或 `MAX_SIZE`      |
| 私有变量 | 小写开头                       | `internalState`              |
| 布尔值   | Is/Has/Can 前缀                | `IsDeleted`, `HasPermission` |

---

## Part 4: RESTful API 规范

### 标准 CRUD 路径

| 操作     | 方法   | 路径                    |
| -------- | ------ | ----------------------- |
| 分页列表 | GET    | `/api/v1/users/page`    |
| 详情     | GET    | `/api/v1/users/:id`     |
| 新增     | POST   | `/api/v1/users`         |
| 更新     | PUT    | `/api/v1/users`         |
| 删除     | DELETE | `/api/v1/users/:id`     |
| 批量删除 | DELETE | `/api/v1/users/batch`   |
| 下拉选项 | GET    | `/api/v1/users/options` |

### Controller 模板

```go
// api/v1/user.go
package v1

import (
    "github.com/gin-gonic/gin"
    "youlai/internal/model/dto"
    "youlai/internal/service"
    "youlai/pkg/response"
)

type UserAPI struct {
    userService *service.UserService
}

func NewUserAPI() *UserAPI {
    return &UserAPI{
        userService: service.NewUserService(),
    }
}

// Page godoc
// @Summary 用户分页列表
// @Tags 用户管理
// @Accept json
// @Produce json
// @Param query query dto.UserPageQuery false "查询参数"
// @Success 200 {object} response.PageResult
// @Router /api/v1/users/page [get]
func (api *UserAPI) Page(c *gin.Context) {
    var query dto.UserPageQuery
    if err := c.ShouldBindQuery(&query); err != nil {
        response.ParamError(c, err.Error())
        return
    }
    result := api.userService.PageUsers(&query)
    response.Success(c, result)
}

// GetByID godoc
// @Summary 用户详情
// @Tags 用户管理
// @Produce json
// @Param id path int true "用户ID"
// @Success 200 {object} response.Result
// @Router /api/v1/users/{id} [get]
func (api *UserAPI) GetByID(c *gin.Context) {
    id := c.Param("id")
    user := api.userService.GetUserByID(id)
    response.Success(c, user)
}

// Create godoc
// @Summary 新增用户
// @Tags 用户管理
// @Accept json
// @Produce json
// @Param form body dto.UserForm true "用户表单"
// @Success 200 {object} response.Result
// @Router /api/v1/users [post]
// @Security Bearer
func (api *UserAPI) Create(c *gin.Context) {
    var form dto.UserForm
    if err := c.ShouldBindJSON(&form); err != nil {
        response.ParamError(c, err.Error())
        return
    }
    id := api.userService.CreateUser(&form)
    response.Success(c, id)
}

// Update godoc
// @Summary 更新用户
// @Tags 用户管理
// @Accept json
// @Produce json
// @Param form body dto.UserForm true "用户表单"
// @Success 200 {object} response.Result
// @Router /api/v1/users [put]
// @Security Bearer
func (api *UserAPI) Update(c *gin.Context) {
    var form dto.UserForm
    if err := c.ShouldBindJSON(&form); err != nil {
        response.ParamError(c, err.Error())
        return
    }
    api.userService.UpdateUser(&form)
    response.Success(c, nil)
}

// Delete godoc
// @Summary 删除用户
// @Tags 用户管理
// @Produce json
// @Param id path int true "用户ID"
// @Success 200 {object} response.Result
// @Router /api/v1/users/{id} [delete]
// @Security Bearer
func (api *UserAPI) Delete(c *gin.Context) {
    id := c.Param("id")
    api.userService.DeleteUserByID(id)
    response.Success(c, nil)
}
```

---

## Part 5: 响应格式

```go
// pkg/response/result.go
package response

import "github.com/gin-gonic/gin"

type Result struct {
    Code int         `json:"code"`
    Msg  string      `json:"msg"`
    Data interface{} `json:"data"`
}

type PageResult struct {
    Code int         `json:"code"`
    Msg  string      `json:"msg"`
    Data struct {
        List  interface{} `json:"list"`
        Total int64       `json:"total"`
    } `json:"data"`
}

func Success(c *gin.Context, data interface{}) {
    c.JSON(200, Result{
        Code: 200,
        Msg:  "操作成功",
        Data: data,
    })
}

func PageSuccess(c *gin.Context, list interface{}, total int64) {
    c.JSON(200, PageResult{
        Code: 200,
        Msg:  "操作成功",
        Data: struct {
            List  interface{} `json:"list"`
            Total int64       `json:"total"`
        }{
            List:  list,
            Total: total,
        },
    })
}

func ParamError(c *gin.Context, msg string) {
    c.JSON(200, Result{
        Code: 400,
        Msg:  msg,
        Data: nil,
    })
}

func Unauthorized(c *gin.Context) {
    c.JSON(200, Result{
        Code: 401,
        Msg:  "未登录或token过期",
        Data: nil,
    })
}

func Forbidden(c *gin.Context) {
    c.JSON(200, Result{
        Code: 403,
        Msg:  "无权限",
        Data: nil,
    })
}
```

---

## Part 6: 实体规范

### 基础实体

```go
// internal/model/entity/base.go
package entity

import (
    "time"
    "gorm.io/gorm"
)

type BaseEntity struct {
    ID         uint           `gorm:"primaryKey;autoIncrement" json:"id"`
    CreateTime time.Time      `gorm:"autoCreateTime" json:"createTime"`
    UpdateTime time.Time      `gorm:"autoUpdateTime" json:"updateTime"`
    CreateBy   uint           `gorm:"default:null" json:"createBy"`
    UpdateBy   uint           `gorm:"default:null" json:"updateBy"`
    Deleted    gorm.DeletedAt `gorm:"index" json:"-"`
}
```

### 实体示例

```go
// internal/model/entity/user.go
package entity

type SysUser struct {
    BaseEntity
    Username string `gorm:"size:50;uniqueIndex;not null" json:"username"`
    Password string `gorm:"size:100;not null" json:"-"`
    Nickname string `gorm:"size:50;not null" json:"nickname"`
    Mobile   string `gorm:"size:20" json:"mobile"`
    Email    string `gorm:"size:100" json:"email"`
    Gender   int    `gorm:"default:0" json:"gender"`
    Status   int    `gorm:"default:1" json:"status"`
    DeptID   uint   `gorm:"default:null" json:"deptId"`
}

func (SysUser) TableName() string {
    return "sys_user"
}
```

---

## Part 7: Service 规范

```go
// internal/service/user_service.go
package service

import (
    "youlai/internal/model/dto"
    "youlai/internal/model/entity"
    "youlai/internal/model/vo"
    "youlai/internal/repository"
)

type UserService struct {
    userRepo *repository.UserRepo
}

func NewUserService() *UserService {
    return &UserService{
        userRepo: repository.NewUserRepo(),
    }
}

func (s *UserService) PageUsers(query *dto.UserPageQuery) *dto.PageResult {
    users, total := s.userRepo.Page(query)
    list := make([]vo.UserVO, len(users))
    for i, user := range users {
        list[i] = vo.UserVO{
            ID:         user.ID,
            Username:   user.Username,
            Nickname:   user.Nickname,
            Mobile:     user.Mobile,
            Email:      user.Email,
            Status:     user.Status,
            CreateTime: user.CreateTime,
        }
    }
    return &dto.PageResult{
        List:  list,
        Total: total,
    }
}

func (s *UserService) GetUserByID(id string) *vo.UserVO {
    user := s.userRepo.GetByID(id)
    if user == nil {
        panic("用户不存在")
    }
    return &vo.UserVO{
        ID:         user.ID,
        Username:   user.Username,
        Nickname:   user.Nickname,
        Mobile:     user.Mobile,
        Email:      user.Email,
        Status:     user.Status,
        CreateTime: user.CreateTime,
    }
}

func (s *UserService) CreateUser(form *dto.UserForm) uint {
    // 检查用户名是否存在
    if s.userRepo.ExistsByUsername(form.Username) {
        panic("用户名已存在")
    }
    user := &entity.SysUser{
        Username: form.Username,
        Password: bcrypt.Hash(form.Password),
        Nickname: form.Nickname,
        Mobile:   form.Mobile,
        Email:    form.Email,
        Status:   form.Status,
    }
    s.userRepo.Create(user)
    return user.ID
}
```

---

## Part 8: 认证规范

### JWT 中间件

```go
// internal/middleware/jwt.go
package middleware

import (
    "strings"
    "github.com/gin-gonic/gin"
    "youlai/pkg/auth"
    "youlai/pkg/response"
)

func JWTAuth() gin.HandlerFunc {
    return func(c *gin.Context) {
        token := c.GetHeader("Authorization")
        if token == "" {
            response.Unauthorized(c)
            c.Abort()
            return
        }

        token = strings.TrimPrefix(token, "Bearer ")
        claims, err := auth.ParseToken(token)
        if err != nil {
            response.Unauthorized(c)
            c.Abort()
            return
        }

        c.Set("userId", claims.UserID)
        c.Set("username", claims.Username)
        c.Next()
    }
}
```

### 权限中间件

```go
// internal/middleware/permission.go
package middleware

import (
    "github.com/gin-gonic/gin"
    "youlai/pkg/response"
)

func Permission(required ...string) gin.HandlerFunc {
    return func(c *gin.Context) {
        permissions := c.GetStringSlice("permissions")
        for _, p := range required {
            for _, perm := range permissions {
                if perm == p {
                    c.Next()
                    return
                }
            }
        }
        response.Forbidden(c)
        c.Abort()
    }
}
```

---

## Part 9: 路由注册

```go
// internal/router/router.go
package router

import (
    "github.com/gin-gonic/gin"
    v1 "youlai/api/v1"
    "youlai/internal/middleware"
)

func SetupRouter() *gin.Engine {
    r := gin.New()
    r.Use(middleware.CORS())

    // API v1
    api := r.Group("/api/v1")
    {
        // 认证接口（无需登录）
        auth := v1.NewAuthAPI()
        api.POST("/auth/login", auth.Login)
        api.POST("/auth/refresh-token", auth.RefreshToken)

        // 需要登录的接口
        authorized := api.Use(middleware.JWTAuth())
        {
            user := v1.NewUserAPI()
            users := authorized.Group("/users")
            {
                users.GET("/page", user.Page)
                users.GET("/:id", user.GetByID)
                users.GET("/options", user.Options)
                users.POST("", middleware.Permission("sys:user:add"), user.Create)
                users.PUT("", middleware.Permission("sys:user:edit"), user.Update)
                users.DELETE("/:id", middleware.Permission("sys:user:delete"), user.Delete)
            }
        }
    }

    return r
}
```

---

## Part 10: 错误处理规范

### 统一错误处理机制

使用 `c.Error(err)` 配合全局错误处理中间件，统一返回错误响应。

```go
// pkg/errs/errs.go
package errs

import (
    "net/http"
    "youlai-gin/pkg/constant"
)

type AppError struct {
    Code       string `json:"code"`
    Msg        string `json:"msg"`
    HTTPStatus int    `json:"-"`
    Err        error  `json:"-"`
}

func BadRequest(msg string) *AppError {
    return &AppError{
        Code:       constant.CodeBadRequest,
        Msg:        msg,
        HTTPStatus: http.StatusBadRequest,
    }
}

func SystemError(msg string) *AppError {
    return &AppError{
        Code:       constant.CodeSystemError,
        Msg:        msg,
        HTTPStatus: http.StatusInternalServerError,
    }
}

func NotFound(msg string) *AppError {
    return &AppError{
        Code:       constant.CodeBadRequest,
        Msg:        msg,
        HTTPStatus: http.StatusBadRequest,
    }
}

func Unauthorized(msg string) *AppError {
    return &AppError{
        Code:       constant.CodeAccessUnauthorized,
        Msg:        msg,
        HTTPStatus: http.StatusUnauthorized,
    }
}
```

### 错误处理中间件

```go
// pkg/middleware/error_handler.go
package middleware

import (
    "github.com/gin-gonic/gin"
    "youlai-gin/pkg/errs"
    "youlai-gin/pkg/response"
)

func ErrorHandler() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Next()

        if len(c.Errors) > 0 {
            err := c.Errors.Last().Err
            if appErr, ok := err.(*errs.AppError); ok {
                response.FromAppError(c, appErr)
            } else {
                response.Fail(c, "系统错误")
            }
        }
    }
}
```

### Handler 错误处理规范

**禁止**直接使用 `response.Fail(c, "参数错误")`，应使用 `c.Error(errs.BadRequest("具体错误信息"))`。

```go
// 正确示例
func GetUserForm(c *gin.Context) {
    userIdStr := c.Param("userId")
    userId, err := strconv.ParseInt(userIdStr, 10, 64)
    if err != nil {
        c.Error(errs.BadRequest("无效的用户ID"))
        return
    }
    // ...
}

// 错误示例
func GetUserForm(c *gin.Context) {
    userIdStr := c.Param("userId")
    userId, _ := strconv.ParseInt(userIdStr, 10, 64)  // 禁止忽略错误
    // ...
}
```

### 参数校验规范

使用 `validator.BindQuery/BindJSON` 统一参数校验，禁止直接使用 `c.ShouldBindQuery/ShouldBindJSON`。

```go
// pkg/validator/validator.go
package validator

import (
    "github.com/gin-gonic/gin"
    "youlai-gin/pkg/errs"
)

func BindQuery(c *gin.Context, obj interface{}) *errs.AppError {
    if err := c.ShouldBindQuery(obj); err != nil {
        return errs.BadRequest("参数格式错误: " + err.Error())
    }
    return nil
}

func BindJSON(c *gin.Context, obj interface{}) *errs.AppError {
    if err := c.ShouldBindJSON(obj); err != nil {
        return errs.BadRequest("请求体格式错误: " + err.Error())
    }
    return nil
}
```

```go
// Handler 使用示例
func GetUserList(c *gin.Context) {
    var query api.UserQueryReq
    if err := validator.BindQuery(c, &query); err != nil {
        c.Error(err)
        return
    }
    // ...
}
```

### 必填参数校验

Handler 层应对必填参数进行显式校验：

```go
func SendMobileCode(c *gin.Context) {
    mobile := c.Query("mobile")
    if mobile == "" {
        c.Error(errs.BadRequest("手机号不能为空"))
        return
    }
    // ...
}
```

### 文件上传限制

文件上传必须校验大小和类型：

```go
const (
    maxUploadSize     = 10 << 20 // 10MB
    allowedExcelTypes = ".xlsx"
)

func ImportUsers(c *gin.Context) {
    file, err := c.FormFile("file")
    if err != nil {
        c.Error(errs.BadRequest("请选择要导入的文件"))
        return
    }

    if file.Size > maxUploadSize {
        c.Error(errs.BadRequest("文件大小不能超过10MB"))
        return
    }

    if !isExcelFile(file.Filename) {
        c.Error(errs.BadRequest("仅支持.xlsx格式的Excel文件"))
        return
    }
    // ...
}
```

### 日志规范

**禁止**使用 `fmt.Printf`，应使用 `slog`：

```go
// 正确示例
slog.Error("降级查询数据库失败", "roles", missingRoles, "error", err)
slog.Info("短信验证码已发送", "mobile", mobile, "code", code)

// 错误示例
fmt.Printf("降级查询数据库失败，角色: %v, 错误: %v\n", missingRoles, err)
```

### 错误信息脱敏

禁止向客户端返回内部错误详情：

```go
// 正确示例
if err != nil {
    slog.Error("获取微信会话信息失败", "code", code, "error", err)
    return nil, errs.BadRequest("微信登录失败，请稍后重试")
}

// 错误示例
if err != nil {
    return nil, errs.BadRequest("微信登录失败：" + err.Error())  // 泄露内部错误
}
```

---

## Part 11: 代码质量检查清单

### 基础规范

- [ ] 遵循 RESTful API 路径规范
- [ ] 使用统一响应格式
- [ ] 实现错误处理中间件
- [ ] 添加权限中间件
- [ ] 分页接口使用 PageResult
- [ ] 实体继承 BaseEntity
- [ ] 使用 GORM 软删除
- [ ] API 添加 Swagger 注释
- [ ] 分层：Controller → Service → Repository
- [ ] 使用依赖注入

### 参数与错误处理

- [ ] strconv.ParseInt/Atoi 必须检查错误
- [ ] 使用 c.Error(errs.BadRequest) 替代 response.Fail
- [ ] 使用 validator.BindQuery/BindJSON 统一参数校验
- [ ] 必填参数显式校验
- [ ] 文件上传校验大小和类型
- [ ] 错误信息脱敏，禁止返回内部错误详情

### 日志规范

- [ ] 使用 slog 替代 fmt.Printf
- [ ] 使用结构化日志（zap/logrus）
- [ ] 错误只在顶层 log 一次
- [ ] 堆栈信息不返回给 API 调用方

### 代码组织

- [ ] 使用 gofmt 格式化代码
- [ ] CI 配置 golangci-lint 检查
- [ ] import 分组有序（标准库 → 第三方 → 内部）
- [ ] 文件内代码布局：const/var → struct/interface → 导出方法 → 私有方法
- [ ] 命名简洁不冗余（包名已提供上下文）

### Context 与 HTTP

- [ ] Context 作为函数第一个参数传入
- [ ] HTTP 调用使用 context.WithTimeout 控制超时

### 测试规范

- [ ] 使用表驱动测试
- [ ] 使用 testify 进行断言
- [ ] 避免使用 gomock.Any()，显式指定参数

---

## Part 12: 编译验证

修改代码后必须执行编译验证：

```bash
cd <项目目录> && go build ./...
```

确保无编译错误后再提交。

---

## Part 13: 最佳实践

### 工具链

| 工具            | 用途       | 要求                       |
| --------------- | ---------- | -------------------------- |
| `gofmt`         | 代码格式化 | 必须使用，消灭缩进争论     |
| `golangci-lint` | 静态检查   | CI 必须在 lint 失败时红线  |
| `go:generate`   | Mock 生成  | 配合 mockgen 自动生成 mock |

### Import 顺序

import 语句分组有序：

```go
import (
    // 1. 标准库
    "context"
    "time"

    // 2. 第三方库
    "github.com/gin-gonic/gin"
    "github.com/sirupsen/logrus"

    // 3. 公司/组织内部库
    "github.company.com/org/repo/gapi"

    // 4. 本项目内部包
    "youlai/internal/service"
    "youlai/pkg/response"
)
```

### 单个文件内的代码布局

一个 `.go` 文件按以下顺序组织：

1. `const` / `var` 定义
2. `struct` / `interface` 定义
3. 导出的（大写）方法
4. 非导出的（小写）方法
5. 辅助函数

### 命名：简单、不冗余

包名已提供上下文，不要在类型名里重复。

```go
// ❌ 冗余写法
type DeploymentTransformerHandlerIntf interface{}
type DeploymentTransformerHandler struct{}

// ✅ 简洁写法
type DeploymentTransformerIntf interface{}
type DeploymentTransformer struct{}
```

### Context：传参，不要塞结构体

永远把 `context.Context` 作为函数的第一个参数传入。

```go
// ❌ 错误做法
type Client struct {
    ctx context.Context
}
func (c *Client) DoSomething(foo string) {
    // 用 c.ctx —— 千万别这样！
}

// ✅ 正确做法
func (c *Client) DoSomething(ctx context.Context, foo string) {
    // 用 ctx
}
```

**铁律**：只要方法需要 context，一律放在第一个参数。

### 函数选项模式（Functional Options）

参数多时推荐使用函数选项模式，可读且灵活。

```go
// ❌ 灾难签名
svr := server.New("localhost", 8080, time.Minute, 120)

// ✅ 自解释写法
svr := server.New(
    server.WithHost("localhost"),
    server.WithPort(8080),
    server.WithTimeout(time.Minute),
    server.WithMaxConn(120),
)
```

后续加参数也不破坏已有调用，扩展性极强。

### 错误处理：上下文才是王道

#### 用堆栈 + 错误包装

推荐使用 `github.com/pingcap/errors` 或 `go.uber.org/multierr`，给错误自动带上调用栈。

```go
// ❌ 层层重复 log
if err != nil {
    log.Errorf("ERROR :: Google Docs :: NewGoogleApiHandler :: Unable to create service :: %v", err)
    return nil, err
}

// ✅ 优雅包装向上透传
if err != nil {
    return nil, fmt.Errorf("failed to create google docs service: %w", err)
}
```

**安全红线**：堆栈信息只写日志，绝不返回给 API 调用方（泄露实现细节是安全漏洞）。

#### 日志级别要分清

| 级别  | 场景                        |
| ----- | --------------------------- |
| Error | 服务内部错误（500 场景）    |
| Warn  | 用户/客户端错误（400 场景） |

#### 错误只在顶层 log 一次

层层 log 同一个错误是反模式。顶层统一 log + 带栈，干净又好 debug。

### 日志：结构化 > 字符串拼接

推荐用 `zap` 或 `logrus`（生产环境开 JSON 格式）。

```go
// ❌ 字符串拼接
log.Errorf("Failed to write summary. Error: %s", err)

// ✅ 结构化
log.WithFields(log.Fields{
    "event": event,
    "topic": topic,
    "key":   key,
}).Info("processing new event")
```

结构化日志在 ELK / Loki / 阿里云 SLS 等平台上搜索、告警效率高出几个量级。

#### 请求级日志（带 trace id）

```go
type Handler struct {
    LoggerFn func(ctx context.Context) *logrus.Entry
}

func (h *Handler) CheckHealth(ctx context.Context) {
    logger := h.LoggerFn(ctx)
    logger.Info("health check started")
}
```

中间件把 request-id / trace-id 塞进 context，自动带到所有日志里，实现全链路追踪。

#### 可忽略的已知错误打 Debug

对于可忽略的已知错误，使用 Debug 级别日志：

```go
if err != nil {
    if errors.Is(err, ErrUserAlreadyExists) {
        log.WithError(err).Debug("user exists, continuing")
        return nil
    }
    return nil, fmt.Errorf("failed to add user %s: %w", userID, err)
}
```

### HTTP 调用：优先用 Context 超时

不要依赖 `http.Client` 的全局 Timeout，用 context 控制更精细。

```go
ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
defer cancel()

req, _ := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
resp, err := client.Do(req)
```

### 单元测试：表驱动测试

推荐工具：`testify`（assert、require、mock）

| 函数                 | 用途                       |
| -------------------- | -------------------------- |
| `assert.Equal`       | 基本相等断言               |
| `assert.EqualValues` | map/slice 忽略顺序比较     |
| `assert.EqualError`  | 简单错误比较               |
| `assert.NoError`     | 无错误断言                 |
| `assert.Error`       | 有错误断言                 |
| `require.NoError`    | 无错误断言（失败立即终止） |

```go
func TestToUpper(t *testing.T) {
    tests := []struct {
        name     string
        input    string
        expected string
        wantErr  bool
    }{
        {
            name:     "正常输入",
            input:    "hello",
            expected: "HELLO",
            wantErr:  false,
        },
        {
            name:     "空字符串",
            input:    "",
            expected: "",
            wantErr:  true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := ToUpper(tt.input)
            if tt.wantErr {
                assert.Error(t, err)
                return
            }
            assert.NoError(t, err)
            assert.Equal(t, tt.expected, got)
        })
    }
}
```

**核心原则**：

- 测试用例自己带 mock 和预期
- 循环体只调用被测代码 + 断言，不要写额外逻辑
- 尽量避免 gomock 的 `Any()`，显式指定更安全
