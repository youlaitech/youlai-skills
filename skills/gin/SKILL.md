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

| 技术 | 版本 | 用途 |
|------|------|------|
| Go | 1.21+ | 运行环境 |
| Gin | 1.9+ | Web 框架 |
| GORM | 1.25+ | ORM 框架 |
| MySQL | 8.0+ | 数据库 |
| Redis | 7.0+ | 缓存 |
| JWT | - | Token 认证 |
| Swagger | - | API 文档 |

---

## Part 2: 目录结构

```
├── api/                        # API 定义
│   └── v1/
│       ├── user.go
│       └── auth.go
├── config/                     # 配置
│   ├── config.go
│   └── config.yaml
├── docs/                       # Swagger 文档
├── internal/                   # 内部模块
│   ├── middleware/             # 中间件
│   │   ├── jwt.go
│   │   ├── permission.go
│   │   └── cors.go
│   ├── model/                  # 模型
│   │   ├── entity/             # 实体
│   │   │   ├── base.go
│   │   │   └── user.go
│   │   ├── dto/                # DTO
│   │   │   ├── user_dto.go
│   │   │   └── page_dto.go
│   │   └── vo/                 # VO
│   │       └── user_vo.go
│   ├── repository/             # 数据访问层
│   │   └── user_repo.go
│   ├── service/                # 业务逻辑层
│   │   └── user_service.go
│   └── router/                 # 路由
│       └── router.go
├── pkg/                        # 公共包
│   ├── response/               # 响应
│   │   └── result.go
│   ├── auth/                   # 认证
│   │   └── jwt.go
│   ├── cache/                  # 缓存
│   │   └── redis.go
│   └── utils/                  # 工具
│       └── bcrypt.go
├── main.go                     # 入口
└── go.mod
```

---

## Part 3: 命名规范

### 文件命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 控制器 | 小写 + _ | `user.go` |
| 服务 | 小写 + _service | `user_service.go` |
| 仓库 | 小写 + _repo | `user_repo.go` |
| 实体 | 小写 | `user.go` |
| DTO | 小写 + _dto | `user_dto.go` |
| VO | 小写 + _vo | `user_vo.go` |

### 结构体命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 实体 | PascalCase | `User` 或 `SysUser` |
| DTO | 功能 + DTO | `UserForm`, `UserPageQuery` |
| VO | 功能 + VO | `UserVO` |
| Service | 实体 + Service | `UserService` |
| Repository | 实体 + Repo | `UserRepo` |

### 方法命名

| 动作 | 前缀 | 示例 |
|------|------|------|
| 查询单个 | Get | `GetByID()` |
| 查询列表 | List | `ListUsers()` |
| 分页查询 | Page | `PageUsers()` |
| 新增 | Create | `CreateUser()` |
| 更新 | Update | `UpdateUser()` |
| 删除 | Delete | `DeleteByID()` |

### 变量命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 变量 | camelCase | `userList` |
| 常量 | PascalCase 或 UPPER_SNAKE_CASE | `MaxSize` 或 `MAX_SIZE` |
| 私有变量 | 小写开头 | `internalState` |
| 布尔值 | Is/Has/Can 前缀 | `IsDeleted`, `HasPermission` |

---

## Part 4: RESTful API 规范

### 标准 CRUD 路径

| 操作 | 方法 | 路径 |
|------|------|------|
| 分页列表 | GET | `/api/v1/users/page` |
| 详情 | GET | `/api/v1/users/:id` |
| 新增 | POST | `/api/v1/users` |
| 更新 | PUT | `/api/v1/users` |
| 删除 | DELETE | `/api/v1/users/:id` |
| 批量删除 | DELETE | `/api/v1/users/batch` |
| 下拉选项 | GET | `/api/v1/users/options` |

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

## Part 10: 代码质量检查清单

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
