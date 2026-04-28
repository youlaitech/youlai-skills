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

- **.NET** - 运行环境
- **ASP.NET Core** - Web 框架
- **EF Core** - ORM 框架
- **MySQL** - 数据库
- **Redis** - 缓存
- **JWT** - Token 认证
- **Swashbuckle** - API 文档

---

## Part 2: 目录结构

```
src/
├── Youlai.Api/                     # API 层
│   ├── Program.cs                  # 启动配置
│   ├── appsettings.json
│   ├── Controllers/                # 控制器（按模块分目录，复数命名）
│   │   ├── Auth/
│   │   │   ├── AuthController.cs
│   │   │   └── WxMaAuthController.cs
│   │   ├── System/
│   │   │   ├── UsersController.cs
│   │   │   ├── RolesController.cs
│   │   │   ├── MenusController.cs
│   │   │   ├── DeptsController.cs
│   │   │   ├── DictsController.cs
│   │   │   ├── ConfigsController.cs
│   │   │   ├── NoticesController.cs
│   │   │   └── LogsController.cs
│   │   ├── Codegen/
│   │   │   └── CodegenController.cs
│   │   ├── File/
│   │   │   └── FilesController.cs
│   │   └── Message/
│   │       └── SseController.cs
│   ├── Converters/                 # JSON 转换器
│   │   └── Int64ToStringJsonConverter.cs
│   ├── Filters/                    # 过滤器
│   │   └── LogActionFilter.cs
│   ├── Middlewares/                # 中间件（复数 s）
│   │   ├── ExceptionHandlingMiddleware.cs
│   │   └── RateLimitMiddleware.cs
│   ├── Security/                   # HTTP 授权
│   │   ├── CurrentUser.cs
│   │   ├── HasPermAttribute.cs
│   │   ├── JwtExtensions.cs
│   │   ├── PermissionAuthorizationHandler.cs
│   │   ├── PermissionPolicyProvider.cs
│   │   └── PermissionRequirement.cs
│   └── Swagger/
│       └── FileUploadOperationFilter.cs
│
├── Youlai.Application/            # 应用层
│   ├── DependencyInjection.cs      # DI 注册
│   ├── Attributes/
│   │   └── LogAttribute.cs
│   ├── Auth/                       # 认证业务
│   │   ├── IAuthService.cs
│   │   ├── AuthService.cs
│   │   ├── ICaptchaService.cs
│   │   ├── CaptchaService.cs
│   │   ├── IWxMaAuthService.cs
│   │   ├── WxMaAuthService.cs
│   │   ├── JwtClaimConstants.cs
│   │   └── Models/
│   │       ├── LoginRequestDto.cs
│   │       ├── AuthenticationTokenDto.cs
│   │       ├── CaptchaInfoDto.cs
│   │       └── WxMaLoginResultDto.cs
│   ├── Codegen/                    # 代码生成业务
│   │   ├── ICodegenService.cs
│   │   ├── CodegenService.cs
│   │   └── Models/
│   │       ├── CodegenTableDto.cs
│   │       ├── CodegenPreviewDto.cs
│   │       ├── CodegenTableQuery.cs
│   │       ├── FieldConfigDto.cs
│   │       └── GenConfigFormDto.cs
│   ├── Common/                     # 通用服务
│   │   ├── ILoggingService.cs
│   │   ├── LoggingService.cs
│   │   ├── ISseService.cs
│   │   ├── SseService.cs
│   │   ├── SseMessage.cs
│   │   └── UserAgentParser.cs
│   ├── Constants/
│   │   ├── DevDefaults.cs
│   │   └── RedisKeyConstants.cs
│   ├── Exceptions/
│   │   └── BusinessException.cs
│   ├── Extensions/
│   │   ├── CollectionExtensions.cs
│   │   ├── CurrentUserExtensions.cs
│   │   └── QueryableExtensions.cs
│   ├── File/                       # 文件业务
│   │   ├── IFileService.cs
│   │   ├── FileService.cs
│   │   ├── IFileStorage.cs
│   │   └── FileInfoDto.cs
│   ├── Models/                     # 通用基础模型
│   │   ├── BaseQuery.cs
│   │   ├── ExcelResult.cs
│   │   ├── KeyValue.cs
│   │   └── Option.cs
│   ├── Options/                    # 配置选项
│   │   ├── SecurityOptions.cs
│   │   ├── DatabaseOptions.cs
│   │   ├── RedisOptions.cs
│   │   ├── CaptchaOptions.cs
│   │   ├── OssOptions.cs
│   │   └── WxMaOptions.cs
│   ├── Persistence/
│   │   └── IDbContext.cs          # 数据上下文接口
│   ├── Results/                    # 统一响应
│   │   ├── IResultCode.cs
│   │   ├── ResultCode.cs
│   │   ├── Result.cs
│   │   └── PageResult.cs
│   ├── Security/                   # 业务权限
│   │   ├── ICurrentUser.cs
│   │   ├── JwtTokenManager.cs
│   │   ├── DataPermissionService.cs
│   │   ├── IRolePermissionService.cs
│   │   ├── RolePermissionService.cs
│   │   ├── IRolePermsCacheInvalidator.cs
│   │   ├── RolePermsCacheInvalidator.cs
│   │   ├── SecurityConstants.cs
│   │   └── Models/
│   │       ├── DataScope.cs
│   │       ├── RoleDataScope.cs
│   │       └── UserSession.cs
│   └── System/                     # 系统管理业务
│       ├── ISystemUserService.cs
│       ├── SystemUserService.cs
│       ├── ISystemRoleService.cs
│       ├── SystemRoleService.cs
│       ├── ISystemMenuService.cs
│       ├── SystemMenuService.cs
│       ├── ISystemDeptService.cs
│       ├── SystemDeptService.cs
│       ├── ISystemDictService.cs
│       ├── SystemDictService.cs
│       ├── ISystemConfigService.cs
│       ├── SystemConfigService.cs
│       ├── ISystemNoticeService.cs
│       ├── SystemNoticeService.cs
│       ├── ISystemLogService.cs
│       ├── SystemLogService.cs
│       └── Models/                 # 按实体分子目录
│           ├── User/               # UserForm, UserPageVo, UserQuery, CurrentUserDto 等
│           ├── Role/               # RoleForm, RolePageVo, RoleQuery
│           ├── Menu/               # MenuForm, MenuVo, MenuQuery, RouteVo
│           ├── Dept/               # DeptForm, DeptVo, DeptQuery
│           ├── Dict/               # DictForm, DictPageVo, DictQuery + DictItem 系列
│           ├── Config/             # ConfigForm, ConfigPageVo, ConfigQuery
│           ├── Notice/             # NoticeForm, NoticePageVo, NoticeQuery, NoticeDetailVo
│           ├── Log/                # LogPageVo, LogQuery
│           └── Statistics/         # VisitStatsVo, VisitTrendVo, VisitTrendQuery
│
├── Youlai.Domain/                  # 领域层（最薄，无依赖）
│   ├── Entities/                   # 实体（SysUser, SysRole, SysMenu 等）
│   └── Enums/                      # 枚举（ActionType, LogModule, MenuType）
│
├── Youlai.Infrastructure/          # 基础设施层
│   ├── DependencyInjection.cs
│   ├── Persistence/
│   │   └── AppDbContext.cs         # EF Core 上下文
│   ├── FileStorage/                # 文件存储实现
│   │   ├── LocalFileStorage.cs
│   │   ├── AliyunFileStorage.cs
│   │   └── MinioFileStorage.cs
│   └── CodegenTemplates/           # Scriban 代码生成模板
│       ├── backend/                # controller.cs.sbn, entity.cs.sbn, form.cs.sbn 等
│       └── frontend/               # js/ + ts/ 模板
│
└── tests/
    └── Youlai.Api.Tests/           # 集成测试
```

---

## Part 3: 命名规范

### 文件命名

| 类型   | 规范            | 示例                       |
| ------ | --------------- | -------------------------- |
| 控制器 | PascalCase      | `UserController.cs`        |
| 服务   | I + PascalCase  | `IUserService.cs`          |
| 实体   | PascalCase      | `SysUser.cs`               |
| DTO    | 功能 + DTO 类型 | `UserVO.cs`, `UserForm.cs` |
| 接口   | I + PascalCase  | `IUserRepository.cs`       |

### 类命名

| 类型     | 规范               | 示例                 |
| -------- | ------------------ | -------------------- |
| 控制器   | 实体 + Controller  | `UserController`     |
| 服务接口 | I + 实体 + Service | `IUserService`       |
| 服务实现 | 实体 + Service     | `UserService`        |
| 实体     | PascalCase         | `SysUser`            |
| DTO      | 功能 + 类型        | `UserVO`, `UserForm` |

### 方法命名

| 动作     | 前缀   | 示例                |
| -------- | ------ | ------------------- |
| 查询单个 | Get    | `GetByIdAsync()`    |
| 查询列表 | List   | `ListUsersAsync()`  |
| 分页查询 | Page   | `PageUsersAsync()`  |
| 新增     | Create | `CreateUserAsync()` |
| 更新     | Update | `UpdateUserAsync()` |
| 删除     | Delete | `DeleteUserAsync()` |

### 变量命名

| 类型     | 规范                           | 示例                         |
| -------- | ------------------------------ | ---------------------------- |
| 局部变量 | camelCase                      | `userList`                   |
| 常量     | PascalCase 或 UPPER_SNAKE_CASE | `MaxSize` 或 `MAX_SIZE`      |
| 私有字段 | \_ 前缀                        | `_userService`               |
| 布尔值   | Is/Has/Can 前缀                | `IsDeleted`, `HasPermission` |

---

## Part 4: RESTful API 规范

### 标准 CRUD 路径

| 操作     | 方法   | 路径                    |
| -------- | ------ | ----------------------- |
| 分页列表 | GET    | `/api/v1/users/page`    |
| 详情     | GET    | `/api/v1/users/{id}`    |
| 新增     | POST   | `/api/v1/users`         |
| 更新     | PUT    | `/api/v1/users`         |
| 删除     | DELETE | `/api/v1/users/{id}`    |
| 批量删除 | DELETE | `/api/v1/users/batch`   |
| 下拉选项 | GET    | `/api/v1/users/options` |

### Controller 模板

```csharp
// Controllers/System/UserController.cs
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Swashbuckle.AspNetCore.Annotations;

namespace Youlai.Api.Controllers.System;

[ApiController]
[Route("api/v1/users")]
[ApiExplorerSettings(GroupName = "用户管理")]
public class UserController : ControllerBase
{
    private readonly IUserService _userService;

    public UserController(IUserService userService)
    {
        _userService = userService;
    }

    [HttpGet("page")]
    [SwaggerOperation(Summary = "用户分页列表")]
    public async Task<Result<PageResult<UserVO>>> Page([FromQuery] UserPageQuery query)
    {
        return Result.Success(await _userService.PageUsersAsync(query));
    }

    [HttpGet("{id}")]
    [SwaggerOperation(Summary = "用户详情")]
    public async Task<Result<UserVO>> GetById(long id)
    {
        return Result.Success(await _userService.GetUserByIdAsync(id));
    }

    [HttpGet("{id}/form")]
    [SwaggerOperation(Summary = "用户表单数据")]
    public async Task<Result<UserForm>> GetFormData(long id)
    {
        return Result.Success(await _userService.GetUserFormDataAsync(id));
    }

    [HttpPost]
    [SwaggerOperation(Summary = "新增用户")]
    [Authorize(Policy = "sys:user:add")]
    public async Task<Result<long>> Create([FromBody] UserForm form)
    {
        return Result.Success(await _userService.CreateUserAsync(form));
    }

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

    [HttpDelete("batch")]
    [SwaggerOperation(Summary = "批量删除用户")]
    [Authorize(Policy = "sys:user:delete")]
    public async Task<Result> BatchDelete([FromBody] List<long> ids)
    {
        await _userService.DeleteUsersAsync(ids);
        return Result.Success();
    }

    [HttpGet("options")]
    [SwaggerOperation(Summary = "用户选项列表")]
    public async Task<Result<List<OptionType>>> Options()
    {
        return Result.Success(await _userService.GetUserOptionsAsync());
    }
}
```

---

## Part 5: 响应格式

```csharp
// Youlai.Common/Models/Result.cs
namespace Youlai.Common.Models;

public class Result<T>
{
    public int Code { get; set; }
    public string Msg { get; set; } = string.Empty;
    public T? Data { get; set; }

    public static Result<T> Success(T? data = default)
    {
        return new Result<T>
        {
            Code = 200,
            Msg = "操作成功",
            Data = data
        };
    }

    public static Result<T> Failed(int code, string msg)
    {
        return new Result<T>
        {
            Code = code,
            Msg = msg,
            Data = default
        };
    }
}

// Youlai.Common/Models/PageResult.cs
public class PageResult<T>
{
    public List<T> List { get; set; } = [];
    public long Total { get; set; }

    public static PageResult<T> Of(List<T> list, long total)
    {
        return new PageResult<T>
        {
            List = list,
            Total = total
        };
    }
}
```

---

## Part 6: 实体规范

### 基础实体

```csharp
// Youlai.Domain/Entities/BaseEntity.cs
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace Youlai.Domain.Entities;

public abstract class BaseEntity
{
    [Key]
    [Column("id")]
    public long Id { get; set; }

    [Column("create_time")]
    public DateTime CreateTime { get; set; } = DateTime.Now;

    [Column("update_time")]
    public DateTime UpdateTime { get; set; } = DateTime.Now;

    [Column("create_by")]
    public long? CreateBy { get; set; }

    [Column("update_by")]
    public long? UpdateBy { get; set; }

    [Column("deleted")]
    public bool Deleted { get; set; } = false;
}
```

### 实体示例

```csharp
// Youlai.Domain/Entities/SysUser.cs
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace Youlai.Domain.Entities;

[Table("sys_user")]
public class SysUser : BaseEntity
{
    [Required]
    [MaxLength(50)]
    [Column("username")]
    public string Username { get; set; } = string.Empty;

    [Required]
    [MaxLength(100)]
    [Column("password")]
    public string Password { get; set; } = string.Empty;

    [Required]
    [MaxLength(50)]
    [Column("nickname")]
    public string Nickname { get; set; } = string.Empty;

    [MaxLength(20)]
    [Column("mobile")]
    public string? Mobile { get; set; }

    [MaxLength(100)]
    [Column("email")]
    public string? Email { get; set; }

    [Column("status")]
    public int Status { get; set; } = 1;

    [Column("dept_id")]
    public long? DeptId { get; set; }
}
```

---

## Part 7: Service 规范

```csharp
// Youlai.Application/Services/IUserService.cs
namespace Youlai.Application.Services;

public interface IUserService
{
    Task<PageResult<UserVO>> PageUsersAsync(UserPageQuery query);
    Task<UserVO> GetUserByIdAsync(long id);
    Task<UserForm> GetUserFormDataAsync(long id);
    Task<long> CreateUserAsync(UserForm form);
    Task UpdateUserAsync(UserForm form);
    Task DeleteUserAsync(long id);
    Task DeleteUsersAsync(List<long> ids);
    Task<List<OptionType>> GetUserOptionsAsync();
}

// Youlai.Application/Services/UserService.cs
using Microsoft.EntityFrameworkCore;

namespace Youlai.Application.Services;

public class UserService : IUserService
{
    private readonly AppDbContext _context;

    public UserService(AppDbContext context)
    {
        _context = context;
    }

    public async Task<PageResult<UserVO>> PageUsersAsync(UserPageQuery query)
    {
        var queryable = _context.Users
            .Where(u => !u.Deleted);

        if (!string.IsNullOrEmpty(query.Keywords))
        {
            queryable = queryable.Where(u =>
                u.Username.Contains(query.Keywords) ||
                u.Nickname.Contains(query.Keywords));
        }

        if (query.Status.HasValue)
        {
            queryable = queryable.Where(u => u.Status == query.Status.Value);
        }

        var total = await queryable.CountAsync();

        var list = await queryable
            .OrderByDescending(u => u.CreateTime)
            .Skip((query.PageNum - 1) * query.PageSize)
            .Take(query.PageSize)
            .Select(u => new UserVO
            {
                Id = u.Id,
                Username = u.Username,
                Nickname = u.Nickname,
                Mobile = u.Mobile,
                Email = u.Email,
                Status = u.Status,
                CreateTime = u.CreateTime
            })
            .ToListAsync();

        return PageResult<UserVO>.Of(list, total);
    }

    public async Task<long> CreateUserAsync(UserForm form)
    {
        var exists = await _context.Users
            .AnyAsync(u => u.Username == form.Username && !u.Deleted);

        if (exists)
        {
            throw new BusinessException("用户名已存在");
        }

        var user = new SysUser
        {
            Username = form.Username,
            Password = BCrypt.HashPassword(form.Password),
            Nickname = form.Nickname,
            Mobile = form.Mobile,
            Email = form.Email,
            Status = form.Status
        };

        _context.Users.Add(user);
        await _context.SaveChangesAsync();

        return user.Id;
    }
}
```

---

## Part 8: 认证规范

### JWT 配置

```csharp
// Program.cs
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = false,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Secret"]!))
        };
    });

builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("sys:user:add", policy => policy.RequireClaim("permission", "sys:user:add"));
    options.AddPolicy("sys:user:edit", policy => policy.RequireClaim("permission", "sys:user:edit"));
    options.AddPolicy("sys:user:delete", policy => policy.RequireClaim("permission", "sys:user:delete"));
});
```

### JWT 服务

```csharp
// Youlai.Infrastructure/Security/JwtService.cs
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;
using Microsoft.IdentityModel.Tokens;

namespace Youlai.Infrastructure.Security;

public class JwtService
{
    private readonly IConfiguration _config;

    public JwtService(IConfiguration config)
    {
        _config = config;
    }

    public string GenerateAccessToken(long userId, string username, List<string> permissions)
    {
        var claims = new List<Claim>
        {
            new(JwtRegisteredClaimNames.Sub, userId.ToString()),
            new(JwtRegisteredClaimNames.UniqueName, username),
            new("permission", string.Join(",", permissions))
        };

        var key = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(_config["Jwt:Secret"]!));
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: _config["Jwt:Issuer"],
            claims: claims,
            expires: DateTime.Now.AddHours(2),
            signingCredentials: creds);

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

### 当前用户

```csharp
// Youlai.Infrastructure/Security/CurrentUser.cs
using System.Security.Claims;

namespace Youlai.Infrastructure.Security;

public interface ICurrentUser
{
    long UserId { get; }
    string Username { get; }
    List<string> Permissions { get; }
}

public class CurrentUser : ICurrentUser
{
    private readonly IHttpContextAccessor _httpContextAccessor;

    public CurrentUser(IHttpContextAccessor httpContextAccessor)
    {
        _httpContextAccessor = httpContextAccessor;
    }

    public long UserId =>
        long.Parse(_httpContextAccessor.HttpContext?.User?.FindFirst(JwtRegisteredClaimNames.Sub)?.Value ?? "0");

    public string Username =>
        _httpContextAccessor.HttpContext?.User?.FindFirst(JwtRegisteredClaimNames.UniqueName)?.Value ?? "";

    public List<string> Permissions =>
        _httpContextAccessor.HttpContext?.User?.FindFirst("permission")?.Value?.Split(',').ToList() ?? [];
}
```

---

## Part 9: 异常处理

```csharp
// Youlai.Api/Middleware/ExceptionMiddleware.cs
using System.Net;
using System.Text.Json;

namespace Youlai.Api.Middleware;

public class ExceptionMiddleware
{
    private readonly RequestDelegate _next;

    public ExceptionMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (BusinessException ex)
        {
            await HandleExceptionAsync(context, (int)HttpStatusCode.OK, ex.Message);
        }
        catch (Exception ex)
        {
            await HandleExceptionAsync(context, (int)HttpStatusCode.InternalServerError, "系统异常");
        }
    }

    private static async Task HandleExceptionAsync(HttpContext context, int code, string msg)
    {
        context.Response.ContentType = "application/json";
        context.Response.StatusCode = 200;

        var result = new
        {
            Code = code,
            Msg = msg,
            Data = (object?)null
        };

        await context.Response.WriteAsync(JsonSerializer.Serialize(result));
    }
}

// Youlai.Application/Exceptions/BusinessException.cs
public class BusinessException : Exception
{
    public BusinessException(string message) : base(message) { }
}
```

---

## Part 10: 代码质量检查清单

- [ ] 遵循 RESTful API 路径规范
- [ ] 使用统一响应格式
- [ ] 实现全局异常处理
- [ ] 添加权限策略
- [ ] 分页接口使用 PageResult
- [ ] 参数校验使用数据注解
- [ ] 重要操作记录日志
- [ ] 使用事务处理
- [ ] 实体继承 BaseEntity
- [ ] 使用 EF Core 软删除
- [ ] API 添加 SwaggerOperation 注解
- [ ] 分层：Controller → Service → Repository
- [ ] 使用依赖注入
