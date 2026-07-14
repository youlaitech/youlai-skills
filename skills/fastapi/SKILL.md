---
name: fastapi
description: FastAPI 后端开发规范. Use this skill when developing FastAPI projects, implementing REST APIs with SQLAlchemy async, Pydantic v2 schemas, JWT/Redis token authentication, or the youlai-fastapi admin backend.
---

# FastAPI 后端开发规范

## 触发条件

- Develop FastAPI projects
- Implement async REST APIs
- Use SQLAlchemy 2.0 async ORM
- Use Pydantic v2 for data validation
- Implement JWT / Redis Token authentication
- Implement role-based permission control
- Develop youlai-fastapi admin backend modules

---

## Part 1: 技术栈

| 层 | 选型 | 说明 |
|---|------|------|
| Web 框架 | **FastAPI** 0.115+ | 异步原生、Pydantic v2 集成、自动 OpenAPI 文档 |
| ASGI 服务器 | **Uvicorn** 0.30+ | FastAPI 官方推荐 |
| ORM | **SQLAlchemy 2.0** + **asyncpg** | `Mapped` + `mapped_column` 声明式、完整 async 支持 |
| 数据验证 | **Pydantic v2** | Rust 核心、FastAPI 原生集成 |
| 数据库 | **PostgreSQL** 16+ | 生产级、asyncpg 异步驱动 |
| 缓存 | **Redis** 7.x | `redis[hiredis]` 异步客户端 |
| 认证 | **PyJWT** 2.x + **bcrypt** 4.x | JWT 签发/验签、密码哈希 |
| 日志 | **loguru** 0.7+ | 零配置、彩色输出、自动 rotate |
| 包管理 | **uv** | Rust 编写、10-100x pip 速度 |
| 代码质量 | **ruff** + **mypy** | Lint + 格式化 + 静态类型检查 |
| 测试 | **pytest** + **httpx** | httpx AsyncClient 直测 FastAPI |

---

## Part 2: 目录结构

> 目录结构以社区标杆 [zhanymkanov/fastapi-best-practices](https://github.com/zhanymkanov/fastapi-best-practices) 为准。

```
youlai-fastapi/
├── app/
│   ├── __init__.py
│   ├── main.py                 # FastAPI 入口：挂载路由、中间件、异常处理器
│   ├── config.py               # Pydantic Settings
│   ├── database.py             # 异步引擎 + session + get_db
│   ├── redis.py                # Redis 连接池 + get_redis
│   ├── response.py             # Result[T] + ResultCode 统一响应
│   ├── exceptions.py           # BusinessException + 全局异常处理器
│   ├── enums.py                 # 全局枚举
│   ├── pagination.py           # PageQuery / PageResult
│   ├── constants.py            # 全局常量：安全前缀 / REDIS_ 前缀 / 系统级
│   ├── dependencies.py         # 全局依赖 get_current_user / require_perm
│   ├── middleware.py           # CORS + 请求日志 + 限流
│   ├── registry.py             # 导入全部域模型，触发 Base.metadata 注册（Base/Mixin 在 database.py）
│   ├── auth/                   # 认证域：router.py / schemas.py / service.py / token.py / utils.py
│   ├── captcha/                # 验证码：service.py / constants.py
│   ├── system/                 # 系统管理域
│   │   ├── user/  role/  menu/  dept/  dict/  log/
│   │   │   └── 各含 router.py  schemas.py  service.py  models.py
│   │   │       （role/ 含 constants.py、data_permission.py；log/ 含 constants.py、operation_log.py）
│   │   ├── config/             # 仅 models.py + router.py（配置内联，无 schemas/service）
│   │   └── notice/             # 仅 models.py + router.py
│   └── tool/                   # 工具与集成域
│       └── file/  codegen/  wxma/  sse/   # 文件 / 代码生成 / 微信小程序 / 服务端推送
├── alembic/                    # 数据库迁移
├── alembic.ini
├── tests/
│   ├── conftest.py
│   ├── auth/test_auth.py       # 按业务域镜像
│   └── test_result_code.py     # 全局
├── Dockerfile
├── docker-compose.yml
├── pyproject.toml              # uv 依赖管理
├── .env.example
├── .gitignore
└── README.md
```

**设计原则**：
- 按业务域组织：每个域目录自包含 `router/schemas/models/service`（按需含 `constants/dependencies/exceptions`），域间通过显式模块名导入。
- `models/` 仅放 ORM 基类与全域注册，纯 ORM 不依赖任何业务层。
- 全局基础设施（config / database / redis / response / exceptions / pagination / constants / dependencies / middleware）平铺在 `app/` 根，被各域共享。
- 不设 `core/` `common/` `framework/` `modules/` 分层包。

---

## Part 3: 命名规范

### 3.1 文件命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 模块目录 | 小写单数名词 | `user/`, `role/`, `menu/` |
| Schema 文件 | `schemas.py` | `app/system/user/schemas.py` |
| Service 文件 | `service.py` | `app/auth/service.py` |
| Router 文件 | `router.py` | `app/system/role/router.py` |
| 配置文件 | `config.py` | `app/config.py` |
| 模型文件 | 按域拆分 | `app/system/user/models.py` |
| 常量文件 | `constants.py` | `app/constants.py`（全局）/ `app/system/user/constants.py`（域） |
| 枚举文件 | `enums.py` / `constants.py` | `app/enums.py`（全局）/ `app/system/role/constants.py`（域枚举） |
| 测试文件 | `test_<模块>.py` | `tests/auth/test_auth.py` |

### 3.2 类命名

| 类型 | 规范 | 示例 |
|------|------|------|
| ORM 模型 | `Sys` + PascalCase | `SysUser`, `SysRole`, `SysMenu`, `SysDept` |
| Pydantic Schema | 功能 + 后缀 | `UserCreate`, `UserUpdate`, `UserVO`, `UserQuery` |
| Service 类 | 模块名 + `Service` | `UserService`, `AuthService`, `RoleService` |
| Config 类 | `Settings` | `Settings`（单例） |
| Enum 类 | 功能名 + `Enum` | `StatusEnum`, `DataScopeEnum` |
| Exception | `BusinessException` | 统一使用 `BusinessException` |

**Schema 后缀规范**：

| 后缀 | 用途 | 示例 |
|------|------|------|
| `Create` | 创建表单 | `UserCreate`, `RoleCreate` |
| `Update` | 更新表单 | `UserUpdate`, `MenuUpdate` |
| `VO` | 视图对象（返回） | `UserVO`, `RoleVO`, `DictVO` |
| `Query` | 分页查询参数 | `UserQuery`, `RoleQuery`, `LogQuery` |
| `Form` | 特定操作表单 | `UserPasswordForm`, `RoleMenuForm` |
| `Result` | 响应结果 | `LoginResult`, `ExcelResultVO` |

### 3.3 方法命名

| 操作 | 方法名 | 示例 |
|------|--------|------|
| 分页查询 | `get_page` | `async def get_page(self, query: UserQuery) -> PageResult` |
| 单条查询 | `get_by_id` | `async def get_by_id(self, id: int) -> UserVO` |
| 树形查询 | `get_tree` | `async def get_tree(self, keywords: str) -> list[MenuVO]` |
| 下拉选项 | `get_options` | `async def get_options(self) -> list[dict]` |
| 创建 | `create` | `async def create(self, form: UserCreate) -> UserVO` |
| 更新 | `update` | `async def update(self, form: UserUpdate) -> UserVO` |
| 删除（批量） | `delete` | `async def delete(self, ids: str) -> int` |
| 状态变更 | `update_status` | `async def update_status(self, id: int, status: int) -> None` |
| 导入 | `import_<资源>` | `async def import_users(self, data: list) -> dict` |
| 导出 | `export_<资源>` | `async def export_users(self, query) -> list[dict]` |

### 3.4 变量命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 变量 | snake_case | `user_list`, `role_codes` |
| 常量 | UPPER_SNAKE_CASE | `MAX_PAGE_SIZE`, `ROOT_ROLE_CODE` |
| 私有方法 | `_` 前缀 | `def _to_vo(self, m: SysUser) -> UserVO` |
| 布尔变量 | `is_`/`has_` 前缀 | `is_root`, `is_deleted`, `has_perm` |
| 数据库字段 | snake_case | `dept_id`, `create_time` |
| JSON/API 字段 | camelCase | `deptId`, `createTime`, `pageSize` |

> **关键约定**：ORM 模型字段用 snake_case（对应 PostgreSQL 列名），Schema/API 字段用 camelCase（对应前端 JSON 键名）。

### 3.5 import 排序

```python
# 1. 标准库
import io
import logging

# 2. 第三方库
from fastapi import APIRouter, Depends
from sqlalchemy import select, func
from sqlalchemy.ext.asyncio import AsyncSession
from pydantic import BaseModel, Field

# 3. 项目内部（按层排序）
from app.database import get_db
from app.constants import ROOT_ROLE_CODE
from app.pagination import PageResult
from app.dependencies import require_perm
from app.exceptions import BusinessException
from app.response import Result, ResultCode
from app.user.models import SysUser
from app.user.schemas import UserCreate, UserVO
```

---

## Part 4: 路由与 API 规范

### 4.1 路由前缀

```python
router = APIRouter(prefix="/api/v1/users", tags=["用户管理"])
```

| 模块 | 前缀 |
|------|------|
| 认证 | `/api/v1/auth` |
| 用户 | `/api/v1/users` |
| 角色 | `/api/v1/roles` |
| 菜单 | `/api/v1/menus` |
| 部门 | `/api/v1/depts` |
| 字典 | `/api/v1/dict` |
| 配置 | `/api/v1/configs` |
| 通知 | `/api/v1/notices` |
| 日志 | `/api/v1/logs` |
| 文件 | `/api/v1/files` |
| SSE | `/api/v1/sse` |

### 4.2 标准 CRUD 路径

| 操作 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 分页列表 | `GET` | `/api/v1/users` | Query 参数：`pageNum`/`pageSize`/`keywords`/... |
| 详情 | `GET` | `/api/v1/users/{user_id}` | 路径参数 `user_id: int` |
| 新增 | `POST` | `/api/v1/users` | Body: `UserCreate` |
| 更新 | `PUT` | `/api/v1/users/{user_id}` | Body: `UserUpdate` |
| 批量删除 | `DELETE` | `/api/v1/users/{ids}` | `ids` 逗号分隔 |
| 修改状态 | `PATCH` | `/api/v1/users/status` | Body: `UserStatusForm` |
| 重置密码 | `PATCH` | `/api/v1/users/password` | Body: `UserPasswordForm` |
| 导出 | `GET` | `/api/v1/users/export` | 返回 StreamingResponse xlsx |
| 导入 | `POST` | `/api/v1/users/import` | UploadFile xlsx |
| 导入模板 | `GET` | `/api/v1/users/template` | 下载模板 xlsx |
| 下拉选项 | `GET` | `/api/v1/roles/options` | 返回 `[{value, label}]` |

### 4.3 路由注册顺序（重要！）

**固定路径端点必须定义在路径参数端点之前**，否则 `/{user_id}` 会先被匹配：

```python
# ✅ 正确顺序
@router.get("")                    # GET /api/v1/users
@router.get("/template")           # 固定路径 —— 必须在 /{user_id} 前
@router.get("/export")             # 固定路径
@router.post("/import")            # 固定路径
@router.patch("/status")           # 固定路径
@router.patch("/password")         # 固定路径

@router.post("")                   # POST /api/v1/users
@router.get("/{user_id}")          # 路径参数 —— 最后注册
@router.put("/{user_id}")          # 路径参数
@router.delete("/{ids}")           # 路径参数
```

### 4.4 Router 函数模板

```python
# router.py — 完整模板

import io
import logging
from fastapi import APIRouter, Depends, Query, UploadFile, File
from sqlalchemy.ext.asyncio import AsyncSession
from openpyxl import Workbook

from app.database import get_db
from app.dependencies import require_perm
from app.response import Result
from app.user.schemas import UserCreate, UserQuery, UserUpdate, UserVO
from app.user.service import UserService

logger = logging.getLogger(__name__)
router = APIRouter(prefix="/api/v1/users", tags=["用户管理"])


@router.get("", summary="用户分页列表",
    dependencies=[Depends(require_perm("sys:user:list"))])
async def get_user_page(
    pageNum: int = Query(default=1, ge=1),
    pageSize: int = Query(default=10, ge=1, le=100),
    keywords: str | None = Query(default=None),
    status: int | None = Query(default=None),
    db: AsyncSession = Depends(get_db),
):
    """分页查询用户列表。

    - **pageNum**: 当前页码
    - **pageSize**: 每页条数（1-100）
    - **keywords**: 搜索关键词（用户名/昵称/手机号）
    - **status**: 状态筛选
    """
    query = UserQuery(pageNum=pageNum, pageSize=pageSize,
                      keywords=keywords, status=status)
    result = await UserService(db).get_page(query)
    return Result(data=result)


@router.get("/{user_id}", summary="用户详情",
    dependencies=[Depends(require_perm("sys:user:detail"))])
async def get_user(user_id: int, db: AsyncSession = Depends(get_db)):
    """根据 ID 获取用户详情。"""
    vo = await UserService(db).get_by_id(user_id)
    return Result(data=vo)


@router.post("", summary="创建用户",
    dependencies=[Depends(require_perm("sys:user:create"))])
async def create_user(form: UserCreate, db: AsyncSession = Depends(get_db)):
    """创建新用户，自动分配角色。"""
    vo = await UserService(db).create(form)
    return Result(data=vo)


@router.put("/{user_id}", summary="更新用户",
    dependencies=[Depends(require_perm("sys:user:edit"))])
async def update_user(user_id: int, form: UserUpdate,
                      db: AsyncSession = Depends(get_db)):
    """更新用户信息及角色。"""
    form.id = user_id
    vo = await UserService(db).update(form)
    return Result(data=vo)


@router.delete("/{ids}", summary="删除用户",
    dependencies=[Depends(require_perm("sys:user:delete"))])
async def delete_users(ids: str, db: AsyncSession = Depends(get_db)):
    """批量逻辑删除用户。"""
    count = await UserService(db).delete(ids)
    return Result(data=count, msg=f"成功删除 {count} 条记录")
```

---

## Part 5: HTTP 状态码与业务错误码

### 5.1 设计理念：双轨制（HTTP 语义 + 业务码）

> **背景**：社区存在两派争论——"统一返回 200，用 body 的 code 区分" vs "用正确的 HTTP 状态码"。本规范采用 **RFC 9457 Problem Details** 思路 + 兼容 youlai-boot/vue3-element-admin 前端拦截器的**双轨制方案**。

| 方案 | 代表 | 缺陷 |
|------|------|------|
| 统一 200 | 部分国内项目、GraphQL | 违背 HTTP 语义；网关/监控无法识别错误；API 工具误判成功 |
| 纯 HTTP 状态码 | RFC 9457、Zuplo 最佳实践 | 无法表达细粒度业务错误（如"用户名不存在" vs "密码错误"都是 401） |
| **双轨制（本项目）** | youlai-fastapi | — |

**双轨制原则**：

```
HTTP 状态码 → 大类划分（2xx 成功 / 4xx 客户端错误 / 5xx 服务端错误）
业务错误码  → 细粒度分类（A0230 Token 过期 / A0402 密码错误 / B0003 数据不存在）
```

- **HTTP 状态码**：协议层，用于网关/监控/API 工具识别请求成败
- **业务错误码**：应用层，用于前端精确判断业务逻辑
- **两者共存**：每个错误响应同时携带正确的 HTTP 状态码和 body 中的 `code` 字段

### 5.2 HTTP 状态码使用规范

| HTTP 状态 | 含义 | 使用场景 | body 中的 code 示例 |
|-----------|------|----------|---------------------|
| `200` | 成功 | 业务操作成功 | `00000` |
| `400` | 客户端请求错误 | 请求格式错误、参数缺失 | `A0400` |
| `401` | 未认证 | Token 无效/过期、未登录 | `A0230`、`A0231` |
| `403` | 无权限 | 已认证但无操作权限 | `A0301` |
| `404` | 资源不存在 | 查询的数据不存在 | `B0003` |
| `409` | 冲突 | 唯一键冲突、状态冲突 | `B0002` |
| `422` | 参数校验失败 | Pydantic 校验失败 | `A0400` |
| `500` | 服务器内部错误 | 未捕获异常、系统错误 | `B0001` |

> **为什么不用 4xx 表示所有业务错误？**
> HTTP 状态码应反映**协议层**的成败。"密码错误"是业务校验失败而非请求格式错误，用 400 会与参数格式错误混淆。本项目将"密码错误"归为 401（认证失败），"用户名不存在"归为 404（资源不存在），"唯一键冲突"归为 409（冲突）——选择语义最匹配的状态码。

### 5.3 业务错误码（ResultCode）

```python
from enum import Enum


class ResultCode(str, Enum):
    """业务错误码 — 与 HTTP 状态码配合使用。

    首位编码：
      0 - 成功
      A - 用户端错误（4xx）
      B - 系统端错误（5xx）
      C - 第三方服务错误（5xx）
    """

    # ── 成功 ──
    SUCCESS = "00000"                    # → HTTP 200

    # ── 认证/授权类（4xx）──
    TOKEN_INVALID = "A0230"              # → HTTP 401 访问令牌无效或过期
    TOKEN_REFRESH_FAIL = "A0231"         # → HTTP 401 刷新令牌无效或过期
    ACCESS_DENIED = "A0301"              # → HTTP 403 权限不足

    # ── 参数/校验类（4xx）──
    PARAM_VALID_FAIL = "A0400"           # → HTTP 422 参数校验失败
    USERNAME_NOT_FOUND = "A0401"         # → HTTP 404 用户名不存在
    BAD_CREDENTIALS = "A0402"            # → HTTP 401 密码错误
    USER_DISABLED = "A0403"             # → HTTP 403 用户被禁用
    CAPTCHA_ERROR = "A0404"             # → HTTP 400 验证码错误

    # ── 业务异常 ──
    SYSTEM_ERROR = "B0001"              # → HTTP 500 系统执行异常（全局兜底）
    DUPLICATE_KEY = "B0002"              # → HTTP 409 数据重复
    DATA_NOT_FOUND = "B0003"             # → HTTP 404 数据不存在
    OPERATE_DENIED = "B0004"             # → HTTP 403 操作不允许

    @property
    def http_status(self) -> int:
        """业务错误码 → 对应的 HTTP 状态码。"""
        mapping = {
            "00000": 200,
            "A0230": 401, "A0231": 401,
            "A0301": 403, "A0403": 403, "B0004": 403,
            "A0400": 422,
            "A0401": 404, "B0003": 404,
            "A0402": 401,
            "A0404": 400,
            "B0002": 409,
            "B0001": 500,
        }
        return mapping.get(self.value, 500)
```

### 5.4 BusinessException 与全局异常处理器

```python
# app/exceptions.py

from fastapi import Request
from fastapi.responses import JSONResponse
from app.response import ResultCode


class BusinessException(Exception):
    """业务异常 — 携带业务错误码和 HTTP 状态码。"""

    def __init__(self, code: ResultCode = ResultCode.SYSTEM_ERROR, msg: str = "系统异常"):
        self.code = code
        self.msg = msg
        self.http_status = code.http_status  # ← 自动映射 HTTP 状态码
        super().__init__(msg)


async def business_exception_handler(request: Request, exc: BusinessException) -> JSONResponse:
    """业务异常处理 — 返回语义化 HTTP 状态码 + body 中的 code。"""
    return JSONResponse(
        status_code=exc.http_status,            # ← HTTP 401/403/404/409/500...
        content={"code": exc.code.value, "msg": exc.msg, "data": None},
    )


async def validation_exception_handler(request: Request, exc) -> JSONResponse:
    """参数校验异常 — HTTP 422 + body code A0400。"""
    return JSONResponse(
        status_code=422,
        content={"code": ResultCode.PARAM_VALID_FAIL.value, "msg": str(exc), "data": None},
    )


async def global_exception_handler(request: Request, exc: Exception) -> JSONResponse:
    """全局兜底 — HTTP 500 + body code B0001。"""
    return JSONResponse(
        status_code=500,
        content={"code": ResultCode.SYSTEM_ERROR.value, "msg": "系统执行异常", "data": None},
    )
```

### 5.5 业务异常抛出示例

```python
# ✅ 认证失败 → HTTP 401
raise BusinessException(code=ResultCode.TOKEN_INVALID, msg="访问令牌无效或过期")
raise BusinessException(code=ResultCode.BAD_CREDENTIALS, msg="密码错误")

# ✅ 权限不足 → HTTP 403
raise BusinessException(code=ResultCode.ACCESS_DENIED, msg="权限不足")

# ✅ 数据不存在 → HTTP 404
raise BusinessException(code=ResultCode.DATA_NOT_FOUND, msg="用户不存在")
raise BusinessException(code=ResultCode.USERNAME_NOT_FOUND, msg="用户名不存在")

# ✅ 唯一键冲突 → HTTP 409
raise BusinessException(code=ResultCode.DUPLICATE_KEY, msg="用户名已存在")

# ✅ 操作不允许 → HTTP 403
raise BusinessException(code=ResultCode.OPERATE_DENIED, msg="存在子菜单，无法删除")

# ✅ 系统异常 → HTTP 500（由全局异常处理器兜底，通常不手动抛出）
```

### 5.6 与前端 axios 拦截器的兼容性

vue3-element-admin 的 axios 拦截器**同时处理两个分支**，天然兼容双轨制：

```typescript
// vue3-element-admin/src/utils/request.ts（已验证）
http.interceptors.response.use(
  (response) => {
    // HTTP 2xx 进入此分支
    const { code, data, msg } = response.data;
    if (code === "00000") return data;        // 业务成功
    ElMessage.error(msg);                      // 业务错误（HTTP 200 但 code 非 00000）
    return Promise.reject(new Error(msg));
  },
  (error) => {
    // HTTP 4xx/5xx 进入此分支
    const { code, msg } = error.response.data;
    if (code === "A0230") { /* Token 过期 → 自动刷新 */ }
    if (code === "A0231") { /* Refresh 失败 → 跳转登录 */ }
    if (code === "A0301") { /* 权限不足 → 刷新权限 */ }
    ElMessage.error(msg);
    return Promise.reject(error);
  }
);
```

> **关键**：前端无论进入哪个分支，都从 `response.data.code` 读取业务码。双轨制方案前端**无需任何改动**。

### 5.7 错误码设计规范

| 首位 | 含义 | HTTP 大类 | 范围 |
|------|------|-----------|------|
| `0` | 成功 | 2xx | `00000` |
| `A` | 用户端错误 | 4xx | `A0001-A0999` |
| `B` | 系统端错误 | 5xx | `B0001-B0999` |
| `C` | 第三方服务错误 | 5xx | `C0001-C0999` |

**第二位子分类**（A 类）：

| 第二位 | 子类 | 示例 |
|--------|------|------|
| `0` | 通用 | `A0001` 未知错误 |
| `2` | 认证/Token | `A0230` Token 过期、`A0231` Refresh 失效 |
| `3` | 授权/权限 | `A0301` 权限不足 |
| `4` | 参数校验 | `A0400` 校验失败、`A0401` 用户不存在、`A0402` 密码错误 |
| `5` | 租户 | `A0250` 需要选择租户 |

**新增错误码流程**：

1. 在 `ResultCode` 枚举中添加新值
2. 在 `http_status` property 的 mapping 中添加映射
3. 更新本规范的 5.3 节表格

---

## Part 6: 统一响应格式

### 6.1 Result 结构

```json
{
    "code": "00000",
    "msg": "成功",
    "data": { … }
}
```

```python
from pydantic import BaseModel, Field
from typing import Generic, TypeVar

T = TypeVar("T")

class Result(BaseModel, Generic[T]):
    code: str = Field(default="00000", description="业务状态码")
    msg: str = Field(default="成功", description="提示信息")
    data: T | None = Field(default=None, description="响应数据")
```

### 6.2 PageResult 结构

```json
{
    "code": "00000",
    "msg": "成功",
    "data": {
        "list": [ … ],
        "total": 100,
        "pageNum": 1,
        "pageSize": 10
    }
}
```

```python
class PageResult(BaseModel):
    list: list = Field(default_factory=list)
    total: int = Field(default=0)
    pageNum: int = Field(default=1)
    pageSize: int = Field(default=10)
```

### 6.3 分页查询参数

```python
class PageQuery(BaseModel):
    pageNum: int = Field(default=1, ge=1)
    pageSize: int = Field(default=10, ge=1, le=100)
```

> **前端约定**：分页参数命名为 `pageNum` / `pageSize`（camelCase），兼容 vue3-element-admin 的 TablePage 组件。

### 6.4 返回示例

```python
# 成功返回
return Result(data=user_vo)

# 成功返回（自定义消息）
return Result(data=count, msg=f"成功删除 {count} 条记录")

# 成功返回（无数据）
return Result(data=None)

# 错误返回
return Result(code=ResultCode.CAPTCHA_ERROR, msg="验证码错误", data=None)
```

---

## Part 7: ORM 模型规范

### 7.1 基类 Mixin

```python
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column
from sqlalchemy import BigInteger, DateTime, Integer, func


class Base(DeclarativeBase):
    """ORM 声明式基类。"""
    pass


class TimestampMixin:
    """自动管理 create_time / update_time。"""
    create_time: Mapped[datetime | None] = mapped_column(
        DateTime, server_default=func.now(), comment="创建时间"
    )
    update_time: Mapped[datetime | None] = mapped_column(
        DateTime, server_default=func.now(), onupdate=func.now(), comment="更新时间"
    )


class SoftDeleteMixin:
    """逻辑删除标识。"""
    is_deleted: Mapped[int] = mapped_column(
        Integer, default=0, server_default="0", comment="逻辑删除 0-未删除 1-已删除"
    )


class BaseIdMixin:
    """自增主键 ID。"""
    id: Mapped[int] = mapped_column(
        BigInteger, primary_key=True, autoincrement=True, comment="主键ID"
    )
```

### 7.2 ORM 模型示例

```python
class SysUser(Base, BaseIdMixin, TimestampMixin, SoftDeleteMixin):
    __tablename__ = "sys_user"

    username: Mapped[str] = mapped_column(String(64), unique=True, comment="用户名")
    nickname: Mapped[str] = mapped_column(String(64), comment="昵称")
    password: Mapped[str] = mapped_column(String(100), comment="密码")
    dept_id: Mapped[int | None] = mapped_column(BigInteger, comment="部门ID")
    mobile: Mapped[str | None] = mapped_column(String(20), comment="手机号")
    email: Mapped[str | None] = mapped_column(String(100), comment="邮箱")
    status: Mapped[int] = mapped_column(Integer, default=1, server_default="1",
                                         comment="状态 1-启用 0-禁用")
```

### 7.3 字段命名对照

| SQL 列名 | ORM 字段 | Schema 字段 | 说明 |
|----------|----------|-------------|------|
| `dept_id` | `dept_id` | `deptId` | 部门ID |
| `create_time` | `create_time` | `createTime` | 创建时间 |
| `is_deleted` | `is_deleted` | — | 逻辑删除（内部使用） |
| `parent_id` | `parent_id` | `parentId` | 父节点ID |

> **转换规则**：ORM → VO 时，`to_vo` 或 `_to_vo` 方法负责 snake_case → camelCase 转换。

### 7.4 VO 转换模板

```python
@staticmethod
def _to_vo(m: SysUser) -> UserVO:
    """ORM 模型 → 视图对象（snake_case → camelCase）。"""
    return UserVO(
        id=m.id,
        username=m.username,
        deptId=m.dept_id,       # ← 手动映射
        createTime=str(m.create_time) if m.create_time else None,
        # ...
    )
```

---

## Part 8: 认证与鉴权

### 8.1 Token 模式

通过 `SESSION_TYPE` 环境变量切换：

| 值 | 模式 | TokenManager |
|----|------|--------------|
| `jwt`（默认） | JWT 自包含 Token | `JwtTokenManager` |
| `redis-token` | Redis 存储 Token | `RedisTokenManager` |

### 8.2 获取当前用户

```python
from app.dependencies import get_current_user

@router.get("/me")
async def my_info(user: SysUserDetails = Depends(get_current_user)):
    return Result(data={"userId": user.userId, "username": user.username})
```

### 8.3 权限校验

```python
from app.dependencies import require_perm

# 精确权限
@router.post("", dependencies=[Depends(require_perm("sys:user:create"))])
async def create_user(form: UserCreate, db: AsyncSession = Depends(get_db)):
    ...

# 登录即可（不检查权限）
@router.get("/options", dependencies=[Depends(require_perm())])
async def get_options(db: AsyncSession = Depends(get_db)):
    ...
```

### 8.4 权限标识命名规范

```
模块:实体:操作
sys:user:create
sys:user:edit
sys:user:delete
sys:role:list
sys:menu:list
sys:dept:list
```

| 操作 | 说明 |
|------|------|
| `list` | 列表/查询 |
| `detail` | 详情 |
| `create` | 新增 |
| `edit` | 编辑 |
| `delete` | 删除 |
| `import` | 导入 |
| `export` | 导出 |
| `reset-pwd` | 重置密码 |
| `assign-menu` | 分配菜单权限 |

---

## Part 9: 数据库操作规范

### 9.1 异步查询

```python
# ✅ 正确：使用 async/await
result = await self.db.execute(
    select(SysUser).where(SysUser.id == user_id, SysUser.is_deleted == 0)
)
user = result.scalar_one_or_none()

# ✅ 正确：分页查询
rows = await self.db.execute(
    select(SysUser)
    .where(*conditions)
    .order_by(SysUser.create_time.desc())
    .offset(offset)
    .limit(page_size)
)
users = rows.scalars().all()

# ✅ 正确：原生 SQL（复杂联表查询）
result = await self.db.execute(
    text("""
        SELECT r.code FROM sys_role r
        INNER JOIN sys_user_role ur ON r.id = ur.role_id
        WHERE ur.user_id = :user_id AND r.is_deleted = 0 AND r.status = 1
    """),
    {"user_id": user_id},
)
```

### 9.2 事务管理

```python
# 由 get_db() 依赖自动管理：
# yield 前 → session 开始
# yield 后 → commit（成功）或 rollback（异常）
# finally → session.close()

async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()     # 自动提交
        except Exception:
            await session.rollback()   # 自动回滚
            raise
        finally:
            await session.close()
```

### 9.3 批量操作

```python
# ✅ 批量删除（原生 SQL）
await self.db.execute(
    text("UPDATE sys_user SET is_deleted = 1 WHERE id = ANY(:ids)"),
    {"ids": id_list},
)

# ✅ 批量插入
for data in batch:
    self.db.add(SysConfig(**data))
await self.db.flush()
```

---

## Part 10: 添加新模块 — 完整教程

以添加 **"数据字典项"** 模块为例，展示从零到完整的步骤。

### Step 1: 检查 ORM 模型

确认 `app/system/dict/models.py` 中已有对应的 ORM 模型。若无，创建：

```python
# app/system/dict/models.py
class SysDictItem(Base, BaseIdMixin):
    __tablename__ = "sys_dict_item"
    dict_code: Mapped[str | None] = mapped_column(String(50))
    value: Mapped[str | None] = mapped_column(String(50))
    label: Mapped[str | None] = mapped_column(String(100))
    tag_type: Mapped[str | None] = mapped_column(String(50))
    status: Mapped[int] = mapped_column(Integer, default=1)
    sort: Mapped[int] = mapped_column(Integer, default=0)
```

### Step 2: 创建模块目录

```bash
mkdir app/dict_item
```

### Step 3: 创建 `schemas.py`

```python
# app/dict_item/schemas.py
"""字典项 Schemas。"""
from pydantic import BaseModel, Field


class DictItemCreate(BaseModel):
    """创建字典项表单。"""
    dictCode: str = Field(..., max_length=50, description="关联字典编码")
    value: str = Field(..., max_length=50, description="字典项值")
    label: str = Field(..., max_length=100, description="字典项标签")
    tagType: str | None = Field(default=None, max_length=50, description="标签类型")
    status: int = Field(default=1, description="状态 1-正常 0-禁用")
    sort: int = Field(default=0, description="排序")


class DictItemUpdate(DictItemCreate):
    """更新字典项表单。"""
    id: int = Field(..., description="字典项ID")


class DictItemVO(BaseModel):
    """字典项视图对象。"""
    id: int | None = None
    dictCode: str = ""
    value: str = ""
    label: str = ""
    tagType: str | None = None
    status: int = 1
    sort: int = 0
    model_config = {"from_attributes": True}


class DictItemQuery(BaseModel):
    """字典项查询参数。"""
    dictCode: str = Field(..., description="字典编码")
```

### Step 4: 创建 `service.py`

```python
# app/dict_item/service.py
"""字典项服务。"""
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from app.exceptions import BusinessException
from app.response import ResultCode
from app.dict.models import SysDictItem
from app.dict_item.schemas import (
    DictItemCreate, DictItemUpdate, DictItemVO, DictItemQuery,
)


class DictItemService:
    """字典项管理服务。"""

    def __init__(self, db: AsyncSession):
        self.db = db

    async def get_items(self, dict_code: str) -> list[DictItemVO]:
        """根据字典编码查询字典项列表。"""
        rows = await self.db.execute(
            select(SysDictItem)
            .where(SysDictItem.dict_code == dict_code)
            .order_by(SysDictItem.sort.asc())
        )
        items = rows.scalars().all()
        return [DictItemVO.model_validate(it, from_attributes=True) for it in items]

    async def get_by_id(self, item_id: int) -> DictItemVO:
        """根据 ID 查询字典项。"""
        obj = await self.db.get(SysDictItem, item_id)
        if obj is None:
            raise BusinessException(code=ResultCode.DATA_NOT_FOUND, msg="字典项不存在")
        return DictItemVO.model_validate(obj, from_attributes=True)

    async def create(self, form: DictItemCreate) -> DictItemVO:
        """创建字典项。"""
        obj = SysDictItem(**form.model_dump())
        self.db.add(obj)
        await self.db.flush()
        return DictItemVO.model_validate(obj, from_attributes=True)

    async def update(self, form: DictItemUpdate) -> DictItemVO:
        """更新字典项。"""
        obj = await self.db.get(SysDictItem, form.id)
        if obj is None:
            raise BusinessException(code=ResultCode.DATA_NOT_FOUND, msg="字典项不存在")
        for key, val in form.model_dump(exclude={"id"}).items():
            setattr(obj, key, val)
        await self.db.flush()
        return DictItemVO.model_validate(obj, from_attributes=True)

    async def delete(self, ids: str) -> int:
        """批量删除字典项。"""
        id_list = [int(x) for x in ids.split(",") if x.strip()]
        for iid in id_list:
            obj = await self.db.get(SysDictItem, iid)
            if obj:
                await self.db.delete(obj)
        await self.db.flush()
        return len(id_list)
```

### Step 5: 创建 `router.py`

```python
# app/dict_item/router.py
"""字典项路由。"""
from fastapi import APIRouter, Depends
from sqlalchemy.ext.asyncio import AsyncSession

from app.database import get_db
from app.dependencies import require_perm
from app.response import Result
from app.dict_item.schemas import (
    DictItemCreate, DictItemUpdate,
)
from app.dict_item.service import DictItemService

router = APIRouter(prefix="/api/v1/dict/items", tags=["字典管理"])


@router.get("/{dict_code}", summary="字典项列表")
async def get_dict_items(dict_code: str, db: AsyncSession = Depends(get_db)):
    items = await DictItemService(db).get_items(dict_code)
    return Result(data=items)


@router.get("/{item_id}/detail", summary="字典项详情")
async def get_dict_item(item_id: int, db: AsyncSession = Depends(get_db)):
    vo = await DictItemService(db).get_by_id(item_id)
    return Result(data=vo)


@router.post("", summary="创建字典项",
    dependencies=[Depends(require_perm("sys:dict:create"))])
async def create_dict_item(form: DictItemCreate,
    db: AsyncSession = Depends(get_db)):
    vo = await DictItemService(db).create(form)
    return Result(data=vo)


@router.put("/{item_id}", summary="更新字典项",
    dependencies=[Depends(require_perm("sys:dict:edit"))])
async def update_dict_item(item_id: int, form: DictItemUpdate,
    db: AsyncSession = Depends(get_db)):
    form.id = item_id
    vo = await DictItemService(db).update(form)
    return Result(data=vo)


@router.delete("/{ids}", summary="删除字典项",
    dependencies=[Depends(require_perm("sys:dict:delete"))])
async def delete_dict_items(ids: str, db: AsyncSession = Depends(get_db)):
    count = await DictItemService(db).delete(ids)
    return Result(data=count)
```

### Step 6: 创建 `__init__.py`

```python
# app/dict_item/__init__.py
```

### Step 7: 在 `main.py` 注册路由

```python
# app/main.py — create_app() 函数中追加
from app.dict_item.router import router as dict_item_router
app.include_router(dict_item_router)
```

### Step 8: 验证

```bash
# 启动服务
fastapi dev app/main.py

# 访问 Swagger 确认路由已注册
open http://localhost:8000/api/v1/swagger-ui.html
```

### 模块文件清单

```
app/dict_item/
├── __init__.py
├── schemas.py       # ← 第一步创建
├── service.py       # ← 第二步创建
└── router.py        # ← 第三步创建
```

### 扩展：单文件模块（简单 CRUD）

对于日志管理等简单模块，可将 schemas + service + router 合并为一个文件：

```python
# app/system/log/router.py
# ┌── schemas ──┐
class LogQuery(BaseModel): ...
class LogVO(BaseModel): ...

# ┌── service ──┐
class LogService: ...
    async def get_page(self, query): ...

# ┌── router ──┐
router = APIRouter(prefix="/api/v1/logs", tags=["日志管理"])

@router.get("", ...)
async def get_logs(...): ...
```

> **选择标准**：单个实体 < 5 个接口 → 单文件模块；> 5 个接口或有复杂查询 → 拆分为 schemas + service + router。

---

## Part 11: 注释规范

> **来源**：PEP 257 (Docstring Conventions) + Google Style Docstrings。

### 11.1 总则

- **docstring 优先**：所有公共模块、类、函数必须有 docstring
- **自解释代码**：好的命名和结构是自解释的，注释描述"为什么"而非"如何做"
- **中文优先**：与其用半吊子英文，不如用中文把问题说清楚；专有名词保留英文（如 `HTTP`、`async`）
- **同步更新**：代码修改时 docstring 必须同步修改

### 11.2 docstring 格式（Google 风格）

```python
async def get_user_page(
    pageNum: int = Query(default=1, ge=1),
    pageSize: int = Query(default=10, ge=1, le=100),
    keywords: str | None = Query(default=None),
    db: AsyncSession = Depends(get_db),
) -> PageResult:
    """分页查询用户列表。

    根据关键词模糊搜索用户名/昵称/手机号，支持状态筛选。

    Args:
        pageNum: 当前页码，从 1 开始。
        pageSize: 每页条数，1-100。
        keywords: 搜索关键词（用户名/昵称/手机号），为空时查全部。
        db: 异步数据库会话。

    Returns:
        分页结果，包含 list 和 total。

    Raises:
        BusinessException: 当 pageSize 超过 100 时（通常由 Pydantic 校验拦截）。
    """
    ...
```

### 11.3 各层注释要点

| 层 | 必须说明 | 示例 |
|----|---------|------|
| 模块（文件头） | 文件职责 | `"""字典项路由。"""` |
| 类 | 职责、关键依赖 | `"""字典项管理服务，依赖 AsyncSession。"""` |
| 公共方法 | 功能、参数、返回值、异常 | 见 11.2 |
| 私有方法（`_` 前缀） | 简要说明即可 | `"""ORM 模型 → 视图对象。"""` |

### 11.4 常见反模式

| 反模式 | 问题 | 正确做法 |
|--------|------|----------|
| `def get_user(id): pass` 无 docstring | 公共方法无文档 | 补充 docstring |
| `"""获取用户"""` 参数无说明 | 缺少 Args/Returns | 补充完整 Args/Returns/Raises |
| docstring 与代码不一致 | 误导维护者 | 代码改动时同步更新 |
| 注释掉的代码块保留 | 历史代码可从 Git 查阅 | 直接删除 |
| `# TODO fix this` | 无标记人、无时间 | `# TODO(zhangsan, 2025/06/01, 下版本修复)` |

### 11.5 TODO / FIXME 标记规范

| 标记 | 用途 | 格式 | 示例 |
|------|------|------|------|
| `TODO` | 待实现 | `TODO(标记人, 日期, [说明])` | `TODO(zhangsan, 2025/06/01, 扩展邮箱验证码)` |
| `FIXME` | 已知 bug | `FIXME(标记人, 日期, [说明])` | `FIXME(lisi, 2025/06/15, 并发场景偶发死锁)` |

---

## Part 12: 代码质量检查清单

### 11.1 通用检查

- [ ] 所有公共类、方法有 docstring
- [ ] 用 `Pydantic v2` 的 `model_dump()` 而非 `dict()`
- [ ] 用 `model_validate(obj, from_attributes=True)` 做 ORM → Schema 转换
- [ ] 所有 `.py` 文件有 `# -*- coding: utf-8 -*-` 或 UTF-8 编码
- [ ] import 按标准库 → 第三方 → 内部顺序排列
- [ ] 无未使用的 import
- [ ] 无 `print()` 调试代码，统一用 `logger`
- [ ] 日志不输出密码、密钥等敏感信息
- [ ] 配置通过 `Settings` + `.env` 管理，不硬编码
- [ ] 所有 `async def` 函数内部使用 `await`

### 11.2 数据库相关

- [ ] ORM 模型字段名使用 `snake_case`，与数据库列名一致
- [ ] 所有查询加上 `is_deleted == 0` 过滤（如模型有 `SoftDeleteMixin`）
- [ ] 使用 `select` 而非 `Model.query`（SQLAlchemy 2.0+ 风格）
- [ ] 大量数据操作使用 `flush()` 而非频繁 `commit()`
- [ ] `ScalarResult.scalar_one_or_none()` 而非 `first()`（区分 None vs 第一条）

### 11.3 API 相关

- [ ] 路由 prefix 统一 `"/api/v1/模块名"`
- [ ] 固定路径端点定义在路径参数端点之前
- [ ] 分页参数命名为 `pageNum` / `pageSize`
- [ ] JSON 字段使用 camelCase（`deptId`、`createTime`）
- [ ] 统一返回 `Result[T]` 包装（body 含 `code`/`msg`/`data`）
- [ ] **HTTP 状态码语义化**：成功 200 / 认证失败 401 / 权限不足 403 / 不存在 404 / 冲突 409 / 校验 422 / 系统错误 500
- [ ] 业务错误码与 HTTP 状态码双轨配合（见 Part 5）
- [ ] `ResultCode.http_status` property 映射正确
- [ ] 错误返回使用 `ResultCode` 枚举，不硬编码字符串
- [ ] 敏感接口添加 `dependencies=[Depends(require_perm(...))]`
- [ ] tree_path 在 `add + flush` 之后再设置（拿到自增 id）

### 11.4 Service 层

- [ ] Service 构造函数接受 `db: AsyncSession`
- [ ] 数据不存在时抛出 `BusinessException(code=ResultCode.DATA_NOT_FOUND)`
- [ ] 唯一键冲突时抛出 `BusinessException(code=ResultCode.DUPLICATE_KEY)`
- [ ] 操作不允许时抛出 `BusinessException(code=ResultCode.OPERATE_DENIED)`
- [ ] 批量操作前检查参数非空
- [ ] `_to_vo` / `_to_tree` 等私有方法前缀 `_`

---

## Part 13: 常见反模式

| 反模式 | 问题 | 正确做法 |
|--------|------|----------|
| `print("debug")` | 无日志级别、无时间戳 | `logger.info()` / `logger.debug()` |
| `result.scalars().first()` | 不区分 None vs 第一条 | `result.scalar_one_or_none()` |
| `model.dict()` | Pydantic v2 已废弃 | `model.model_dump()` |
| 硬编码配置 | 不便运维 | 用 `Settings` + `.env` |
| `db.execute()` 后不 `await` | 异步未执行 | `await self.db.execute(...)` |
| `flush` 前取 `obj.id` | 自增 id 尚未生成 | 先 `flush()` 再取 `id` |
| SQL 查询缺 `is_deleted = 0` | 查到已删除数据 | 所有查询加逻辑删除条件 |
| 固定路径在 `/{id}` 之后 | 先被路径参数拦截 | 固定路径注册在前 |
| 重复 import 同一模块 | 不必要 | 合并为单条 import |
| `try/except` 捕获所有异常后 pass | 隐藏错误 | 只捕获已知异常，记录日志 |
| **所有错误统一返回 HTTP 200** | 网关/监控无法识别错误；违背 HTTP 语义 | HTTP 状态码语义化 + body 中 `code` 双轨制（Part 5） |
| **业务错误硬编码 HTTP 状态码** | 如 `JSONResponse(status_code=401)` 散落各处 | 用 `ResultCode.http_status` property 统一映射 |
| **字符串硬编码业务码** | 如 `code="A0230"` | 用 `ResultCode.TOKEN_INVALID.value` 枚举 |

---

## Part 14: 单模块文件

对于 `config`、`notice`、`log` 等简单模块，可将 schemas + service + router 合并到单个 `router.py` 文件中，减少文件碎片化。

---

## Part 15: Docker 部署

```yaml
# docker/docker-compose.yml
services:
  api:
    build: ..
    ports: ["8000:8000"]
    depends_on: [postgres, redis]
    environment:
      DATABASE_URL: postgresql+asyncpg://postgres:123456@postgres:5432/youlai_admin
      REDIS_URL: redis://redis:6379/0
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: youlai_admin
      POSTGRES_PASSWORD: 123456
    ports: ["5432:5432"]

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]
```

```bash
# 启动
docker-compose -f docker/docker-compose.yml up -d
```
