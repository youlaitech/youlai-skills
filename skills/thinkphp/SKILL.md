---
name: thinkphp
description: ThinkPHP 后端开发规范。当开发 ThinkPHP 项目、实现 REST API、模型数据访问、JWT 认证时使用此 skill。
---

# ThinkPHP 后端开发规范

## 触发条件

- 开发 ThinkPHP 项目
- 实现 REST API
- 使用模型数据访问
- 实现 JWT 认证
- 实现权限控制

---

## Part 1: 技术栈

| 技术 | 版本 | 用途 |
|------|------|------|
| PHP | 8.2+ | 运行环境 |
| ThinkPHP | 8.0+ | Web 框架 |
| MySQL | 8.0+ | 数据库 |
| Redis | 7.0+ | 缓存 |
| firebase/php-jwt | 6.0+ | Token 认证 |

---

## Part 2: 目录结构

```
app/
├── controller/                # 控制器
│   ├── system/                # 系统模块
│   │   ├── User.php
│   │   ├── Role.php
│   │   └── Menu.php
│   └── auth/                  # 认证模块
│       └── Auth.php
├── model/                     # 模型
│   ├── system/
│   │   ├── User.php
│   │   └── Role.php
│   └── BaseModel.php
├── validate/                  # 验证器
│   └── system/
│       └── UserValidate.php
├── middleware/                # 中间件
│   ├── Auth.php
│   ├── Permission.php
│   └── Cors.php
├── common/                    # 公共模块
│   ├── enum/
│   │   └── ResultCode.php
│   ├── trait/
│   │   └── ResponseTrait.php
│   └── service/
│       ├── JwtService.php
│       └── AuthService.php
├── config/                    # 配置
│   ├── database.php
│   ├── cache.php
│   └── jwt.php
└── route/                     # 路由
    └── api.php
```

---

## Part 3: 命名规范

### 文件命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 控制器 | PascalCase | `User.php` |
| 模型 | PascalCase | `User.php` |
| 验证器 | PascalCase + Validate | `UserValidate.php` |
| 中间件 | PascalCase | `Auth.php` |
| 配置 | snake_case | `database.php` |

### 类命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 控制器 | PascalCase | `User` |
| 模型 | PascalCase | `User` 或 `SysUser` |
| 验证器 | 功能 + Validate | `UserValidate` |
| 中间件 | PascalCase | `Auth` |

### 方法命名

| 动作 | 前缀 | 示例 |
|------|------|------|
| 查询单个 | get | `getById()` |
| 查询列表 | list | `listUsers()` |
| 分页查询 | page | `pageUsers()` |
| 新增 | create | `createUser()` |
| 更新 | update | `updateUser()` |
| 删除 | delete | `deleteUser()` |

### 变量命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 变量 | camelCase | `$userList` |
| 常量 | UPPER_SNAKE_CASE | `MAX_SIZE` |
| 私有变量 | _ 前缀 | `$_internalState` |
| 布尔值 | is/has/can 前缀 | `$isDeleted`, `$hasPermission` |

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

```php
<?php
namespace app\controller\system;

use app\BaseController;
use app\model\system\User;
use app\validate\system\UserValidate;
use app\common\trait\ResponseTrait;
use think\facade\Request;

class User extends BaseController
{
    use ResponseTrait;

    /**
     * 用户分页列表
     */
    public function page()
    {
        $params = Request::get();
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

        return $this->pageResult($result->items(), $result->total());
    }

    /**
     * 用户详情
     */
    public function detail($id)
    {
        $user = User::find($id);
        if (!$user) {
            return $this->error('用户不存在');
        }
        return $this->success($user->toArray());
    }

    /**
     * 新增用户
     */
    public function create()
    {
        $params = Request::post();
        validate(UserValidate::class)->scene('create')->check($params);

        // 检查用户名是否存在
        if (User::where('username', $params['username'])->find()) {
            return $this->error('用户名已存在');
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
        $params = Request::put();
        validate(UserValidate::class)->scene('update')->check($params);

        $user = User::find($params['id']);
        if (!$user) {
            return $this->error('用户不存在');
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
            return $this->error('用户不存在');
        }
        $user->delete();
        return $this->success();
    }

    /**
     * 批量删除
     */
    public function batchDelete()
    {
        $ids = Request::delete('ids');
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

```php
<?php
namespace app\common\trait;

trait ResponseTrait
{
    /**
     * 成功响应
     */
    protected function success($data = null, string $msg = '操作成功')
    {
        return json([
            'code' => 200,
            'msg' => $msg,
            'data' => $data,
        ]);
    }

    /**
     * 分页响应
     */
    protected function pageResult($list, int $total, string $msg = '操作成功')
    {
        return json([
            'code' => 200,
            'msg' => $msg,
            'data' => [
                'list' => $list,
                'total' => $total,
            ],
        ]);
    }

    /**
     * 错误响应
     */
    protected function error(string $msg = '操作失败', int $code = 400)
    {
        return json([
            'code' => $code,
            'msg' => $msg,
            'data' => null,
        ]);
    }
}
```

---

## Part 6: 模型规范

### 基础模型

```php
<?php
namespace app\model;

use think\Model;

class BaseModel extends Model
{
    protected $autoWriteTimestamp = true;
    protected $createTime = 'create_time';
    protected $updateTime = 'update_time';

    // 软删除
    use \think\model\concern\SoftDelete;
    protected $deleteTime = 'delete_time';
    protected $defaultSoftDelete = 0;

    // 全局作用域：自动填充创建人/更新人
    public static function onBeforeInsert($model)
    {
        $model->create_by = request()->userId ?? 0;
        $model->update_by = request()->userId ?? 0;
    }

    public static function onBeforeUpdate($model)
    {
        $model->update_by = request()->userId ?? 0;
    }
}
```

### 模型示例

```php
<?php
namespace app\model\system;

use app\model\BaseModel;

class User extends BaseModel
{
    protected $name = 'sys_user';
    protected $pk = 'id';

    // 隐藏字段
    protected $hidden = ['password', 'delete_time'];

    // 类型转换
    protected $type = [
        'id' => 'integer',
        'status' => 'integer',
        'create_time' => 'datetime',
        'update_time' => 'datetime',
    ];

    // 关联部门
    public function dept()
    {
        return $this->belongsTo(Dept::class, 'dept_id');
    }

    // 关联角色
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
    Route::post('login', 'auth.Auth/login');
    Route::post('logout', 'auth.Auth/logout');
    Route::post('refresh-token', 'auth.Auth/refreshToken');
})->allowCrossDomain();

// 需要登录的接口
Route::group('api/v1', function () {
    // 用户管理
    Route::group('users', function () {
        Route::get('page', 'system.User/page');
        Route::get(':id', 'system.User/detail');
        Route::get('options', 'system.User/options');
        Route::post('', 'system.User/create');
        Route::put('', 'system.User/update');
        Route::delete(':id', 'system.User/delete');
        Route::delete('batch', 'system.User/batchDelete');
    });

    // 角色管理
    Route::group('roles', function () {
        Route::get('page', 'system.Role/page');
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
namespace app\common\service;

use Firebase\JWT\JWT;
use Firebase\JWT\Key;
use think\facade\Config;

class JwtService
{
    /**
     * 生成 Access Token
     */
    public static function generateAccessToken(array $payload): string
    {
        $payload['exp'] = time() + Config::get('jwt.access_ttl', 7200);
        $payload['iat'] = time();
        $payload['iss'] = Config::get('jwt.issuer', 'youlai-think');
        
        return JWT::encode($payload, Config::get('jwt.secret'), 'HS256');
    }

    /**
     * 解析 Token
     */
    public static function parseToken(string $token): ?array
    {
        try {
            $decoded = JWT::decode($token, new Key(Config::get('jwt.secret'), 'HS256'));
            return (array) $decoded;
        } catch (\Exception $e) {
            return null;
        }
    }

    /**
     * 生成 Refresh Token
     */
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
namespace app\middleware;

use app\common\service\JwtService;
use Closure;
use think\Request;
use think\Response;

class Auth
{
    public function handle(Request $request, Closure $next): Response
    {
        $token = $request->header('Authorization', '');
        $token = str_replace('Bearer ', '', $token);

        if (empty($token)) {
            return json([
                'code' => 401,
                'msg' => '未登录或token过期',
                'data' => null,
            ]);
        }

        $payload = JwtService::parseToken($token);
        if (!$payload) {
            return json([
                'code' => 401,
                'msg' => '未登录或token过期',
                'data' => null,
            ]);
        }

        // 注入用户信息到请求
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
namespace app\validate\system;

use think\Validate;

class UserValidate extends Validate
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

- [ ] 遵循 RESTful API 路径规范
- [ ] 使用统一响应格式
- [ ] 模型继承 BaseModel
- [ ] 使用软删除
- [ ] 分页接口使用 pageResult
- [ ] 使用验证器校验参数
- [ ] 路由显式映射
- [ ] 中间件实现认证
- [ ] 配置文件管理配置
- [ ] 使用 trait 复用代码
