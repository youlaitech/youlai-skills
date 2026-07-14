---
name: aspnet
description: ASP.NET Core backend development standards. Use this skill when developing ASP.NET Core projects, implementing REST APIs, using Entity Framework Core, or JWT authentication.
---

# ASP.NET Core 后端开发规范

## 触发条件

- Develop ASP.NET Core projects
- Implement REST APIs
- Use Entity Framework Core for data access
- Implement JWT authentication
- Implement permission control

---

## Part 1: 技术栈

| 层 | 选型 | 说明 |
|----|------|------|
| 运行环境 | **.NET 8+** | 跨平台运行时 |
| Web 框架 | **ASP.NET Core** | 内置依赖注入、中间件管道 |
| ORM | **Entity Framework Core** | LINQ 查询、Code First、迁移 |
| 数据库 | **MySQL 8.x** | InnoDB 引擎 |
| 缓存 | **Redis 7.x** | 分布式缓存 |
| 认证 | **JWT** (Microsoft.AspNetCore.Authentication.JwtBearer) | Token 认证 |
| API 文档 | **Swashbuckle** | OpenAPI 生成 |

---

## Part 2: 目录结构

```
src/
├── Youlai.Api/                     # API 层（Controller + 中间件）
│   ├── Program.cs                  # 启动配置
│   ├── Controllers/                # 按模块分目录，复数命名
│   │   ├── Auth/{AuthController, WxMaAuthController}
│   │   ├── System/{UsersController, RolesController, MenusController, ...}
│   │   ├── Codegen/{CodegenController}
│   │   ├── File/{FilesController}
│   │   └── Message/{SseController}
│   ├── Filters/                    # LogActionFilter
│   ├── Middlewares/                # ExceptionHandlingMiddleware, RateLimitMiddleware
│   ├── Security/                   # CurrentUser, HasPermAttribute, JwtExtensions, PermissionAuthorizationHandler
│   └── Swagger/                    # FileUploadOperationFilter
│
├── Youlai.Application/            # 应用层（Service + DTO）
│   ├── DependencyInjection.cs
│   ├── Auth/                       # IAuthService, AuthService, WxMaAuthService + Models/
│   ├── Common/                     # ILoggingService, SseService
│   ├── Constants/                  # RedisKeyConstants
│   ├── Exceptions/                 # BusinessException
│   ├── Options/                    # SecurityOptions, RedisOptions, WxMaOptions
│   ├── Results/                    # Result, PageResult, ResultCode
│   ├── Security/                   # ICurrentUser, JwtTokenManager, DataPermissionService + Models/
│   └── System/                     # IUserService, UserService 等 + Models/{User,Role,Menu,Dept,...}/
│
├── Youlai.Domain/                  # 领域层（最薄，无依赖）
│   ├── Entities/                   # SysUser, SysRole, SysMenu 等
│   └── Enums/                      # ActionType, LogModule, MenuType
│
├── Youlai.Infrastructure/          # 基础设施层
│   ├── DependencyInjection.cs
│   ├── Persistence/AppDbContext.cs
│   ├── FileStorage/                # LocalFileStorage, MinioFileStorage
│   └── CodegenTemplates/           # Scriban 模板
│
└── tests/Youlai.Api.Tests/         # 集成测试
```

**设计原则**：
- `Youlai.Domain` 是最薄层，不依赖任何层
- `Youlai.Application` 依赖 `Domain`，不依赖 `Api` 和 `Infrastructure`
- `Youlai.Infrastructure` 实现接口，依赖 `Application`
- `Youlai.Api` 作为入口，组合所有层

---

## Part 3: 命名规范

### 3.1 文件命名

| 类型 | 规范 | 示例 |
|------|------|------|
| Controller | `{Entity}Controller.cs` | `UserController.cs` |
| Service 接口 | `I{Entity}Service.cs` | `IUserService.cs` |
| Service 实现 | `{Entity}Service.cs` | `UserService.cs` |
| Entity | PascalCase | `SysUser.cs` |
| DTO | 功能 + 类型后缀 | `UserForm.cs`, `UserVO.cs` |
| Enums | PascalCase + 无后缀 | `ActionType.cs` |

### 3.2 类命名

| 类型 | 规范 | 示例 |
|------|------|------|
| Controller | `{Entity}Controller` | `UserController` |
| Service 接口 | `I{Entity}Service` | `IUserService` |
| Service 实现 | `{Entity}Service` | `UserService` |
| Entity | PascalCase | `SysUser` |
| DTO | 功能 + 类型 | `UserVO`, `UserForm`, `UserPageQuery` |

**DTO 后缀规范**：

| 后缀 | 用途 | 示例 |
|------|------|------|
| `VO` | 视图对象（返回前端） | `UserVO`, `RolePageVO` |
| `Form` / `RequestDto` | 表单/请求对象 | `UserForm`, `LoginRequestDto` |
| `Query` | 查询参数 | `UserQuery`, `RolePageQuery` |
| `Dto` | 传输对象 | `CurrentUserDto`, `ExcelResultDto` |

### 3.3 方法命名

| 动作 | 前缀 | 示例 |
|------|------|------|
| 查询单个 | Get | `GetByIdAsync()` |
| 查询列表 | List | `ListUsersAsync()` |
| 分页查询 | Page | `PageUsersAsync()` |
| 新增 | Create | `CreateUserAsync()` |
| 更新 | Update | `UpdateUserAsync()` |
| 删除 | Delete | `DeleteUserAsync()` |
| 下拉选项 | GetXxxOptions | `GetUserOptionsAsync()` |

### 3.4 变量命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 局部变量 | camelCase | `userList` |
| 常量 | PascalCase | `MaxSize` |
| 私有字段 | `_` 前缀 camelCase | `_userService` |
| 布尔值 | Is/Has/Can 前缀 | `IsDeleted`, `HasPermission` |

> ⚠️ ASP.NET Core 约定私有字段用 `_` 前缀（与 Spring Boot 不同），与官方文档和 EF Core 一致。

---

## Part 4: RESTful API 规范

### 4.1 标准 CRUD 路径

| 操作 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 分页列表 | `GET` | `/api/v1/users/page` | Query 参数 |
| 详情 | `GET` | `/api/v1/users/{id}` | 路径参数 |
| 表单数据 | `GET` | `/api/v1/users/{id}/form` | 编辑回显 |
| 新增 | `POST` | `/api/v1/users` | Body: `UserForm` |
| 更新 | `PUT` | `/api/v1/users` | Body: `UserForm` |
| 删除 | `DELETE` | `/api/v1/users/{id}` | 路径参数 |
| 批量删除 | `DELETE` | `/api/v1/users/batch` | Body: `List<long>` |
| 下拉选项 | `GET` | `/api/v1/users/options` | 返回 `List<OptionType>` |

### 4.2 Controller 模板

```csharp
[ApiController]
[Route("api/v1/users")]
[ApiExplorerSettings(GroupName = "用户管理")]
public class UserController : ControllerBase
{
    private readonly IUserService _userService;

    public UserController(IUserService userService) =>
        _userService = userService;

    [HttpGet("page")]
    [SwaggerOperation(Summary = "用户分页列表")]
    public async Task<Result<PageResult<UserVO>>> Page([FromQuery] UserPageQuery query) =>
        Result.Success(await _userService.PageUsersAsync(query));

    [HttpPost]
    [SwaggerOperation(Summary = "新增用户")]
    [Authorize(Policy = "sys:user:add")]
    public async Task<Result<long>> Create([FromBody] UserForm form) =>
        Result.Success(await _userService.CreateUserAsync(form));

    [HttpPut]
    [SwaggerOperation(Summary = "更新用户")]
    [Authorize(Policy = "sys:user:edit")]
    public async Task<Result> Update([FromBody] UserForm form)
    {
        await _userService.UpdateUserAsync(form);
        return Result.Success();
    }

    [HttpDelete("{id}")]
    [SwaggerOperation(Summary = "删除用户")]
    [Authorize(Policy = "sys:user:delete")]
    public async Task<Result> Delete(long id)
    {
        await _userService.DeleteUserAsync(id);
        return Result.Success();
    }

    [HttpGet("options")]
    [SwaggerOperation(Summary = "用户选项列表")]
    public async Task<Result<List<OptionType>>> Options() =>
        Result.Success(await _userService.GetUserOptionsAsync());
}
```

---

## Part 5: 响应格式与异常处理

### 5.1 统一响应

```csharp
public class Result<T>
{
    public int Code { get; set; }
    public string Msg { get; set; } = string.Empty;
    public T? Data { get; set; }

    public static Result<T> Success(T? data = default) => new() { Code = 200, Msg = "操作成功", Data = data };
    public static Result<T> Failed(int code, string msg) => new() { Code = code, Msg = msg };
}

public class PageResult<T>
{
    public List<T> List { get; set; } = [];
    public long Total { get; set; }

    public static PageResult<T> Of(List<T> list, long total) => new() { List = list, Total = total };
}
```

### 5.2 业务异常

```csharp
public class BusinessException : Exception
{
    public BusinessException(string message) : base(message) { }
}
```

### 5.3 全局异常处理

```csharp
public class ExceptionMiddleware
{
    private readonly RequestDelegate _next;

    public ExceptionMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context)
    {
        try { await _next(context); }
        catch (BusinessException ex) { await WriteResult(context, 200, ex.Message); }
        catch (Exception ex) { await WriteResult(context, 500, "系统异常"); }
    }

    private static async Task WriteResult(HttpContext context, int code, string msg)
    {
        context.Response.StatusCode = 200;
        context.Response.ContentType = "application/json";
        await context.Response.WriteAsync(JsonSerializer.Serialize(new { Code = code, Msg = msg, Data = (object?)null }));
    }
}
```

---

## Part 6: 实体规范

```csharp
public abstract class BaseEntity
{
    [Key][Column("id")] public long Id { get; set; }
    [Column("create_time")] public DateTime CreateTime { get; set; } = DateTime.Now;
    [Column("update_time")] public DateTime UpdateTime { get; set; } = DateTime.Now;
    [Column("deleted")] public bool Deleted { get; set; } = false;
}

[Table("sys_user")]
public class SysUser : BaseEntity
{
    [Required][MaxLength(50)][Column("username")] public string Username { get; set; } = "";
    [Required][MaxLength(100)][Column("password")] public string Password { get; set; } = "";
    [Required][MaxLength(50)][Column("nickname")] public string Nickname { get; set; } = "";
    [MaxLength(20)][Column("mobile")] public string? Mobile { get; set; }
    [Column("status")] public int Status { get; set; } = 1;
    [Column("dept_id")] public long? DeptId { get; set; }
}
```

---

## Part 7: Service 规范

```csharp
public interface IUserService
{
    Task<PageResult<UserVO>> PageUsersAsync(UserPageQuery query);
    Task<UserVO> GetUserByIdAsync(long id);
    Task<UserForm> GetUserFormDataAsync(long id);
    Task<long> CreateUserAsync(UserForm form);
    Task UpdateUserAsync(UserForm form);
    Task DeleteUserAsync(long id);
}

public class UserService : IUserService
{
    private readonly AppDbContext _context;

    public UserService(AppDbContext context) => _context = context;

    public async Task<PageResult<UserVO>> PageUsersAsync(UserPageQuery query)
    {
        var q = _context.Users.Where(u => !u.Deleted);
        if (!string.IsNullOrEmpty(query.Keywords))
            q = q.Where(u => u.Username.Contains(query.Keywords));
        if (query.Status.HasValue)
            q = q.Where(u => u.Status == query.Status.Value);

        var total = await q.CountAsync();
        var list = await q.OrderByDescending(u => u.CreateTime)
            .Skip((query.PageNum - 1) * query.PageSize).Take(query.PageSize)
            .Select(u => new UserVO { Id = u.Id, Username = u.Username, Status = u.Status, CreateTime = u.CreateTime })
            .ToListAsync();
        return PageResult<UserVO>.Of(list, total);
    }

    public async Task<long> CreateUserAsync(UserForm form)
    {
        if (await _context.Users.AnyAsync(u => u.Username == form.Username && !u.Deleted))
            throw new BusinessException("用户名已存在");
        var user = new SysUser { Username = form.Username, Password = BCrypt.HashPassword(form.Password), Status = form.Status };
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
        return user.Id;
    }
}
```

---

## Part 8: 认证规范

### 8.1 JWT 配置

```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => {
        options.TokenValidationParameters = new TokenValidationParameters {
            ValidateIssuer = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Secret"]!))
        };
    });

builder.Services.AddAuthorization(options => {
    options.AddPolicy("sys:user:add", p => p.RequireClaim("permission", "sys:user:add"));
    options.AddPolicy("sys:user:edit", p => p.RequireClaim("permission", "sys:user:edit"));
});
```

### 8.2 权限校验

```csharp
// Controller 中使用 Policy 进行权限校验
[Authorize(Policy = "sys:user:add")]

// 权限标识命名：模块:实体:操作
// sys:user:create / sys:user:edit / sys:user:delete / sys:role:list / sys:menu:assign
```

### 8.3 当前用户

```csharp
public interface ICurrentUser
{
    long UserId { get; }
    string Username { get; }
    List<string> Permissions { get; }
}

public class CurrentUser : ICurrentUser
{
    public long UserId => long.Parse(HttpContext.User.FindFirst(JwtRegisteredClaimNames.Sub)?.Value ?? "0");
    public string Username => HttpContext.User.FindFirst(JwtRegisteredClaimNames.UniqueName)?.Value ?? "";
    public List<string> Permissions => HttpContext.User.FindFirst("permission")?.Value?.Split(',').ToList() ?? [];
}
```

---

## Part 9: 注释规范

- **公共方法用 `/// <summary>` XML 注释**（IDE 智能提示可识别）
- Swagger 使用 `[SwaggerOperation(Summary = "...")]`
- 代码修改时注释必须同步更新

```csharp
/// <summary>
/// 用户管理控制器。
/// </summary>
[ApiExplorerSettings(GroupName = "用户管理")]
public class UserController : ControllerBase
{
    /// <summary>
    /// 创建新用户。
    /// </summary>
    /// <param name="form">用户表单（username 必填且唯一）</param>
    /// <returns>新用户 ID</returns>
    /// <exception cref="BusinessException">用户名已存在</exception>
    [HttpPost]
    [SwaggerOperation(Summary = "新增用户")]
    public async Task<Result<long>> Create([FromBody] UserForm form) { ... }
}
```

---

## Part 10: 代码质量检查清单

- [ ] 遵循 RESTful API 路径规范
- [ ] 使用统一响应格式
- [ ] 实现全局异常处理中间件
- [ ] 权限校验使用 `Authorize(Policy = "...")`
- [ ] 分页接口使用 `PageResult`
- [ ] 实体继承 `BaseEntity`
- [ ] 使用 EF Core 逻辑删除
- [ ] API 添加 `[SwaggerOperation]` 注解
- [ ] 分层：Controller → Service → Repository
- [ ] 异步方法以 `Async` 结尾
- [ ] 公共方法有 `/// <summary>` XML 注释

### 常见反模式

| 反模式 | 正确做法 |
|--------|----------|
| `throw new Exception("用户不存在")` | `throw new BusinessException("用户不存在")` |
| `catch { }` 空捕获 | 捕获并记录日志或抛出业务异常 |
| `public async Task<T> Xxx()` 无 `Async` 后缀 | 加 `Async` 后缀：`XxxAsync()` |
| Controller 中写业务逻辑 | 委托 Service 层处理 |
| 同步方法调 `SaveChangesAsync().Result` | 使用 `await SaveChangesAsync()` |
