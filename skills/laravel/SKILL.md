---
name: laravel
description: Laravel 后端开发规范. Use this skill when developing Laravel web projects, implementing REST APIs with Eloquent ORM, JWT/Redis token authentication, RBAC permissions, or the youlai-laravel admin backend (MySQL).
---

# Laravel 后端开发规范（Laravel + Eloquent）

> 本 Skill 配合 `youlai-laravel` 仓库使用。AI 助手依据本规范可从 0 到 1 创建与 youlai-boot 功能等价的 Laravel 后端。

## 触发条件

- 开发 Laravel Web 项目
- 使用 Eloquent ORM 实现数据访问
- 使用 `#[Fillable]` / `#[Hidden]` 等 PHP 8 属性注解
- 实现 JWT 认证与基于角色的权限控制（RBAC）
- 编写 REST API 控制器、Service、Validate
- 开发 youlai-laravel 管理后台模块
- 依据 `youlai-laravel` 仓库创建新模块或修复问题

---

## Part 1: 技术栈

| 层 | 选型 | 说明 |
| --- | --- | --- |
| 运行环境 | **PHP 8.3+** | PHP 8 属性注解、枚举、只读属性 |
| Web 框架 | **Laravel 13** | 路由、容器、中间件、队列 |
| ORM | **Eloquent** | ActiveRecord 风格，对标 MyBatis-Plus |
| 认证 | **tymon/jwt-auth** | JWT 自包含 Token，guard 名 `admin` |
| 缓存 | **Redis 7.x** | `cache()` / `Cache::` + Redis 驱动 |
| 数据库 | **MySQL 8.0+** | 复用 `youlai-laravel/database/youlai_admin.sql` |
| 验证码 | **gregwar/captcha** | 图形验证码 base64 输出 |
| 密码哈希 | **password_hash** | `PASSWORD_DEFAULT`（bcrypt） |
| 接口文档 | **Swagger / OpenAPI** | `/api/docs` 或 Knife4j 风格 |
| 代码生成 | **模板 + stub** | `templates/` 下 `.stub`/`.tpl` 生成前后端 CRUD |
| 实时通信 | **SSE** | `php artisan sse:start` 独立常驻进程 |
| 测试 | **PHPUnit** | `phpunit.xml` + `tests/` |
| 包管理 | **Composer** | `composer.json` |
| 前端构建 | **Vite** | `vite.config.js` |
| 代码质量 | **Pint** | Laravel 官方代码风格 |

> **含多租户**：youlai-laravel 实现多租户（`tenant_id` 字段 + `sys_tenant` 表 + 切换租户接口），新增模块需在查询中携带 `tenant_id` 隔离条件。

---

## Part 2: 目录结构

```
youlai-laravel/
├── app/
│   ├── Attributes/                # 自定义属性类（如 LogAction）
│   ├── Common/                    # 通用工具/枚举/常量
│   │   ├── Enums/                 # ResultCode 业务码枚举
│   │   ├── Constants/             # RedisKey 等常量
│   │   └── Interfaces/            # IResultCode 接口
│   ├── Console/                   # 命令行命令（sse:start 等）
│   ├── Exceptions/                # BusinessException 业务异常
│   ├── Extensions/                # 框架扩展
│   ├── Http/
│   │   ├── Controllers/           # 控制器
│   │   │   ├── Controller.php     # 抽象基类（ApiResponse + validateScene）
│   │   │   └── V1/                # API v1 版本控制器
│   │   │       ├── AuthController.php
│   │   │       └── SysUserController.php
│   │   ├── Middleware/            # jwt.admin / permission / 限流等中间件
│   │   └── Validates/             # 表单验证类（按模块划分）
│   ├── Models/                    # Eloquent 模型（继承 Base）
│   │   ├── Base.php               # 基础模型（状态/删除常量、时间字段）
│   │   └── SysUser.php
│   ├── Services/                  # 业务逻辑层（继承 BaseService）
│   │   ├── BaseService.php        # 基础服务（LoggableService）
│   │   └── SysUserService.php
│   ├── Pagination/                # 分页相关
│   ├── Providers/                 # 服务提供者
│   ├── Requests/                  # 请求验证类
│   ├── Scope/                     # 查询作用域
│   ├── Traits/                    # 可复用 Trait（ApiResponse 等）
│   └── Utils/                     # 工具类（VerifyCode / LogContext）
├── bootstrap/                     # 框架启动（app.php 注册中间件别名）
├── config/                        # 配置文件
├── database/                      # 迁移 / 工厂 / 填充 / SQL
│   └── youlai_admin.sql           # 系统数据库脚本
├── public/                        # 入口目录（index.php）
├── resources/                     # 视图 / 语言包 / 前端资源
├── routes/                        # 路由定义（api.php / web.php / console.php）
├── storage/                       # 运行时存储
├── templates/                     # 代码生成模板（.stub / .tpl）
├── tests/                         # PHPUnit 测试
├── artisan                        # Laravel 命令行入口
├── composer.json
└── .env                           # 环境配置
```

**分层与依赖关系**：

- `Models/` 是纯 Eloquent 模型，依赖 `Base`
- `Services/` 是业务层，依赖 `Models/` 与 `Validates/`
- `Http/Controllers/V1/` 是接口层，依赖 `Services/` 与 `Validates/`，通过 `validateScene()` 复用验证器
- `Common/` 是常量/枚举/接口，被所有层共享
- `Traits/ApiResponse.php` 提供统一响应，被 `Controller` 与 `BaseService` 共用

---

## Part 3: 命名规范

### 3.1 文件与目录命名

| 类型 | 规范 | 示例 |
| --- | --- | --- |
| 控制器目录 | `V1`（大写的 API 版本） | `app/Http/Controllers/V1/` |
| 控制器类 | 表名转 PascalCase + `Controller` | `SysUserController`、`SysRoleController` |
| Service 类 | 表名转 PascalCase + `Service` | `SysUserService`、`SysMenuService` |
| Validate 类 | 表名转 PascalCase + `Validate` | `SysUserValidate`、`SysRoleValidate` |
| Model 类 | 表名转 PascalCase（去前缀） | `SysUser`、`SysMenu`、`SysDept` |
| 模型目录 | 单数类名复数表名 | `sys_user` → `SysUser` |
| 配置文件 | 小写下划线 | `config/jwt.php`、`config/captcha.php` |
| 常量/枚举目录 | 见命名空间 | `app/Common/Enums/ResultCode.php` |

### 3.2 类型与成员命名

| 类型 | 规范 | 示例 |
| --- | --- | --- |
| Model 类 | 表名去 `sys_` 前缀转 PascalCase | `SysUser`、`SysDictItem` |
| Service 类 | 模块名 + `Service` | `SysUserService` |
| Validate 类 | 模块名 + `Validate` | `SysUserValidate` |
| 控制器方法 | RESTful 标准动作 | `index`/`show`/`store`/`update`/`destroy` + 业务动作 |
| 业务方法 | 动词开头 snake_case | `paginate`/`create`/`assignRoles`/`restPassword` |
| 数据库字段 | snake_case | `dept_id`、`create_time`、`is_deleted` |
| JSON/API 字段 | camelCase（响应自动转换） | `deptId`、`createTime`、`roleIds` |
| 路由参数 | snake_case 或 kebab | `{id}`、`{name}` |

### 3.3 方法命名（Service 层）

| 操作 | 方法名 | 示例 |
| --- | --- | --- |
| 分页列表 | `paginate` | `paginate(array $filters, int $perPage)` |
| 单条详情 | `show` | `show(int $id): ?SysUser` |
| 创建 | `create` | `create(array $data): SysUser` |
| 更新 | `update` | `update(SysUser $model, array $data)` |
| 删除（批量） | `delete` | `delete(array $ids): int` |
| 状态变更 | `updateStatus` | 业务语义命名 |
| 重置密码 | `restPassword` | `restPassword(SysUser $user, array $data)` |
| 分配角色 | `assignRoles` | `assignRoles(SysUser $user, array $roleIds)` |
| 下拉选项 | `getOptions` | `getOptions(): array` |
| 用户信息 | `getUserInfo` | `getUserInfo(): array` |
| 数据格式化 | `format` | `format(SysUser $user): array` |

### 3.4 变量命名

| 类型 | 规范 | 示例 |
| --- | --- | --- |
| 变量 | camelCase | `$userList`、`$roleCodes` |
| 常量 | SNAKE_CASE（类常量） | `PER_PAGE`、`STATUS_ENABLE` |
| 布尔变量 | `is_`/`has_` 前缀（PHP 属性） | `$isDeleted`、`$hasPerm` |
| 数据库字段 | snake_case | `dept_id`、`create_time` |
| JSON 键 | camelCase（响应层自动转换） | `deptId`、`createTime`、`pageSize` |

> **关键约定**：Model / Service 内部字段用 snake_case（对应 MySQL 列名）；响应输出经 `ApiResponse` 的 `CaseConversion` 自动转为 camelCase，前端收到的 JSON 键名为驼峰。无需手动 `toCamelCase`。

---

## Part 4: 路由与 API 规范

### 4.1 路由文件与版本前缀

所有 API 定义在 `routes/api.php`，统一加 `v1` 前缀、命名空间 `App\Http\Controllers\V1`：

```php
// routes/api.php
Route::prefix('v1')->namespace('App\Http\Controllers\V1')->group(function () {
    // 公开
    Route::prefix('auth')->group(function () {
        Route::post('login', [AuthController::class, 'login']);
        Route::get('captcha', [AuthController::class, 'captcha']);
    });

    // 需要 JWT 认证
    Route::middleware(['jwt.admin'])->group(function () {
        Route::prefix('users')->group(function () {
            Route::get('me', [SysUserController::class, 'me']);
            Route::put('{id}/password/reset', [SysUserController::class, 'restPassword'])
                ->where(['id' => '[0-9]+']);
            Route::get('options', [SysUserController::class, 'options']);
            Route::get('{id}/form', [SysUserController::class, 'show'])
                ->where(['id' => '[0-9]+']);
            apiIdResource(SysUserController::class);
        });
    });
});
```

| 模块 | 前缀 | 说明 |
| --- | --- | --- |
| 认证 | `/api/v1/auth` | login / register / captcha / refresh-token |
| 用户 | `/api/v1/users` | 用户管理 |
| 角色 | `/api/v1/roles` | 角色管理 |
| 菜单 | `/api/v1/menus` | 菜单管理 |
| 部门 | `/api/v1/depts` | 部门管理 |
| 字典 | `/api/v1/dicts`、`/api/v1/dicts/{name}/items` | 字典与字典项 |
| 配置 | `/api/v1/configs` | 系统配置 |
| 通知 | `/api/v1/notices` | 通知公告 |
| 日志 | `/api/v1/logs` | 操作日志 |
| 文件 | `/api/v1/files` | 文件上传/删除 |
| 代码生成 | `/api/v1/codegen` | 代码生成 |
| 租户 | `/api/v1/tenants`、`/api/v1/tenant-plans` | 多租户 |

### 4.2 标准 CRUD 路径

| 操作 | 方法 | 路径 | 说明 |
| --- | --- | --- | --- |
| 分页列表 | `GET` | `/api/v1/users` | Query：`pageNum`/`pageSize`/`keywords`/... |
| 表单数据 | `GET` | `/api/v1/users/{id}/form` | 路径参数 `id` |
| 新增 | `POST` | `/api/v1/users` | Body：snake_case 字段 |
| 更新 | `PUT` | `/api/v1/users/{id}` | Body：snake_case 字段 |
| 批量删除 | `DELETE` | `/api/v1/users/{ids}` | `ids` 逗号分隔 |
| 重置密码 | `PUT` | `/api/v1/users/{id}/password/reset` | |
| 分配角色 | `POST` / `PUT` | `/api/v1/users/{id}/roles` | |
| 下拉选项 | `GET` | `/api/v1/roles/options` | 返回 `[{value, label}]` |
| 当前用户 | `GET` | `/api/v1/users/me` | 当前登录用户信息 |
| 菜单路由 | `GET` | `/api/v1/menus/routes` | 动态路由树 |

### 4.3 路由注册顺序（重要！）

**固定路径端点必须注册在路径参数端点之前**，否则 Laravel 会优先匹配 `{id}/form` 等泛化路由：

```php
Route::prefix('users')->group(function () {
    Route::get('me', [SysUserController::class, 'me']);                 // 固定 —— 在前
    Route::get('options', [SysUserController::class, 'options']);       // 固定 —— 在前
    Route::get('export', [SysUserController::class, 'export']);         // 固定 —— 在前
    Route::get('{id}/form', [SysUserController::class, 'show'])         // 路径参数
        ->where(['id' => '[0-9]+']);
    apiIdResource(SysUserController::class);                            // index/store/show/update/destroy
});
```

> `apiIdResource()` 是项目封装助手：把标准 `apiResource` 的单参数 `id` 约束为数字，避免每个模块重复写 `->where()`。新增模块直接传控制器类名即可。

### 4.4 控制器方法模板

```php
<?php
namespace App\Http\Controllers\V1;

use App\Common\Enums\ResultCode;
use App\Exceptions\BusinessException;
use App\Http\Controllers\Controller;
use App\Http\Validates\SysUserValidate;
use App\Models\SysUser;
use App\Services\SysUserService;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class SysUserController extends Controller
{
    protected string $validatorClass = SysUserValidate::class;   // 绑定验证器

    public function __construct(
        protected SysUserService $userService,
    ) {}

    /**
     * 用户分页列表
     */
    public function index(Request $request): JsonResponse
    {
        $filters  = $request->only(['keywords', 'status', 'created_at', 'dept_id']);
        $pageSize = (int) $request->input('pageSize', self::PER_PAGE);
        $result   = $this->userService->paginate($filters, $pageSize);

        return $this->success($result);
    }

    /**
     * 创建用户
     */
    public function store(Request $request): JsonResponse
    {
        $validated = $this->validateScene($request, 'create');   // 调用 SysUserValidate::rules('create')
        $data = $this->userService->create($validated);
        return $this->success($this->userService->format($data), '创建成功');
    }
}
```

---

## Part 5: HTTP 状态码与业务错误码

### 5.1 设计理念

对齐 youlai-boot：**HTTP 状态码恒为 200**，业务状态码通过 body 的 `code` 字段传递。前端 axios 拦截器统一从 `response.data.code` 读取业务码。

### 5.2 业务错误码（ResultCode 枚举）

枚举定义在 `app/Common/Enums/ResultCode.php`，实现 `IResultCode` 接口，提供 `getCode()` / `getMsg()`：

| 错误码 | 枚举 | 含义 |
| --- | --- | --- |
| `00000` | `SUCCESS` | 成功 |
| `A0201` | `ACCOUNT_NOT_FOUND` | 用户账户不存在 |
| `A0202` | `ACCOUNT_FROZEN` | 用户账户被冻结 |
| `A0205` | `USER_TOKEN_INVALID` | 无效的 Token |
| `A0206` | `USER_TOKEN_EXPIRED` | Token 过期 |
| `A0210` | `USER_PASSWORD_ERROR` | 用户密码错误 |
| `A0211` | `DELETE_SUPER_ADMIN` | 不允许删除超级管理员 |
| `A0212` | `CANNOT_DELETE_SELF` | 不能删除当前登录账号 |
| `A0301` | `ACCESS_UNAUTHORIZED` | 访问未授权 |
| `A0400` | `USER_REQUEST_PARAMETER_ERROR` | 用户请求参数错误 |
| `B0001` | `SYSTEM_ERROR` | 系统执行出错 |
| `C0300` | `DATABASE_SERVICE_ERROR` | 数据库服务出错 |

首位编码规则：
- `0` 成功（`00000`）
- `A` 用户端错误（认证/权限/参数）
- `B` 系统端错误
- `C` 第三方/数据库错误

### 5.3 业务异常抛出

```php
use App\Common\Enums\ResultCode;
use App\Exceptions\BusinessException;

// ✅ 参数校验失败（控制器里由 validateScene 自动抛出，无需手写）
throw new BusinessException(ResultCode::USER_REQUEST_PARAMETER_ERROR->getCode(), $validator->errors()->first());

// ✅ 数据不存在
throw new BusinessException(ResultCode::ACCOUNT_NOT_FOUND->getMsg());

// ✅ 权限不足
throw new BusinessException(ResultCode::ACCESS_UNAUTHORIZED->getMsg());

// ✅ 不能删除自己
throw new BusinessException(ResultCode::CANNOT_DELETE_SELF->getMsg());
```

`BusinessException` 由 `bootstrap/app.php` 的异常处理器或全局异常渲染为统一响应（HTTP 200 + `code`/`msg`/`data`）。

---

## Part 6: 统一响应格式

### 6.1 Result 结构

```json
{
    "code": "00000",
    "msg": "操作成功",
    "data": { "…" },
    "traceId": "uuid-…"
}
```

响应由 `App\Traits\ApiResponse` 提供，所有控制器基类 `Controller` 已 `use ApiResponse`：

```php
// 成功（data 可空，自动 snake_case → camelCase）
return $this->success($data, '操作成功');

// 失败（不推荐直接用，优先用业务异常）
return $this->fail('操作失败', 'B0001');

// 业务错误（推荐，传枚举）
return $this->error(ResultCode::ACCESS_UNAUTHORIZED);

// 参数验证失败
return $this->validationError($errors, '参数验证失败');
```

### 6.2 分页响应

Eloquent 分页器（`LengthAwarePaginator`）直接作为 `data` 返回，经 `CaseConversion` 自动转为：

```json
{
    "code": "00000",
    "msg": "操作成功",
    "data": {
        "list": [ "…" ],
        "total": 100,
        "pageNum": 1,
        "pageSize": 10
    }
}
```

> **前端约定**：分页参数命名为 `pageNum` / `pageSize`（camelCase）。Laravel 端读取 `$request->input('pageSize')` 传给 `paginate()`，响应中由转换 Trait 输出驼峰键。

### 6.3 驼峰自动转换

`ApiResponse` 通过 `CaseConversion` trait 的 `convertToCamelCase()` 在输出时把数组键 / 对象属性从 snake_case 转为 camelCase。因此：

- Model 属性、`$request->input()` 用 snake_case（`dept_id`、`create_time`）
- 返回的 `data` 自动变 camelCase（`deptId`、`createTime`）
- 不需要手动在 Service 里 `toCamelCase`，但 `format()` 等手工组装数组时请直接用 camelCase 键名

---

## Part 7: Eloquent Model 规范

### 7.1 Model 继承 Base

```php
<?php
namespace App\Models;

use App\Traits\HasOperator;
use App\Traits\IsDeletedSoftDeletes;
use Illuminate\Database\Eloquent\Attributes\Fillable;
use Illuminate\Database\Eloquent\Attributes\Hidden;
use Tymon\JWTAuth\Contracts\JWTSubject;
use Illuminate\Foundation\Auth\User as Authenticatable;

#[Fillable(['username', 'password', 'avatar', 'mobile', 'email', 'status'])]
#[Hidden(['password', 'salt'])]
class SysUser extends Authenticatable implements JWTSubject
{
    use HasOperator, IsDeletedSoftDeletes;

    protected $table      = 'sys_user';
    protected $primaryKey = 'id';

    const string SYS_USER_TOKEN = 'admin';   // JWT guard 名
    const int USER_FREEZE   = 0;
    const int USER_ACTIVATE = 1;
    const string CAPTCHA_KEY = 'captcha:';

    // 自定义时间字段名（覆盖 Eloquent 默认 created_at/updated_at）
    const string CREATED_AT = 'create_time';
    const string UPDATED_AT = 'update_time';

    protected $casts = [
        'id'          => 'string',
        'status'      => 'integer',
        'create_time' => 'datetime',
        'update_time' => 'datetime',
    ];

    // 统一序列化格式
    protected function serializeDate(\DateTimeInterface $date): string
    {
        return $date->format('Y-m-d H:i:s');
    }
}
```

### 7.2 基础模型 Base

`app/Models/Base.php` 定义全项目共享常量，业务模型继承它：

| 常量 | 值 | 含义 |
| --- | --- | --- |
| `STATUS_ENABLE` | `1` | 启用 |
| `STATUS_DISABLE` | `0` | 禁用 |
| `HAVE_DELETED` / `NOT_DELETED` | `1` / `0` | 逻辑删除标记 |
| `CREATED_AT` / `UPDATED_AT` | `'create_time'` / `'update_time'` | 时间字段名 |

### 7.3 MySQL 类型映射

| MySQL 类型 | PHP 类型 / cast | 说明 |
| --- | --- | --- |
| `bigint` | `string`（cast） | 主键、外键（避免 JS 精度丢失） |
| `int` / `tinyint` | `integer` | 状态、标识位 |
| `varchar(n)` | `string` | 变长字符串 |
| `text` | `string` | 长文本 |
| `datetime` / `timestamp` | `datetime` | 时间 |
| `json` | `array` / `json` | JSON 字段 |

### 7.4 逻辑删除与字段命名

- 逻辑删除用 `is_deleted` 字段（由 `IsDeletedSoftDeletes` trait 处理），查询默认过滤 `is_deleted = 0`
- 字段命名对照：

| SQL 列名 | Model 属性 | 响应键（驼峰） | 说明 |
| --- | --- | --- | --- |
| `dept_id` | `dept_id` | `deptId` | 部门 ID |
| `create_time` | `create_time` | `createTime` | 创建时间 |
| `is_deleted` | `is_deleted` | — | 逻辑删除（内部） |
| `parent_id` | `parent_id` | `parentId` | 父节点 ID |

> **转换规则**：Model → 响应 时由 `ApiResponse` 自动完成 snake_case → camelCase，无需手写 `toVo()`。

### 7.5 关联与访问器

```php
// 多对多：用户-角色（带 tenant_id pivot）
public function roles(): BelongsToMany
{
    return $this->belongsToMany(SysRole::class, 'sys_user_role', 'user_id', 'role_id')
        ->withPivot('tenant_id')
        ->where('sys_role.is_deleted', Base::NOT_DELETED)
        ->where('status', Base::STATUS_ENABLE);
}

// 访问器：输出时计算 dept_name
public function getDeptNameAttribute(): string
{
    return $this->dept?->name ?? '';
}
```

---

## Part 8: 认证与鉴权

### 8.1 JWT 中间件

路由通过 `jwt.admin` 别名挂载认证中间件（注册于 `bootstrap/app.php`）：

```php
// bootstrap/app.php
$middleware->alias([
    'jwt.admin' => AdminAuthMiddleware::class,
    'permission' => PermissionMiddleware::class,
]);
```

`AdminAuthMiddleware` 行为：
- 用 `auth('admin')->check()` 校验 Token（`admin` 即 `SysUser::SYS_USER_TOKEN`）
- 过期抛 `USER_TOKEN_EXPIRED`，无效抛 `USER_NO_LOGIN`
- 通过后将 `authUser`（来自 `getJWTCustomClaims()`）写入 `$request->attributes->set('authUser', ...)`
- 当前登录用户用 `$request->user()` 或 `request()->user()` 获取

### 8.2 登录与 Token

```php
// AuthController::login
$user = $this->userService->accountLogin($account);   // 按 username/email/mobile 猜测字段
if (!password_verify($password, $user->password)) {
    throw new BusinessException(ResultCode::USER_PASSWORD_ERROR->getMsg());
}
return $this->success($this->buildToken($user), '登录成功');
// buildToken 返回 { access_token, refresh_token, token_type: Bearer, expires_in }
```

### 8.3 权限校验

- 权限标识格式：`模块:实体:操作`（如 `sys:user:query`、`sys:user:create`）
- 菜单表 `sys_menu.perm` 存放权限字符串，用户权限由角色聚合
- 权限判断在 Service 层通过 `authUser['authorities']`（角色编码 `ROLE_xxx`）与模型 `hasPermission()` 完成
- `PermissionMiddleware` 为占位实现，真正的鉴权在业务/路由层处理；新增敏感接口时按约定在 `routes/api.php` 包 `permission` 中间件或于 Service 内校验

### 8.4 多租户

- 模型带 `tenant_id` 字段；用户 JWT claims 含 `tenant_id`
- 切换租户：`POST /api/v1/auth/switch-tenant`
- 租户管理接口：`/api/v1/tenants`、`/api/v1/tenant-plans`
- 新增查询需在 Service 层携带 `tenant_id` 隔离条件，避免跨租户数据泄漏

---

## Part 9: 数据库操作规范

### 9.1 Eloquent 查询

```php
// ✅ 单条查询
$user = SysUser::with(['roles'])->findOrFail($id);

// ✅ 分页 + 筛选
$query = SysUser::query()->select(self::LIST_FIELDS)->with('dept')->orderBy('id', 'asc');
$this->applyFilters($query, $filters);     // 业务筛选
$query->paginate($perPage);

// ✅ 模糊搜索（orWhere 包子查询）
$query->where(function ($q) use ($keyword) {
    $q->whereLike('username', "%{$keyword}%")
      ->orWhere('nickname', 'like', "%{$keyword}%")
      ->orWhere('mobile', 'like', "%{$keyword}%");
});

// ✅ 关联存在性过滤（部门树）
$query->whereHas('dept', function ($q) use ($params) {
    $q->whereRaw("concat(',', concat(tree_path, ',', id), ',') like ?", ["%,{$params['dept_id']},%"]);
});
```

### 9.2 事务管理

```php
// ✅ 多步写用事务
return DB::transaction(function () use ($ids) {
    SysUserRole::query()->whereIn('user_id', $ids)->delete();
    SysUser::query()->whereIn('id', $ids)->delete();
    return count($ids);
});
```

### 9.3 批量与关联操作

```php
// ✅ 创建并同步多对多
$user = SysUser::query()->create($data);
if (!empty($data['role_ids'])) {
    $user->roles()->sync($data['role_ids']);   // 支持 tenant_id pivot
}

// ✅ 逻辑删除（is_deleted 标记）
SysUser::query()->whereIn('id', $ids)->update(['is_deleted' => Base::HAVE_DELETED]);

// ✅ 软删除 trait（IsDeletedSoftDeletes）自动过滤已删除
```

---

## Part 10: 添加新模块 — 完整教程

以添加 **"字典项"** 模块为例。

### Step 1: 创建 Model

```php
<?php
namespace App\Models;

use App\Traits\IsDeletedSoftDeletes;
use Illuminate\Database\Eloquent\Attributes\Fillable;

#[Fillable(['dict_code', 'value', 'label', 'tag_type', 'status', 'sort'])]
class SysDictItem extends Base
{
    protected $table = 'sys_dict_item';

    const string CREATED_AT = 'create_time';
    const string UPDATED_AT = 'update_time';
}
```

### Step 2: 创建 Validate

```php
<?php
namespace App\Http\Validates;

use Illuminate\Validation\Rule;

class SysDictItemValidate extends BaseValidate
{
    public function rules(string $scene = 'create', ?int $id = null): array
    {
        return match ($scene) {
            'create' => [
                'dict_code' => ['required', 'string', 'max:50'],
                'value'     => ['required', 'string', 'max:50'],
                'label'     => ['required', 'string', 'max:100'],
                'status'    => 'nullable|integer|in:0,1',
                'sort'      => 'nullable|integer',
            ],
            'update' => [
                'dict_code' => ['nullable', 'string', 'max:50'],
                'value'     => ['nullable', 'string', 'max:50'],
                'label'     => ['nullable', 'string', 'max:100'],
            ],
            default => [],
        };
    }

    public function messages(): array
    {
        return [
            'dict_code.required' => '字典编码不能为空',
            'value.required'     => '字典值不能为空',
            'label.required'     => '字典标签不能为空',
        ];
    }
}
```

### Step 3: 创建 Service

```php
<?php
namespace App\Services;

use App\Attributes\LogAction;
use App\Common\Enums\ActionType;
use App\Common\Enums\LogModule;
use App\Models\SysDictItem;

class SysDictItemService extends BaseService
{
    #[LogAction(module: LogModule::DICT, action: ActionType::QUERY)]
    protected function paginate(array $filters, int $perPage)
    {
        $query = SysDictItem::query()->where('is_deleted', 0);
        // … 筛选 + 分页
        return $query->paginate($perPage);
    }

    #[LogAction(module: LogModule::DICT, action: ActionType::CREATE)]
    protected function create(array $data): SysDictItem
    {
        return SysDictItem::query()->create($data);
    }

    protected function format(SysDictItem $item): array
    {
        return [
            'id'        => $item->id,
            'dictCode'  => $item->dict_code,
            'value'     => $item->value,
            'label'     => $item->label,
            'status'    => $item->status,
            'sort'      => $item->sort,
        ];
    }
}
```

### Step 4: 创建 Controller

```php
<?php
namespace App\Http\Controllers\V1;

use App\Http\Controllers\Controller;
use App\Http\Validates\SysDictItemValidate;
use App\Services\SysDictItemService;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class SysDictItemController extends Controller
{
    protected string $validatorClass = SysDictItemValidate::class;

    public function __construct(protected SysDictItemService $service) {}

    public function index(Request $request): JsonResponse
    {
        $filters  = $request->only(['dict_code', 'value']);
        $pageSize = (int) $request->input('pageSize', self::PER_PAGE);
        return $this->success($this->service->paginate($filters, $pageSize));
    }

    public function store(Request $request): JsonResponse
    {
        $validated = $this->validateScene($request, 'create');
        return $this->success($this->service->format($this->service->create($validated)), '创建成功');
    }
}
```

### Step 5: 注册路由

```php
// routes/api.php（在 jwt.admin 分组内）
Route::prefix('dicts')->group(function () {
    Route::get('{id}/form', [SysDictItemController::class, 'show'])
        ->where(['id' => '[0-9]+']);
    apiIdResource(SysDictItemController::class);
    Route::prefix('{name}/items')->group(function () {
        Route::get('/', [SysDictItemController::class, 'items']);
        Route::get('options', [SysDictItemController::class, 'options']);
    });
});
```

### Step 6: 验证

```bash
php artisan serve          # 启动 HTTP
php artisan sse:start --auth=admin start   # 启动 SSE（如需）
# 访问 /api/v1/dicts 确认路由已注册
```

---

## Part 11: 注释规范

### 11.1 总则

- **PHPDoc 优先**：所有公共类、方法用 `/** ... */` 文档注释
- **自解释代码**：好的命名和结构是自解释的，注释描述"为什么"
- **中文优先**：用中文把问题说清楚；专有名词保留英文（`HTTP`、`JWT`、`Eloquent`）
- **同步更新**：代码修改时文档注释必须同步修改

### 11.2 PHPDoc 格式

```php
/**
 * 用户分页列表
 * @param Request $request 含 pageNum / pageSize / keywords 等查询参数
 * @return JsonResponse 统一响应 { code, msg, data:{list,total} }
 * @throws BusinessException 参数校验失败 / 无权限
 */
public function index(Request $request): JsonResponse
```

### 11.3 各层注释要点

| 层 | 必须说明 |
| --- | --- |
| 控制器方法 | 功能、请求参数、返回值结构、可能异常 |
| Service 方法 | 业务语义、数据权限影响、事务范围 |
| Model | 表含义、`$table` / 时间字段 / 软删除策略 |
| Validate | 各 scene 的校验规则与中文报错 |

### 11.4 TODO / FIXME 标记

| 标记 | 用途 | 格式 |
| --- | --- | --- |
| `TODO` | 待实现 | `// TODO(标记人, 日期, 说明)` |
| `FIXME` | 已知 bug | `// FIXME(标记人, 日期, 说明)` |

---

## Part 12: 代码质量检查清单

### 12.1 通用检查

- [ ] 所有公共类/方法有 PHPDoc
- [ ] 响应经 `ApiResponse` 输出（`success`/`error`/`fail`），不手动 `response()->json`
- [ ] 数据经 `CaseConversion` 自动转驼峰，手工数组用 camelCase 键
- [ ] 配置通过 `.env` + `config/` 管理，不硬编码
- [ ] 敏感信息（密码、token）不写入日志、不进入响应
- [ ] `composer lint`（Laravel Pint）无风格问题

### 12.2 数据库相关

- [ ] Model 字段名 snake_case，与 MySQL 列名一致
- [ ] 所有查询携带 `is_deleted = 0`（或经软删除 trait）
- [ ] 多步写操作使用 `DB::transaction`
- [ ] `id` 用 `string` cast 避免 JS 精度问题
- [ ] 时间字段用 `create_time` / `update_time`（覆盖 Eloquent 默认）

### 12.3 API 相关

- [ ] 路由 prefix 统一 `/api/v1/模块名`
- [ ] 固定路径端点注册在 `{id}` 路径参数之前
- [ ] 分页参数命名为 `pageNum` / `pageSize`
- [ ] 响应 JSON 键驼峰（`deptId`、`createTime`）
- [ ] HTTP 状态码恒为 200，业务码通过 body 的 `code` 传递
- [ ] 错误用 `ResultCode` 枚举，不硬编码字符串
- [ ] 敏感接口挂载 `jwt.admin` 中间件
- [ ] 多租户查询携带 `tenant_id` 隔离

### 12.4 Service 层

- [ ] 控制器注入 Service（构造器），不直接写查询
- [ ] 数据不存在返回 `BusinessException(ACCOUNT_NOT_FOUND / NO_DATA_WAS_FOUND)`
- [ ] 批量删除前检查参数非空、保护超级管理员
- [ ] `format()` 统一把 Model 转为驼峰响应数组
- [ ] 写操作加 `#[LogAction]` 属性记录操作日志

---

## Part 13: 常见反模式

| 反模式 | 问题 | 正确做法 |
| --- | --- | --- |
| 控制器里写 SQL/查询 | 违反分层 | 查询放到 Service |
| `response()->json([...])` 手写 | 格式不统一、无 traceId | 用 `ApiResponse::success/error` |
| 响应里手写 snake_case 键 | 前端解析失败 | 靠 `CaseConversion` 自动转，或手工用 camelCase |
| SQL 缺 `is_deleted = 0` | 查到已删除数据 | 所有查询加逻辑删除条件 / 用软删除 trait |
| 固定路径在 `{id}` 之后 | 路由被错误匹配 | 固定路径注册在前 |
| `code="A0210"` 硬编码字符串 | 易出错、难维护 | 用 `ResultCode::USER_PASSWORD_ERROR` |
| 密码明文存储 | 安全事故 | `password_hash()` / `Hash::make()` |
| 多租户漏加 `tenant_id` | 跨租户泄漏 | Service 查询携带租户隔离 |
| `dd()` / `dump()` 调试 | 生产泄露 | 用 Laravel 日志 `info()`/`debug()` |
| 验证写在控制器 | 复用差 | 用 `Validate` 类 + `validateScene()` |

---

## Part 14: Docker 部署

```yaml
# docker-compose.yml（示意）
services:
  app:
    build: .
    ports: ["8000:8000"]
    depends_on: [mysql, redis]
    environment:
      DB_HOST: mysql
      DB_DATABASE: youlai_admin
      DB_USERNAME: root
      DB_PASSWORD: "123456"
      REDIS_HOST: redis
    command: php artisan serve --host=0.0.0.0 --port=8000

  sse:
    build: .
    command: php artisan sse:start --auth=admin start
    depends_on: [app]

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: youlai_admin
      MYSQL_ROOT_PASSWORD: "123456"
    volumes:
      - ./database/youlai_admin.sql:/docker-entrypoint-initdb.d/init.sql

  redis:
    image: redis:7-alpine
```

> 部署前执行：`composer install --no-dev` 导入 `database/youlai_admin.sql`，并配置 `.env` 的 `APP_KEY`、`DB_*`、`REDIS_*` 与 `SSE_LISTEN_URL_ADMIN`。

---

## Part 15: 从 0 到 1 创建项目指引（AI 专用）

> 当 AI 助手被要求"依据规范创建 youlai-laravel 模块/项目"时，按以下步骤执行。

### 必读文件

1. **仓库**：`youlai-laravel`（完整目录、路由、示例模块）
2. **本 Skill**：`youlai-skills/skills/laravel/SKILL.md`（命名规范、开发约定）
3. **参考实现**：`app/Http/Controllers/V1/SysUserController.php`、`app/Services/SysUserService.php`、`app/Http/Validates/SysUserValidate.php`

### 创建步骤

1. **准备数据库**：导入 `database/youlai_admin.sql` 到 MySQL 8.0+
2. **配置环境**：复制 `.env.example` → `.env`，设置 `APP_KEY`、`DB_*`、`REDIS_*`，执行 `composer install`
3. **按目录结构创建骨架**（见 Part 2）
4. **新增模块按 Part 10 流程**：Model → Validate → Service → Controller → 路由
5. **统一响应**：控制器继承 `Controller`，用 `success()`/`error()`；Model 用 snake_case，响应自动驼峰
6. **认证**：敏感路由挂 `jwt.admin` 中间件；Service 用 `$request->user()` 取当前用户
7. **多租户**：查询携带 `tenant_id` 隔离
8. **日志**：写操作加 `#[LogAction]` 属性
9. **验证**：`php artisan serve` + `php artisan sse:start` + PHPUnit

### 关键约束

- 数据库为 **MySQL 8.0+**，逻辑删除用 `is_deleted`
- API 路径、响应格式、错误码 **100% 对齐 youlai-boot**（见 API 契约文档）
- HTTP 状态码恒为 200，业务码通过 `code` 字段传递
- JSON 键名驼峰（由 `ApiResponse` 的 `CaseConversion` 自动转换）
- 默认账号：`admin` / `123456`
- **含多租户**，新增模块必须处理 `tenant_id` 隔离
