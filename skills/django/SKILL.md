---
name: django
description: Django REST Framework 后端开发规范。当开发 Django 项目、实现 REST API、DRF ViewSet、Serializer、JWT 认证时使用此 skill。
---

# Django REST Framework 后端开发规范

## 触发条件

- 开发 Django 项目
- 实现 REST API
- 使用 DRF ViewSet
- 实现 JWT 认证
- 实现 Serializer

---

## Part 1: 技术栈

- **Python** - 运行环境
- **Django** - Web 框架
- **DRF** - REST 框架
- **MySQL** - 数据库
- **Redis** - 缓存
- **PyJWT** - Token 认证
- **drf-spectacular** - API 文档

---

## Part 2: 目录结构

```
project/
├── apps/                       # 应用模块
│   ├── system/                 # 系统模块
│   │   ├── models.py           # 模型
│   │   ├── serializers.py      # 序列化器
│   │   ├── views.py            # 视图（ViewSet）
│   │   ├── urls.py             # 路由
│   │   ├── filters.py          # 过滤器
│   │   └── admin.py            # Admin 配置
│   └── auth/                    # 认证模块
│       ├── views.py
│       ├── serializers.py
│       └── urls.py
├── config/                     # 配置
│   ├── settings/
│   │   ├── base.py
│   │   ├── development.py
│   │   └── production.py
│   ├── urls.py                 # 根路由
│   └── wsgi.py
├── common/                     # 公共模块
│   ├── response.py             # 响应封装
│   ├── exceptions.py           # 异常处理
│   ├── pagination.py           # 分页
│   ├── permissions.py          # 权限
│   └── middleware.py           # 中间件
├── utils/                      # 工具类
│   ├── jwt_util.py
│   └── bcrypt_util.py
├── manage.py
└── requirements.txt
```

---

## Part 3: 命名规范

### 文件命名

| 类型     | 规范 | 示例             |
| -------- | ---- | ---------------- |
| 模块     | 小写 | `system/`        |
| 模型     | 小写 | `models.py`      |
| 序列化器 | 小写 | `serializers.py` |
| 视图     | 小写 | `views.py`       |
| 路由     | 小写 | `urls.py`        |

### 类命名

| 类型       | 规范              | 示例                                   |
| ---------- | ----------------- | -------------------------------------- |
| 模型       | PascalCase        | `User` 或 `SysUser`                    |
| Serializer | 功能 + Serializer | `UserSerializer`, `UserPageSerializer` |
| ViewSet    | 功能 + ViewSet    | `UserViewSet`                          |
| Filter     | 功能 + Filter     | `UserFilter`                           |

### 方法命名

| 动作     | 前缀    | 示例           |
| -------- | ------- | -------------- |
| 查询单个 | get     | `get_object()` |
| 查询列表 | list    | `list()`       |
| 新增     | create  | `create()`     |
| 更新     | update  | `update()`     |
| 删除     | destroy | `destroy()`    |

### 变量命名

| 类型     | 规范             | 示例                           |
| -------- | ---------------- | ------------------------------ |
| 变量     | snake_case       | `user_list`                    |
| 常量     | UPPER_SNAKE_CASE | `MAX_SIZE`                     |
| 私有变量 | \_ 前缀          | `_internal_state`              |
| 布尔值   | is/has/can 前缀  | `is_deleted`, `has_permission` |

---

## Part 4: RESTful API 规范

### 标准 CRUD 路径

| 操作     | 方法   | 路径                     |
| -------- | ------ | ------------------------ |
| 分页列表 | GET    | `/api/v1/users/page/`    |
| 详情     | GET    | `/api/v1/users/{id}/`    |
| 新增     | POST   | `/api/v1/users/`         |
| 更新     | PUT    | `/api/v1/users/{id}/`    |
| 删除     | DELETE | `/api/v1/users/{id}/`    |
| 批量删除 | DELETE | `/api/v1/users/batch/`   |
| 下拉选项 | GET    | `/api/v1/users/options/` |

### ViewSet 模板

```python
# apps/system/views.py
from rest_framework.viewsets import ModelViewSet
from rest_framework.decorators import action
from rest_framework.response import Response
from drf_spectacular.utils import extend_schema, OpenApiResponse

from common.response import success_result, page_result
from common.permissions import HasPermission
from .models import SysUser
from .serializers import UserSerializer, UserPageSerializer, UserFormSerializer
from .filters import UserFilter


@extend_schema(tags=["用户管理"])
class UserViewSet(ModelViewSet):
    queryset = SysUser.objects.all()
    serializer_class = UserSerializer
    filterset_class = UserFilter
    permission_classes = [HasPermission]

    @extend_schema(
        summary="用户分页列表",
        responses={200: OpenApiResponse(response=UserPageSerializer)}
    )
    @action(detail=False, methods=["get"])
    def page(self, request):
        queryset = self.filter_queryset(self.get_queryset())
        page = self.paginate_queryset(queryset)
        serializer = self.get_serializer(page, many=True)
        return page_result(serializer.data, self.paginator.count)

    @extend_schema(summary="用户详情")
    def retrieve(self, request, *args, **kwargs):
        instance = self.get_object()
        serializer = self.get_serializer(instance)
        return success_result(serializer.data)

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
        instance = self.get_object()
        instance.delete()
        return success_result()

    @extend_schema(summary="批量删除用户")
    @action(detail=False, methods=["delete"])
    def batch(self, request):
        ids = request.data.get("ids", [])
        SysUser.objects.filter(id__in=ids).delete()
        return success_result()

    @extend_schema(summary="用户选项列表")
    @action(detail=False, methods=["get"])
    def options(self, request):
        users = SysUser.objects.values("id", "nickname")
        return success_result(list(users))
```

---

## Part 5: 响应格式

```python
# common/response.py
from rest_framework.response import Response


def success_result(data=None, msg="操作成功"):
    return Response({
        "code": 200,
        "msg": msg,
        "data": data
    })


def page_result(list_data, total, msg="操作成功"):
    return Response({
        "code": 200,
        "msg": msg,
        "data": {
            "list": list_data,
            "total": total
        }
    })


def error_result(code=400, msg="操作失败", data=None):
    return Response({
        "code": code,
        "msg": msg,
        "data": data
    })
```

### 异常处理

```python
# common/exceptions.py
from rest_framework.exceptions import APIException
from rest_framework import status


class BusinessException(APIException):
    status_code = status.HTTP_200_OK
    default_detail = "业务异常"
    default_code = 400

    def __init__(self, detail=None, code=400):
        self.detail = detail or self.default_detail
        self.code = code


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

### 基础模型

```python
# common/models.py
from django.db import models


class BaseModel(models.Model):
    id = models.BigAutoField(primary_key=True)
    create_time = models.DateTimeField(auto_now_add=True, verbose_name="创建时间")
    update_time = models.DateTimeField(auto_now=True, verbose_name="更新时间")
    create_by = models.BigIntegerField(null=True, blank=True, verbose_name="创建人")
    update_by = models.BigIntegerField(null=True, blank=True, verbose_name="更新人")
    is_deleted = models.BooleanField(default=False, verbose_name="是否删除")

    class Meta:
        abstract = True

    def delete(self, using=None, keep_parents=False):
        self.is_deleted = True
        self.save()
```

### 模型示例

```python
# apps/system/models.py
from django.db import models
from common.models import BaseModel


class SysUser(BaseModel):
    username = models.CharField(max_length=50, unique=True, verbose_name="用户名")
    password = models.CharField(max_length=100, verbose_name="密码")
    nickname = models.CharField(max_length=50, verbose_name="昵称")
    mobile = models.CharField(max_length=20, blank=True, verbose_name="手机号")
    email = models.EmailField(max_length=100, blank=True, verbose_name="邮箱")
    gender = models.SmallIntegerField(default=0, verbose_name="性别")
    status = models.SmallIntegerField(default=1, verbose_name="状态")
    dept = models.ForeignKey("SysDept", on_delete=models.SET_NULL, null=True, verbose_name="部门")

    class Meta:
        db_table = "sys_user"
        verbose_name = "用户"

    def __str__(self):
        return self.username
```

---

## Part 7: Serializer 规范

```python
# apps/system/serializers.py
from rest_framework import serializers
from .models import SysUser


class UserSerializer(serializers.ModelSerializer):
    """用户VO序列化器"""
    class Meta:
        model = SysUser
        fields = ["id", "username", "nickname", "mobile", "email", "status", "create_time"]


class UserFormSerializer(serializers.ModelSerializer):
    """用户表单序列化器"""
    class Meta:
        model = SysUser
        fields = ["id", "username", "password", "nickname", "mobile", "email", "status", "dept"]

    def validate_username(self, value):
        if SysUser.objects.filter(username=value, is_deleted=False).exclude(id=self.instance.id if self.instance else None).exists():
            raise serializers.ValidationError("用户名已存在")
        return value

    def create(self, validated_data):
        validated_data["password"] = bcrypt.hashpw(validated_data["password"])
        return super().create(validated_data)


class UserPageSerializer(serializers.ModelSerializer):
    """用户分页序列化器"""
    dept_name = serializers.CharField(source="dept.name", read_only=True)

    class Meta:
        model = SysUser
        fields = ["id", "username", "nickname", "mobile", "email", "status", "dept_name", "create_time"]
```

---

## Part 8: 路由规范

```python
# apps/system/urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import UserViewSet, RoleViewSet, MenuViewSet

router = DefaultRouter()
router.register(r"users", UserViewSet, basename="user")
router.register(r"roles", RoleViewSet, basename="role")
router.register(r"menus", MenuViewSet, basename="menu")

urlpatterns = [
    path("", include(router.urls)),
]

# config/urls.py
from django.urls import path, include

urlpatterns = [
    path("api/v1/auth/", include("apps.auth.urls")),
    path("api/v1/system/", include("apps.system.urls")),
]
```

---

## Part 9: 认证规范

### JWT 认证

```python
# common/authentication.py
import jwt
from rest_framework.authentication import BaseAuthentication
from rest_framework.exceptions import AuthenticationFailed
from django.conf import settings


class JWTAuthentication(BaseAuthentication):
    def authenticate(self, request):
        token = request.META.get("HTTP_AUTHORIZATION", "").replace("Bearer ", "")
        if not token:
            return None

        try:
            payload = jwt.decode(token, settings.JWT_SECRET, algorithms=["HS256"])
        except jwt.ExpiredSignatureError:
            raise AuthenticationFailed("Token已过期")
        except jwt.InvalidTokenError:
            raise AuthenticationFailed("无效Token")

        user = self.get_user(payload["sub"])
        return (user, token)

    def get_user(self, user_id):
        from apps.system.models import SysUser
        try:
            return SysUser.objects.get(id=user_id, is_deleted=False)
        except SysUser.DoesNotExist:
            raise AuthenticationFailed("用户不存在")
```

### 权限类

```python
# common/permissions.py
from rest_framework.permissions import BasePermission


class HasPermission(BasePermission):
    def has_permission(self, request, view):
        if not request.user:
            return False

        required = getattr(view, "required_permissions", [])
        if not required:
            return True

        user_permissions = request.user.permissions
        return any(p in user_permissions for p in required)
```

---

## Part 10: 代码质量检查清单

- [ ] 遵循 RESTful API 路径规范
- [ ] 使用统一响应格式
- [ ] ViewSet + Serializer 模式
- [ ] 添加权限检查
- [ ] 分页接口使用 page_result
- [ ] 模型继承 BaseModel
- [ ] 使用逻辑删除
- [ ] API 添加 @extend_schema 注解
- [ ] 显式 urls 映射
- [ ] Filter 类实现过滤
