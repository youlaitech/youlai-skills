---
name: django
description: Django REST Framework backend development standards. Use this skill when developing Django projects, implementing REST APIs with DRF ViewSets, Serializers, or JWT authentication.
---

# Django REST Framework 后端开发规范

## 触发条件

- Develop Django projects
- Implement REST APIs
- Use DRF ViewSets and Serializers
- Implement JWT authentication
- Implement permission control

---

## Part 1: 技术栈

| 层 | 选型 | 说明 |
|----|------|------|
| 运行环境 | **Python 3.11+** | 类型注解增强 |
| Web 框架 | **Django 4.2+** | 全栈框架、ORM 内置 |
| REST API | **Django REST Framework (DRF)** | ViewSet、Serializer、认证 |
| 数据库 | **MySQL 8.x** (mysqlclient) | InnoDB 引擎 |
| 缓存 | **Redis 7.x** (django-redis) | 分布式缓存 |
| 认证 | **PyJWT** + DRF Authentication | Token 签发/验签 |
| API 文档 | **drf-spectacular** | OpenAPI 3.0 自动生成 |

---

## Part 2: 目录结构

```
project/
├── apps/                           # 应用模块
│   ├── system/                     # 系统管理（Django App）
│   │   ├── users/                  # 用户管理（models, serializers, views, urls, filters）
│   │   ├── roles/                  # 角色管理
│   │   ├── menus/                  # 菜单管理
│   │   ├── dept/                   # 部门管理
│   │   ├── dicts/                  # 字典管理
│   │   ├── configs/                # 系统配置
│   │   ├── notices/                # 通知公告
│   │   ├── logs/                   # 日志管理
│   │   └── utils/                  # decorators, exception_handler, rate_limit
│   ├── auth/                       # 认证模块
│   │   ├── views.py, serializers.py, urls.py
│   │   ├── utils/                  # jwt_authentication, redis_token_authentication
│   │   └── models/                 # user_session, user_social
│   ├── codegen/                    # 代码生成器（models, views, urls, templates/）
│   ├── file/                       # 文件管理
│   └── message/                    # 消息模块（SSE）
├── core/                           # 公共模块
│   ├── response.py                 # 统一响应 + 分页
│   ├── viewsets.py                 # ViewSet 基类
│   ├── serializers.py              # Serializer 基类
│   ├── exceptions/                 # BusinessException + 全局处理器
│   ├── permissions/                # 接口权限 + 数据权限
│   └── middleware/                 # rate_limit, request_context
├── config/                         # 项目配置
│   ├── settings/{base,dev,prod}.py
│   ├── urls.py                     # 根路由
│   └── env.py
├── sql/mysql/youlai_admin_django.sql
├── manage.py
└── requirements.txt
```

**设计原则**：
- `core/` 是公共基础设施，被所有 App 共享，不依赖业务 App
- `apps/` 中每 App 按实体分子模块（user/role/menu），每子模块含 models/serializers/views/urls/filters
- ViewSet + Serializer 是标准模式，复杂业务逻辑抽到 `services.py`

---

## Part 3: 命名规范

### 3.1 文件命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 模块目录 | 小写复数 | `users/`, `roles/`, `menus/` |
| 模型 | `models.py` | `apps/system/users/models.py` |
| Serializer | `serializers.py` | 同上 |
| 视图 | `views.py` | 同上 |
| 路由 | `urls.py` | 同上 |
| 过滤器 | `filters.py` | 同上 |

### 3.2 类命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 模型 | PascalCase，Sys 前缀 | `SysUser`, `SysRole`, `SysMenu` |
| Serializer | 功能 + Serializer | `UserSerializer`, `UserFormSerializer` |
| ViewSet | 功能 + ViewSet | `UserViewSet` |
| Filter | 功能 + Filter | `UserFilter` |

### 3.3 方法命名

| 动作 | DRF 内置方法 | 自定义 action |
|------|-------------|---------------|
| 查询列表 | `list()` | `page()` |
| 查询详情 | `retrieve()` | — |
| 新增 | `create()` | — |
| 更新 | `update()` | — |
| 删除 | `destroy()` | — |
| 批量删除 | — | `batch()` |
| 下拉选项 | — | `options()` |

### 3.4 变量命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 变量 | snake_case | `user_list` |
| 常量 | UPPER_SNAKE_CASE | `MAX_PAGE_SIZE` |
| 私有方法 | `_` 前缀 | `_parse_date()` |
| 布尔值 | is_/has_ 前缀 | `is_deleted`, `has_permission` |

---

## Part 4: RESTful API 规范

### 4.1 标准 CRUD 路径

| 操作 | 方法 | 路径 |
|------|------|------|
| 分页列表 | `GET` | `/api/v1/users/page/` |
| 详情 | `GET` | `/api/v1/users/{id}/` |
| 新增 | `POST` | `/api/v1/users/` |
| 更新 | `PUT` | `/api/v1/users/{id}/` |
| 删除 | `DELETE` | `/api/v1/users/{id}/` |
| 批量删除 | `DELETE` | `/api/v1/users/batch/` |
| 下拉选项 | `GET` | `/api/v1/users/options/` |

### 4.2 ViewSet 模板

```python
@extend_schema(tags=["用户管理"])
class UserViewSet(ModelViewSet):
    queryset = SysUser.objects.all()
    serializer_class = UserSerializer
    filterset_class = UserFilter
    permission_classes = [HasPermission]

    @extend_schema(summary="用户分页列表")
    @action(detail=False, methods=["get"])
    def page(self, request):
        queryset = self.filter_queryset(self.get_queryset())
        page = self.paginate_queryset(queryset)
        return page_result(self.get_serializer(page, many=True).data, self.paginator.count)

    @extend_schema(summary="新增用户")
    def create(self, request, *args, **kwargs):
        serializer = UserFormSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        user = serializer.save()
        return success_result({"id": user.id})

    @extend_schema(summary="更新用户")
    def update(self, request, *args, **kwargs):
        instance = self.get_object()
        serializer = UserFormSerializer(instance, data=request.data)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return success_result()

    @extend_schema(summary="删除用户")
    def destroy(self, request, *args, **kwargs):
        self.get_object().delete()
        return success_result()
```

---

## Part 5: 响应格式与异常处理

### 5.1 统一响应

```python
def success_result(data=None, msg="操作成功"):
    return Response({"code": 200, "msg": msg, "data": data})

def page_result(list_data, total, msg="操作成功"):
    return Response({"code": 200, "msg": msg, "data": {"list": list_data, "total": total}})

def error_result(code=400, msg="操作失败", data=None):
    return Response({"code": code, "msg": msg, "data": data})
```

### 5.2 业务异常

```python
from rest_framework.exceptions import APIException

class BusinessException(APIException):
    status_code = 200  # 统一 200，业务码在 body 中
    default_detail = "业务异常"
    default_code = 400

    def __init__(self, detail=None, code=400):
        self.detail = detail or self.default_detail
        self.code = code
```

### 5.3 全局异常处理

```python
class GlobalExceptionHandler:
    def __call__(self, exc, context):
        if isinstance(exc, BusinessException):
            return error_result(code=exc.code, msg=str(exc.detail))
        if isinstance(exc, APIException):
            return error_result(code=exc.status_code, msg=str(exc.detail))
        return error_result(code=500, msg="系统异常")
```

---

## Part 6: 模型规范

```python
class BaseModel(models.Model):
    id = models.BigAutoField(primary_key=True)
    create_time = models.DateTimeField(auto_now_add=True, verbose_name="创建时间")
    update_time = models.DateTimeField(auto_now=True, verbose_name="更新时间")
    is_deleted = models.BooleanField(default=False, verbose_name="是否删除")

    class Meta:
        abstract = True

    def delete(self, *args, **kwargs):
        """逻辑删除。"""
        self.is_deleted = True
        self.save()

class SysUser(BaseModel):
    username = models.CharField(max_length=50, unique=True, verbose_name="用户名")
    password = models.CharField(max_length=100, verbose_name="密码")
    nickname = models.CharField(max_length=50, verbose_name="昵称")
    mobile = models.CharField(max_length=20, blank=True, verbose_name="手机号")
    status = models.SmallIntegerField(default=1, verbose_name="状态")
    dept = models.ForeignKey("SysDept", on_delete=models.SET_NULL, null=True, verbose_name="部门")

    class Meta:
        db_table = "sys_user"
```

---

## Part 7: Serializer 规范

```python
class UserSerializer(serializers.ModelSerializer):
    """用户 VO 序列化器。"""
    class Meta:
        model = SysUser
        fields = ["id", "username", "nickname", "mobile", "email", "status", "create_time"]

class UserFormSerializer(serializers.ModelSerializer):
    """用户表单序列化器。"""
    class Meta:
        model = SysUser
        fields = ["id", "username", "password", "nickname", "mobile", "email", "status", "dept"]

    def validate_username(self, value):
        if SysUser.objects.filter(username=value, is_deleted=False).exclude(
            id=self.instance.id if self.instance else None).exists():
            raise serializers.ValidationError("用户名已存在")
        return value

    def create(self, validated_data):
        validated_data["password"] = bcrypt.hashpw(validated_data["password"])
        return super().create(validated_data)
```

---

## Part 8: 认证规范

### 8.1 JWT 认证

```python
class JWTAuthentication(BaseAuthentication):
    def authenticate(self, request):
        token = request.META.get("HTTP_AUTHORIZATION", "").replace("Bearer ", "")
        if not token:
            return None
        try:
            payload = jwt.decode(token, settings.JWT_SECRET, algorithms=["HS256"])
        except jwt.ExpiredSignatureError:
            raise AuthenticationFailed("Token 已过期")
        except jwt.InvalidTokenError:
            raise AuthenticationFailed("无效 Token")
        user = SysUser.objects.get(id=payload["sub"], is_deleted=False)
        return (user, token)
```

### 8.2 权限类

```python
class HasPermission(BasePermission):
    def has_permission(self, request, view):
        required = getattr(view, "required_permissions", [])
        if not required:
            return True
        return any(p in request.user.permissions for p in required)
```

---

## Part 9: 注释规范

- **docstring 优先**：所有公共类、方法必须有 docstring
- **中文优先**：专有名词保留英文，解释用中文

```python
class UserViewSet(ModelViewSet):
    """用户管理 ViewSet。

    提供用户分页查询、增删改查、下拉选项等接口。
    所有接口（除 options 外）需认证。
    """

    @action(detail=False, methods=["get"])
    def page(self, request):
        """用户分页列表。

        Args:
            request: DRF Request 对象，Query 参数含 pageNum/pageSize/keywords/status。

        Returns:
            Response: {"code": 200, "data": {"list": [...], "total": int}}
        """
        ...
```

---

## Part 10: 代码质量检查清单

- [ ] 遵循 RESTful API 路径规范
- [ ] 使用统一响应格式（`success_result` / `page_result`）
- [ ] ViewSet + Serializer 标准模式
- [ ] 权限检查使用 `HasPermission`
- [ ] 模型继承 `BaseModel`
- [ ] 使用逻辑删除（`is_deleted`）
- [ ] API 添加 `@extend_schema` 注解
- [ ] Serializer 校验不可变操作
- [ ] 复杂查询使用 `Filter` 类
- [ ] 公共方法有 docstring

### 常见反模式

| 反模式 | 正确做法 |
|--------|----------|
| `return Response(data)` 直接返回 | 用 `success_result(data)` 统一封装 |
| `try/except Exception: pass` | 抛出 `BusinessException` 或记录日志 |
| `User.objects.get(id=id)` 无异常处理 | 捕获 `DoesNotExist` → 抛出 `BusinessException` |
| 在 View 中写复杂业务逻辑 | 抽到 Model Manager 或 services.py |
| `request.data["key"]` 无校验 | 走 Serializer `is_valid(raise_exception=True)` |
