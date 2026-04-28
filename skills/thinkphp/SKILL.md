---
name: thinkphp
description: ThinkPHP backend development standards. Use this skill when developing ThinkPHP projects, implementing REST APIs, model data access, or JWT authentication.
---

# ThinkPHP 后端开发规范

> 参考：[ThinkPHP 8.0 开发规范](https://doc.thinkphp.cn/v8_0/development_specifications.html) | [ThinkPHP 8.0 目录结构](https://doc.thinkphp.cn/v8_0/directory_structure.html)

## 触发条件

- Develop ThinkPHP projects
- Implement REST APIs
- Use model data access
- Implement JWT authentication
- Implement permission control

---

## Part 1: 技术栈

- **PHP** - 运行环境
- **ThinkPHP** - Web 框架
- **MySQL** - 数据库
- **Redis** - 缓存
- **firebase/php-jwt** - Token 认证

---

## Part 2: 目录结构

> 参考：[ThinkPHP 8.0 目录结构](https://doc.thinkphp.cn/v8_0/directory_structure.html)

```
app/
├─ BaseController.php              # 基础控制器类
├─ ExceptionHandle.php             # ★ 全局异常处理类
├─ common.php                      # 全局公共函数
├─ middleware.php                   # 全局中间件定义
├─ provider.php                    # 服务提供定义
├─ event.php                       # 全局事件定义
│
├─ auth/                           # 认证模块
│  ├─ controller/
│  │  ├─ AuthController.php        # 登录/登出/刷新令牌
│  │  └─ WxMaAuthController.php    # 微信小程序认证
│  └─ service/
│     ├─ AuthService.php
│     └─ WxMaAuthService.php
│
├─ system/                         # 系统管理模块
│  ├─ annotation/
│  │  └─ Log.php                   # @Log 注解（操作日志标记）
│  ├─ controller/
│  │  ├─ UserController.php
│  │  ├─ RoleController.php
│  │  ├─ MenuController.php
│  │  ├─ DeptController.php
│  │  ├─ DictController.php
│  │  ├─ ConfigController.php
│  │  ├─ NoticeController.php
│  │  └─ LogController.php
│  ├─ model/                       # 模型（User, Role, Menu 等）
│  ├─ service/                     # 业务逻辑
│  │  ├─ UserService.php
│  │  ├─ RoleService.php
│  │  ├─ RolePermService.php
│  │  ├─ DataPermissionService.php
│  │  └─ ...
│  ├─ enums/
│  │  ├─ ActionType.php
│  │  └─ LogModule.php
│  └─ validate/
│     ├─ UserValidate.php
│     └─ RoleValidate.php
│
├─ codegen/                        # 代码生成模块
│  ├─ controller/CodegenController.php
│  └─ service/CodegenService.php
│
├─ file/                           # 文件模块
│  ├─ controller/FileController.php
│  └─ service/FileService.php
│
├─ common/                         # 公共代码（非模块）
│  ├─ constants/                   # 常量（RedisConstants, RedisKey, NoticeEvents）
│  ├─ enums/                       # 枚举（DataScopeEnum）
│  ├─ exception/                   # ★ 异常类（BusinessException）
│  ├─ middleware/                   # ★ 中间件（7 个）
│  │  ├─ AuthMiddleware.php        # 认证
│  │  ├─ PermMiddleware.php        # 接口权限
│  │  ├─ DataScopeMiddleware.php   # 数据权限
│  │  ├─ Cors.php                  # 跨域
│  │  ├─ LogMiddleware.php         # 操作日志
│  │  ├─ RateLimitMiddleware.php   # 限流
│  │  └─ ConvertCaseMiddleware.php # 大小写转换
│  ├─ model/
│  │  └─ BaseModel.php             # 基础模型
│  ├─ traits/                      # ★ 复用特性
│  │  ├─ AuthTrait.php             # 认证复用
│  │  └─ ParamsTrait.php           # 参数处理复用
│  ├─ util/                        # 工具类
│  │  ├─ CaseConverter.php
│  │  ├─ IdStringify.php
│  │  ├─ PageUtil.php
│  │  ├─ TemplateRenderer.php
│  │  └─ VerifyCodeHelper.php
│  ├─ validate/
│  │  └─ BaseValidate.php          # 基础验证器
│  └─ web/                         # ★ 统一响应封装
│     ├─ IResultCode.php           # 结果码接口
│     ├─ ResultCode.php            # 结果码枚举
│     ├─ Result.php                # 统一响应类
│     └─ PageResult.php            # 分页响应类
│
├─ controller/BaseController.php   # 基础控制器

extend/                            # 扩展类库
├─ jwt/                            # JWT 认证
│  ├─ JwtTokenManager.php
│  ├─ TokenManager.php
│  ├─ TokenManagerResolver.php
│  ├─ AuthenticationToken.php
│  ├─ JwtClaimConstants.php
│  └─ SecurityConstants.php
├─ redis/                          # Redis 工具
│  ├─ RedisClient.php
│  └─ KeyFormatter.php
├─ sse/                            # SSE 推送
│  ├─ SseEmitter.php
│  ├─ SseService.php
│  ├─ SseEventPublisher.php
│  ├─ SseSessionRegistry.php
│  ├─ SseTopics.php
│  ├─ SseWorkerServer.php
│  └─ NoticeListener.php
└─ http/
   └─ HttpClient.php

config/                            # 应用配置
├─ app.php, database.php, cache.php, log.php
├─ security.php, wechat.php, captcha.php
├─ filesystem.php, queue.php, worker.php
├─ middleware.php, route.php, trace.php

route/
└─ app.php                         # 路由定义

public/
├─ index.php                       # 入口文件
├─ swagger.json                    # API 文档
└─ swagger-ui/index.html

sql/
└─ mysql/youlai_admin.sql
```

---

## Part 3: 命名规范

> 参考：[ThinkPHP 8.0 开发规范](https://doc.thinkphp.cn/v8_0/development_specifications.html)

### 目录和文件

| 规则 | 示例 |
| --- | --- |
| 目录使用小写+下划线 | `controller/`, `user_service/` |
| 类库、函数文件统一以 `.php` 为后缀 | `UserController.php` |
| 类的文件名与命名空间路径一致 | `app\system\controller\UserController` → `app/system/controller/UserController.php` |
| 类文件采用驼峰法命名（首字母大写） | `UserController.php`, `UserType.php` |
| 其它文件采用小写+下划线命名 | `common.php`, `route.php` |

### 类命名

| 规则                         | 示例                                       |
| ---------------------------- | ------------------------------------------ |
| 类名采用驼峰法（首字母大写） | `User`, `UserType`, `UserController`       |
| 类名与文件名保持一致         | `UserController` 类 → `UserController.php` |

### 方法和属性命名

| 规则                         | 示例                             |
| ---------------------------- | -------------------------------- |
| 方法采用驼峰法（首字母小写） | `getUserName()`, `getUserById()` |
| 属性采用驼峰法（首字母小写） | `$tableName`, `$instance`        |
| 魔术方法以双下划线开头       | `__call()`, `__autoload()`       |

### 函数命名

| 规则                     | 示例                                 |
| ------------------------ | ------------------------------------ |
| 函数使用小写字母和下划线 | `get_client_ip()`, `array_to_tree()` |

### 常量和配置

| 类型     | 规则             | 示例                          |
| -------- | ---------------- | ----------------------------- |
| 常量     | 大写字母和下划线 | `APP_PATH`, `HAS_ONE`         |
| 配置参数 | 小写字母和下划线 | `url_route_on`, `url_convert` |
| 环境变量 | 大写字母和下划线 | `APP_DEBUG`, `DB_HOST`        |

### 数据表和字段

| 规则                   | 示例                        |
| ---------------------- | --------------------------- |
| 数据表采用小写+下划线  | `sys_user`, `sys_role_menu` |
| 字段采用小写+下划线    | `user_name`, `created_at`   |
| 禁止使用驼峰和中文命名 | ❌ `userName`, ❌ `用户表`  |

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

```php
<?php
declare(strict_types=1);

namespace app\system\controller;

use app\BaseController;
use app\system\model\User;
use app\system\validate\UserValidate;
use app\common\exception\BusinessException;
use app\common\web\ResultCode;
use think\facade\Request;

final class UserController extends BaseController
{
    protected bool $requireAuth = true;

    /**
     * 用户分页列表
     */
    public function page()
    {
        $params = $this->request->get();
        $result = User::where(function ($query) use ($params) {
            if (!empty($params['keywords'])) {
                $query->whereLike('username|nickname', $params['keywords']);
            }
            if (isset($params['status'])) {
                $query->where('status', $params['status']);
            }
        })
        ->order('create_time', 'desc')
        ->paginate([
            'page' => $params['pageNum'] ?? 1,
            'list_rows' => $params['pageSize'] ?? 20,
        ]);

        return $this->successPaginate($result->items(), $result->total());
    }

    /**
     * 用户详情
     */
    public function detail($id)
    {
        $user = User::find($id);
        if (!$user) {
            throw new BusinessException(ResultCode::USER_ERROR);
        }
        return $this->success($user->toArray());
    }

    /**
     * 新增用户
     */
    public function create()
    {
        $params = $this->request->post();
        $this->validate($params, UserValidate::class, 'create');

        // 检查用户名是否存在
        if (User::where('username', $params['username'])->find()) {
            throw new BusinessException(ResultCode::USER_ERROR, '用户名已存在');
        }

        $user = new User();
        $user->username = $params['username'];
        $user->password = password_hash($params['password'], PASSWORD_BCRYPT);
        $user->nickname = $params['nickname'];
        $user->mobile = $params['mobile'] ?? '';
        $user->email = $params['email'] ?? '';
        $user->status = $params['status'] ?? 1;
        $user->save();

        return $this->success(['id' => $user->id]);
    }

    /**
     * 更新用户
     */
    public function update()
    {
        $params = $this->request->put();
        $this->validate($params, UserValidate::class, 'update');

        $user = User::find($params['id']);
        if (!$user) {
            throw new BusinessException(ResultCode::USER_ERROR);
        }

        $user->nickname = $params['nickname'];
        $user->mobile = $params['mobile'] ?? '';
        $user->email = $params['email'] ?? '';
        $user->status = $params['status'] ?? 1;
        $user->save();

        return $this->success();
    }

    /**
     * 删除用户
     */
    public function delete($id)
    {
        $user = User::find($id);
        if (!$user) {
            throw new BusinessException(ResultCode::USER_ERROR);
        }
        $user->delete();
        return $this->success();
    }

    /**
     * 批量删除
     */
    public function batchDelete()
    {
        $ids = $this->request->delete('ids');
        User::destroy($ids);
        return $this->success();
    }

    /**
     * 用户选项列表
     */
    public function options()
    {
        $list = User::field('id, nickname')->select();
        return $this->success($list->toArray());
    }
}
```

---

## Part 5: 响应格式

> 参考：[接口协议](/reference/api-specification)

```php
<?php
declare(strict_types=1);

namespace app\common\web;

use think\response\Json;

class Result
{
    public static function success(mixed $data = null, string $msg = '成功'): array
    {
        return [
            'code' => '00000',
            'msg' => $msg,
            'data' => $data,
        ];
    }

    public static function page(array $list, int $total): array
    {
        return [
            'code' => '00000',
            'msg' => '成功',
            'data' => [
                'list' => $list,
                'total' => $total,
            ],
        ];
    }

    public static function failedWith(ResultCode $code, string $msg = ''): array
    {
        return [
            'code' => $code->value,
            'msg' => $msg ?: $code->getMsg(),
            'data' => null,
        ];
    }
}
```

### BaseController 响应方法

```php
<?php
declare(strict_types=1);

namespace app;

use app\common\web\Result;
use app\common\web\ResultCode;
use think\App;
use think\response\Json;

abstract class BaseController
{
    protected App $app;
    protected $request;
    protected bool $requireAuth = false;

    public function __construct(App $app)
    {
        $this->app = $app;
        $this->request = $app->request;
        $this->initialize();
    }

    protected function initialize(): void
    {
        if ($this->requireAuth && $this->getAuthUserId() <= 0) {
            throw new BusinessException(ResultCode::ACCESS_TOKEN_INVALID);
        }
    }

    protected function success(mixed $data = null, string $msg = ''): Json
    {
        return json(Result::success($data, $msg ?: ResultCode::SUCCESS->getMsg()));
    }

    protected function successPaginate(array $list, int $total): Json
    {
        return json(Result::page($list, $total));
    }

    protected function fail(ResultCode $code, string $msg = ''): Json
    {
        return json(Result::failedWith($code, $msg));
    }
}
```

---

## Part 6: 模型规范

### 基础模型

```php
<?php
declare(strict_types=1);

namespace app\common\model;

use think\Model;

class BaseModel extends Model
{
    protected $autoWriteTimestamp = true;
    protected $createTime = 'create_time';
    protected $updateTime = 'update_time';

    use \think\model\concern\SoftDelete;
    protected $deleteTime = 'delete_time';
    protected $defaultSoftDelete = 0;

    public static function onBeforeInsert($model): void
    {
        $model->create_by = request()->userId ?? 0;
        $model->update_by = request()->userId ?? 0;
    }

    public static function onBeforeUpdate($model): void
    {
        $model->update_by = request()->userId ?? 0;
    }
}
```

### 模型示例

```php
<?php
declare(strict_types=1);

namespace app\system\model;

use app\common\model\BaseModel;

final class User extends BaseModel
{
    protected $name = 'sys_user';
    protected $pk = 'id';

    protected $hidden = ['password', 'delete_time'];

    protected $type = [
        'id' => 'integer',
        'status' => 'integer',
        'create_time' => 'datetime',
        'update_time' => 'datetime',
    ];

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
<?php
use think\facade\Route;

// 认证接口（无需登录）
Route::group('api/v1/auth', function () {
    Route::post('login', 'auth.controller.AuthController/login');
    Route::post('logout', 'auth.controller.AuthController/logout');
    Route::post('refresh-token', 'auth.controller.AuthController/refreshToken');
})->allowCrossDomain();

// 需要登录的接口
Route::group('api/v1', function () {
    // 用户管理
    Route::group('users', function () {
        Route::get('page', 'system.controller.UserController/page');
        Route::get(':id', 'system.controller.UserController/detail');
        Route::get('options', 'system.controller.UserController/options');
        Route::post('', 'system.controller.UserController/create');
        Route::put('', 'system.controller.UserController/update');
        Route::delete(':id', 'system.controller.UserController/delete');
        Route::delete('batch', 'system.controller.UserController/batchDelete');
    });

    // 角色管理
    Route::group('roles', function () {
        Route::get('page', 'system.controller.RoleController/page');
        // ...
    });
})->middleware(\app\middleware\Auth::class)
  ->allowCrossDomain();
```

---

## Part 8: 认证规范

### JWT 服务

```php
<?php
declare(strict_types=1);

namespace app\common\security;

use Firebase\JWT\JWT;
use Firebase\JWT\Key;
use think\facade\Config;

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
            $decoded = JWT::decode($token, new Key(Config::get('jwt.secret'), 'HS256'));
            return (array) $decoded;
        } catch (\Exception $e) {
            return null;
        }
    }

    public static function generateRefreshToken(int $userId): string
    {
        return JWT::encode([
            'sub' => $userId,
            'type' => 'refresh',
            'exp' => time() + Config::get('jwt.refresh_ttl', 604800),
        ], Config::get('jwt.secret'), 'HS256');
    }
}
```

### 认证中间件

```php
<?php
declare(strict_types=1);

namespace app\middleware;

use app\common\security\JwtService;
use app\common\web\ResultCode;
use app\common\web\Result;
use Closure;
use think\Request;
use think\Response;

final class Auth
{
    public function handle(Request $request, Closure $next): Response
    {
        $token = $request->header('Authorization', '');
        $token = str_replace('Bearer ', '', $token);

        if (empty($token)) {
            return json(Result::failedWith(ResultCode::ACCESS_TOKEN_INVALID));
        }

        $payload = JwtService::parseToken($token);
        if (!$payload) {
            return json(Result::failedWith(ResultCode::ACCESS_TOKEN_INVALID));
        }

        $request->userId = $payload['sub'];
        $request->username = $payload['username'] ?? '';

        return $next($request);
    }
}
```

---

## Part 9: 异常处理

### 全局异常处理器

```php
<?php
declare(strict_types=1);

namespace app;

use app\common\exception\BusinessException;
use app\common\web\Result;
use app\common\web\ResultCode;
use think\db\exception\DataNotFoundException;
use think\db\exception\ModelNotFoundException;
use think\exception\Handle;
use think\exception\HttpException;
use think\exception\HttpResponseException;
use think\exception\ValidateException;
use think\Response;
use Throwable;

class ExceptionHandle extends Handle
{
    public function render($request, Throwable $e): Response
    {
        // 业务异常：返回统一错误响应
        if ($e instanceof BusinessException) {
            return json(Result::failedWith($e->getResultCode(), $e->getMessage()));
        }

        // 验证异常
        if ($e instanceof ValidateException) {
            return json(Result::failedWith(ResultCode::PARAM_ERROR, $e->getError()));
        }

        // HTTP 异常
        if ($e instanceof HttpException) {
            return json(Result::failedWith(ResultCode::fromValue($e->getStatusCode()), $e->getMessage()));
        }

        // 其他异常：返回系统错误（生产环境隐藏详情）
        return parent::render($request, $e);
    }
}
```

### 业务异常类

```php
<?php
declare(strict_types=1);

namespace app\common\exception;

use app\common\web\ResultCode;
use RuntimeException;

final class BusinessException extends RuntimeException
{
    private ResultCode $resultCode;

    public function __construct(ResultCode $resultCode, string $message = '')
    {
        $this->resultCode = $resultCode;
        parent::__construct($message ?: $resultCode->getMsg());
    }

    public function getResultCode(): ResultCode
    {
        return $this->resultCode;
    }
}
```

### 使用规范

```php
// 在 Service 层抛出业务异常
if (!$user) {
    throw new BusinessException(ResultCode::USER_NOT_EXIST);
}

// 自定义错误消息
throw new BusinessException(ResultCode::USER_ERROR, '用户名已存在');

// 在 Controller 层无需 try-catch，ExceptionHandle 统一捕获
```

### 配置异常处理器

```php
// config/app.php
return [
    'exception_handle' => \app\ExceptionHandle::class,
];
```

---

## Part 10: 验证器规范

```php
<?php
declare(strict_types=1);

namespace app\system\validate;

use think\Validate;

final class UserValidate extends Validate
{
    protected $rule = [
        'id' => 'require|integer',
        'username' => 'require|max:50',
        'password' => 'require|min:6|max:20',
        'nickname' => 'require|max:50',
        'mobile' => 'mobile',
        'email' => 'email',
        'status' => 'in:0,1',
    ];

    protected $message = [
        'username.require' => '用户名不能为空',
        'username.max' => '用户名最多50个字符',
        'password.require' => '密码不能为空',
        'password.min' => '密码至少6个字符',
        'nickname.require' => '昵称不能为空',
        'mobile.mobile' => '手机号格式错误',
        'email.email' => '邮箱格式错误',
    ];

    protected $scene = [
        'create' => ['username', 'password', 'nickname', 'mobile', 'email', 'status'],
        'update' => ['id', 'nickname', 'mobile', 'email', 'status'],
    ];
}
```

---

## Part 11: 代码质量检查清单

- [ ] 目录使用小写+下划线
- [ ] 类名采用驼峰法（首字母大写）
- [ ] 方法采用驼峰法（首字母小写）
- [ ] 数据表和字段采用小写+下划线
- [ ] 遵循 RESTful API 路径规范
- [ ] 使用统一响应格式（Result/ResultCode）
- [ ] 实现全局异常处理（ExceptionHandle）
- [ ] 业务异常使用 BusinessException + ResultCode
- [ ] 模型继承 BaseModel
- [ ] 使用软删除
- [ ] 分页接口使用 successPaginate
- [ ] 使用验证器校验参数
- [ ] 路由显式映射
- [ ] 中间件实现认证

---

## 参考资料

- [ThinkPHP 8.0 开发规范](https://doc.thinkphp.cn/v8_0/development_specifications.html)
- [ThinkPHP 8.0 目录结构](https://doc.thinkphp.cn/v8_0/directory_structure.html)
- [PSR-2 编码风格规范](https://www.php-fig.org/psr/psr-2/)
- [PSR-4 自动加载规范](https://www.php-fig.org/psr/psr-4/)
- [PHP 保留字列表](http://php.net/manual/zh/reserved.keywords.php)
