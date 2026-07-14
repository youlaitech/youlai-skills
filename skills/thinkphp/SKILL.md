---
name: thinkphp
description: ThinkPHP backend development standards. Use this skill when developing ThinkPHP projects, implementing REST APIs, model data access, or JWT authentication.
---

# ThinkPHP 后端开发规范

> 参考：[ThinkPHP 8.0 开发规范](https://doc.thinkphp.cn/v8_0/development_specifications.html) | [PSR-4 自动加载](https://www.php-fig.org/psr/psr-4/)

## 触发条件

- Develop ThinkPHP projects
- Implement REST APIs
- Use model data access
- Implement JWT authentication
- Implement permission control

---

## Part 1: 技术栈

| 层 | 选型 | 说明 |
|----|------|------|
| 运行环境 | **PHP 8.1+** | 纤程、枚举、只读属性 |
| 框架 | **ThinkPHP 8.x** | MVC 架构、ORM 内置 |
| 数据库 | **MySQL 8.x** | InnoDB 引擎 |
| 缓存 | **Redis 7.x** | 分布式缓存 |
| 认证 | **firebase/php-jwt** | JWT 签发/验签 |
| API 文档 | **Swagger UI** (swagger.json) | 静态文档 |

---

## Part 2: 目录结构

> 基础目录以 [ThinkPHP 8 官方目录结构](https://doc.thinkphp.cn/v8_0/directory_structure.html) 为准；`app/` 在此基础上按业务模块（`auth`、`system` 等）划分，模块内含 controller/model/service/validate。

```
app/
├── BaseController.php              # 基础控制器
├── ExceptionHandle.php             # 全局异常处理
├── common.php, middleware.php       # 公共函数 / 中间件定义
│
├── auth/                           # 认证模块
│   ├── controller/{AuthController, WxMaAuthController}
│   └── service/{AuthService, WxMaAuthService}
│
├── system/                         # 系统管理
│   ├── controller/{UserController, RoleController, MenuController, ...}
│   ├── model/{User, Role, Menu, ...}
│   ├── service/{UserService, RoleService, RolePermService, DataPermissionService, ...}
│   ├── validate/{UserValidate, RoleValidate}
│   └── enums/{ActionType, LogModule}
│
├── codegen/                        # 代码生成器
├── file/                           # 文件管理
│
├── common/                         # 公共代码
│   ├── constants/                  # RedisConstants, RedisKey
│   ├── enums/                      # DataScopeEnum
│   ├── exception/                  # BusinessException
│   ├── middleware/                  # Auth, Perm, DataScope, Cors, Log, RateLimit, ConvertCase
│   ├── model/BaseModel.php
│   ├── traits/{AuthTrait, ParamsTrait}
│   ├── util/{CaseConverter, IdStringify, PageUtil, VerifyCodeHelper}
│   └── web/                        # Result, PageResult, ResultCode, IResultCode
│
├── controller/BaseController.php   # 基础控制器
│
extend/                             # 扩展类库
├── jwt/{JwtTokenManager, TokenManager, ...}
├── redis/{RedisClient, KeyFormatter}
├── sse/{SseEmitter, SseService, ...}
└── http/HttpClient.php

config/   route/   public/   sql/
```

**设计原则**：
- `common/` 是公共组件，被所有模块共享
- `extend/` 是第三方/框架扩展（JWT/Redis/SSE），不依赖业务
- `app/{module}/` 按模块组织，每模块含 controller/model/service/validate

---

## Part 3: 命名规范

> 遵循 [ThinkPHP 8.0 开发规范](https://doc.thinkphp.cn/v8_0/development_specifications.html) + [PSR-4](https://www.php-fig.org/psr/psr-4/)。

### 3.1 文件和类

| 规则 | 示例 |
|------|------|
| 目录使用小写+下划线 | `controller/`, `user_service/` |
| 类名采用驼峰法（首字母大写） | `UserController`, `UserValidate` |
| 类名与文件名一致 | `UserController` → `UserController.php` |
| 命名空间与目录路径一致 | `app\system\controller` → `app/system/controller/` |

### 3.2 常量和配置

| 类型 | 规范 | 示例 |
|------|------|------|
| 常量 | 大写字母和下划线 | `APP_PATH`, `HAS_ONE` |
| 配置参数 | 小写字母和下划线 | `url_route_on` |
| 环境变量 | 大写字母和下划线 | `APP_DEBUG`, `DB_HOST` |

### 3.3 数据表和字段

| 规则 | 示例 |
|------|------|
| 数据表使用小写+下划线 | `sys_user`, `sys_role_menu` |
| 字段使用小写+下划线 | `user_name`, `create_time` |
| 禁止驼峰和中文命名 | ❌ `userName`, ❌ `用户表` |

### 3.4 函数和变量

| 类型 | 规范 | 示例 |
|------|------|------|
| 方法 | 驼峰法（首字母小写） | `getUserName()` |
| 全局函数 | 小写字母和下划线 | `get_client_ip()` |
| 属性 | 驼峰法（首字母小写） | `$tableName` |

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

### 4.2 Controller 模板

```php
declare(strict_types=1);
namespace app\system\controller;

use app\BaseController;
use app\system\model\User;
use app\system\validate\UserValidate;
use app\common\exception\BusinessException;
use app\common\web\ResultCode;

final class UserController extends BaseController
{
    protected bool $requireAuth = true;

    /** 用户分页列表 */
    public function page()
    {
        $params = $this->request->get();
        $result = User::where(function ($q) use ($params) {
            if (!empty($params['keywords']))
                $q->whereLike('username|nickname', $params['keywords']);
            if (isset($params['status']))
                $q->where('status', $params['status']);
        })->order('create_time', 'desc')
          ->paginate(['page' => $params['pageNum'] ?? 1, 'list_rows' => $params['pageSize'] ?? 20]);
        return $this->successPaginate($result->items(), $result->total());
    }

    /** 新增用户 */
    public function create()
    {
        $params = $this->request->post();
        $this->validate($params, UserValidate::class, 'create');
        if (User::where('username', $params['username'])->find())
            throw new BusinessException(ResultCode::USER_ERROR, '用户名已存在');

        $user = new User();
        $user->username = $params['username'];
        $user->password = password_hash($params['password'], PASSWORD_BCRYPT);
        $user->save();
        return $this->success(['id' => $user->id]);
    }
}
```

---

## Part 5: 响应格式与异常处理

### 5.1 统一响应

```php
class Result
{
    public static function success(mixed $data = null, string $msg = '成功'): array
    {
        return ['code' => '00000', 'msg' => $msg, 'data' => $data];
    }

    public static function page(array $list, int $total): array
    {
        return ['code' => '00000', 'msg' => '成功', 'data' => ['list' => $list, 'total' => $total]];
    }

    public static function failedWith(ResultCode $code, string $msg = ''): array
    {
        return ['code' => $code->value, 'msg' => $msg ?: $code->getMsg(), 'data' => null];
    }
}
```

### 5.2 基础控制器响应方法

```php
abstract class BaseController
{
    protected function success(mixed $data = null, string $msg = ''): Json
    {
        return json(Result::success($data, $msg ?: ResultCode::SUCCESS->getMsg()));
    }

    protected function successPaginate(array $list, int $total): Json
    {
        return json(Result::page($list, $total));
    }
}
```

### 5.3 业务异常

```php
final class BusinessException extends RuntimeException
{
    private ResultCode $resultCode;

    public function __construct(ResultCode $resultCode, string $message = '')
    {
        $this->resultCode = $resultCode;
        parent::__construct($message ?: $resultCode->getMsg());
    }

    public function getResultCode(): ResultCode { return $this->resultCode; }
}
```

### 5.4 全局异常处理

```php
class ExceptionHandle extends Handle
{
    public function render($request, Throwable $e): Response
    {
        if ($e instanceof BusinessException)
            return json(Result::failedWith($e->getResultCode(), $e->getMessage()));
        if ($e instanceof ValidateException)
            return json(Result::failedWith(ResultCode::PARAM_ERROR, $e->getError()));
        return parent::render($request, $e); // 生产环境隐藏详情
    }
}
```

```php
// config/app.php — 注册异常处理器
return ['exception_handle' => \app\ExceptionHandle::class];
```

---

## Part 6: 模型规范

```php
class BaseModel extends Model
{
    protected $autoWriteTimestamp = true;
    protected $createTime = 'create_time';
    protected $updateTime = 'update_time';

    use \think\model\concern\SoftDelete;
    protected $deleteTime = 'delete_time';
    protected $defaultSoftDelete = 0;
}

final class User extends BaseModel
{
    protected $name = 'sys_user';
    protected $pk = 'id';
    protected $hidden = ['password', 'delete_time'];

    public function dept()
    {
        return $this->belongsTo(Dept::class, 'dept_id');
    }

    public function roles()
    {
        return $this->belongsToMany(Role::class, UserRole::class, 'role_id', 'user_id');
    }
}
```

---

## Part 7: 路由规范

```php
use think\facade\Route;

// 认证接口（无需登录）
Route::group('api/v1/auth', function () {
    Route::post('login', 'auth.controller.AuthController/login');
    Route::post('logout', 'auth.controller.AuthController/logout');
    Route::post('refresh-token', 'auth.controller.AuthController/refreshToken');
})->allowCrossDomain();

// 需要登录的接口
Route::group('api/v1', function () {
    Route::group('users', function () {
        Route::get('page', 'system.controller.UserController/page');
        Route::get(':id', 'system.controller.UserController/detail');
        Route::post('', 'system.controller.UserController/create');
        Route::put('', 'system.controller.UserController/update');
        Route::delete(':id', 'system.controller.UserController/delete');
        Route::delete('batch', 'system.controller.UserController/batchDelete');
    });
})->middleware(\app\middleware\Auth::class)->allowCrossDomain();
```

---

## Part 8: 认证规范

### 8.1 JWT 服务

```php
final class JwtService
{
    public static function generateAccessToken(array $payload): string
    {
        $payload['exp'] = time() + Config::get('jwt.access_ttl', 7200);
        $payload['iat'] = time();
        $payload['iss'] = Config::get('jwt.issuer', 'youlai-think');
        return JWT::encode($payload, Config::get('jwt.secret'), 'HS256');
    }

    public static function parseToken(string $token): ?array
    {
        try {
            return (array) JWT::decode($token, new Key(Config::get('jwt.secret'), 'HS256'));
        } catch (\Exception) { return null; }
    }
}
```

### 8.2 认证中间件

```php
final class Auth
{
    public function handle(Request $request, Closure $next): Response
    {
        $token = str_replace('Bearer ', '', $request->header('Authorization', ''));
        if (empty($token)) return json(Result::failedWith(ResultCode::ACCESS_TOKEN_INVALID));

        $payload = JwtService::parseToken($token);
        if (!$payload) return json(Result::failedWith(ResultCode::ACCESS_TOKEN_INVALID));

        $request->userId = $payload['sub'];
        $request->username = $payload['username'] ?? '';
        return $next($request);
    }
}
```

---

## Part 9: 验证器规范

```php
final class UserValidate extends Validate
{
    protected $rule = [
        'username' => 'require|max:50',
        'password' => 'require|min:6|max:20',
        'nickname' => 'require|max:50',
        'mobile' => 'mobile',
        'status' => 'in:0,1',
    ];

    protected $scene = [
        'create' => ['username', 'password', 'nickname', 'mobile', 'status'],
        'update' => ['id', 'nickname', 'mobile', 'status'],
    ];
}
```

---

## Part 10: 注释规范

- **所有公共方法用 PHPDoc `/** */` 注释**
- 方法注释含 `@param`/`@return`/`@throws`
- `declare(strict_types=1)` 在文件首行（PHP 8 强制类型检查）

```php
/**
 * 用户管理控制器。
 */
final class UserController extends BaseController
{
    /**
     * 创建新用户。
     *
     * @return Json {"code":"00000","data":{"id":1}}
     * @throws BusinessException 用户名已存在
     */
    public function create()
    {
        // ...
    }
}
```

---

## Part 11: 代码质量检查清单

- [ ] 目录使用小写+下划线，类名驼峰法
- [ ] 数据表和字段使用小写+下划线
- [ ] 遵循 RESTful API 路径规范
- [ ] 使用统一响应格式（`Result` / `ResultCode`）
- [ ] 实现全局异常处理（`ExceptionHandle`）
- [ ] 业务异常使用 `BusinessException` + `ResultCode`
- [ ] 模型继承 `BaseModel`，使用软删除
- [ ] 使用 `UserValidate` 验证器校验参数
- [ ] 路由显式映射
- [ ] `declare(strict_types=1)` 在文件首行

### 常见反模式

| 反模式 | 正确做法 |
|--------|----------|
| `return json(['code' => 0])` 散落各处 | 用 `Result::success()` 统一封装 |
| Controller 中直接 SQL 查询 | 委托 Model / Service 层 |
| `if (!$user) { exit('用户不存在'); }` | 抛 `BusinessException`，走全局异常处理 |
| `catch (\Exception $e) { }` 空捕获 | 记录日志或重新抛出 `BusinessException` |
| 表名使用驼峰 | `sys_user`（小写+下划线），禁止 `SysUser` |
