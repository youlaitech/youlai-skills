---
name: thinkphp
description: ThinkPHP 后端开发规范。当开发 ThinkPHP 项目、实现 REST API、模型数据访问、JWT 认证时使用此 skill。
---

# ThinkPHP 后端开发规范

> 参考：[ThinkPHP 8.0 开发规范](https://doc.thinkphp.cn/v8_0/development_specifications.html) | [ThinkPHP 8.0 目录结构](https://doc.thinkphp.cn/v8_0/directory_structure.html)

## 触发条件

- 开发 ThinkPHP 项目
- 实现 REST API
- 使用模型数据访问
- 实现 JWT 认证
- 实现权限控制

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
├─ BaseController.php          # 基础控制器类（官方推荐位置）
├─ ExceptionHandle.php         # 应用异常处理
├─ common.php                  # 全局公共函数
├─ middleware.php              # 全局中间件定义
├─ provider.php                # 服务提供定义
├─ event.php                   # 全局事件定义
│
├─ auth/                       # 认证模块（目录使用小写）
│  ├─ controller/              # 控制器目录
│  ├─ service/                 # 服务目录
│  └─ validate/                # 验证器目录
│
├─ system/                     # 系统模块（目录使用小写）
│  ├─ controller/              # 控制器目录
│  ├─ service/                 # 服务目录
│  ├─ model/                   # 模型目录
│  └─ validate/                # 验证器目录
│
├─ file/                       # 文件模块
├─ codegen/                    # 代码生成模块
│
├─ common/                     # 公共代码（非模块）
│  ├─ constants/               # 常量定义
│  ├─ enums/                   # 枚举类
│  ├─ exception/               # 异常类
│  ├─ redis/                   # Redis 工具
│  ├─ security/                # 认证/权限
│  ├─ util/                    # 工具类
│  └─ web/                     # Web 相关
│
├─ middleware/                 # 全局中间件
├─ traits/                     # 复用特性
└─ websocket/                  # WebSocket
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

## Part 9: 验证器规范

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

## Part 10: 代码质量检查清单

- [ ] 目录使用小写+下划线
- [ ] 类名采用驼峰法（首字母大写）
- [ ] 方法采用驼峰法（首字母小写）
- [ ] 数据表和字段采用小写+下划线
- [ ] 遵循 RESTful API 路径规范
- [ ] 使用统一响应格式（Result/ResultCode）
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
