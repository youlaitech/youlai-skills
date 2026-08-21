---
name: spring-boot
description: This skill should be used when developing Spring Boot projects, implementing REST APIs with MyBatis-Plus, configuring authentication with JWT/Redis tokens, implementing permission control with Spring Security, or adding new backend modules in the youlai-boot project.
---

# Spring Boot 后端开发规范

## 技术栈

| 层 | 选型 | 说明 |
|----|------|------|
| JDK | Java 17+ | 运行环境 |
| 基础框架 | Spring Boot 3.x | 自动配置、内嵌容器 |
| ORM | MyBatis-Plus | 增强 CRUD、代码生成 |
| 数据库 | MySQL 8.x | InnoDB 引擎 |
| 缓存 | Redis 7.x | 分布式缓存、Token 存储 |
| 认证 | Spring Security + JWT / Redis Token | 双模式可切换 |
| API 文档 | Knife4j (Swagger) | OpenAPI 3.0 |
| 对象映射 | MapStruct | 编译期生成，禁止 BeanUtils |

## 目录结构

```
src/main/java/com/youlai/boot/
├── YouLaiBootApplication.java      # 启动类
├── common/                         # 公共模块（常量/枚举/工具/基类）
├── framework/                      # 框架层（不依赖业务模块）
│   ├── security/                   # 安全内核（通用，Port 接口解耦）
│   │   ├── port/                   # UserAuthenticationPort, PermissionPort
│   │   ├── service/                # SecurityUserDetailsService, PermissionService
│   │   └── token/                  # TokenManager, JwtTokenManager, RedisTokenManager
│   ├── mybatis/                    # MybatisConfig, MyMetaObjectHandler
│   ├── cache/                      # RedisConfig
│   └── web/                        # Web 基础设施
│       ├── advice/                 # GlobalExceptionHandler
│       ├── filter/                 # 请求过滤器
│       ├── log/                    # OperationLogPort, OperationLogAspect, OperationLog
│       ├── ratelimit/              # 限流脚本
│       └── util/                   # Web 工具
├── auth/                           # 认证模块（登录、Token 发放）
│   ├── controller/                 # AuthController
│   └── security/                   # 认证流程实现（Provider/Filter/Handler）
├── system/                         # 系统管理模块（用户/角色/菜单/部门/字典）
│   ├── controller/                 # UserController 等
│   ├── converter/                  # MapStruct Converter
│   ├── mapper/                     # MyBatis-Plus Mapper
│   ├── model/                      # entity/ form/ query/ vo/ dto/
│   ├── service/                    # Service + impl/
│   └── adapter/                    # 实现 framework 层 Port
│       ├── log/                    # OperationLogAdapter
│       └── security/               # UserAuthenticationAdapter, PermissionAdapter
├── codegen/                        # 代码生成模块
├── file/                           # 文件管理模块
└── message/                        # 消息推送模块（SSE）
```

设计原则：`common/` 被所有层共享；`framework/` 不依赖业务模块，通过 Port 接口解耦；`auth/` 含认证流程实现；`system/` 通过 Adapter 实现 Port。

## 命名规范

### 类命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 实体 | `Sys` 前缀 + PascalCase | `SysUser`, `SysRole` |
| DTO | 功能 + 类型后缀 | `UserVO`, `UserForm`, `UserPageQuery` |
| Service | 实体 + Service | `UserService` |
| Controller | 实体 + Controller | `UserController` |
| Mapper | 实体 + Mapper | `UserMapper` |
| Converter | 实体 + Converter | `UserConverter` |
| 异常 | 名词短语描述状态 + Exception | `MobileNotBoundException` |

### DTO 后缀

| 后缀 | 用途 | 示例 |
|------|------|------|
| `VO` | 视图对象（返回前端） | `UserVO` |
| `Form` | 表单对象（新增/更新） | `UserForm` |
| `Query` | 查询参数 | `UserPageQuery` |
| `DTO` | 传输对象（内部/服务间） | `RolePermsDTO` |

### 方法命名

| 动作 | 前缀 | 示例 |
|------|------|------|
| 查询单个 | getXxx | `getUserById()` |
| 查询列表 | listXxx | `listUsers()` |
| 分页查询 | getXxxPage | `getUserPage()` |
| 新增 | saveXxx | `saveUser()` |
| 更新 | updateXxx | `updateUser()` |
| 删除 | removeXxx | `removeByIds()` |
| 判断存在 | existsXxx | `existsByUsername()` |
| 下拉选项 | listXxxOptions | `listRoleOptions()` |

### import 排序

```java
// 1. java.* / jakarta.* 标准库
// 2. org.springframework.* Spring 框架
// 3. 第三方库（cn.hutool, com.baomidou, io.swagger 等）
// 4. 项目内部（com.youlai.boot.*，按 common → framework → 业务模块排序）
```

## RESTful API 规范

### 路径命名

- 资源路径：`/api/v1/{资源复数}`，如 `/api/v1/users`
- 子资源：`/{id}/menus`、`/{id}/form`、`/{id}/status`

### 标准 CRUD

| 操作 | 方法 | 路径 |
|------|------|------|
| 分页列表 | GET | `/api/v1/roles` |
| 表单详情 | GET | `/api/v1/roles/{id}/form` |
| 新增 | POST | `/api/v1/roles` |
| 更新 | PUT | `/api/v1/roles/{id}` |
| 删除（批量） | DELETE | `/api/v1/roles/{ids}` |
| 下拉选项 | GET | `/api/v1/roles/options` |

### Controller 模板

```java
@Tag(name = "03.角色接口")
@RestController
@RequestMapping("/api/v1/roles")
@RequiredArgsConstructor
public class RoleController {

    private final RoleService roleService;

    @Operation(summary = "角色分页列表")
    @GetMapping
    @Log(module = LogModuleEnum.ROLE, value = ActionTypeEnum.LIST)
    public PageResult<RolePageVO> getRolePage(RoleQuery queryParams) {
        Page<RolePageVO> result = roleService.getRolePage(queryParams);
        return PageResult.success(result);
    }

    @Operation(summary = "新增角色")
    @PostMapping
    @PreAuthorize("@ss.hasPerm('sys:role:create')")
    @RepeatSubmit
    @Log(module = LogModuleEnum.ROLE, value = ActionTypeEnum.INSERT)
    public Result<?> addRole(@Valid @RequestBody RoleForm roleForm) {
        return Result.judge(roleService.saveRole(roleForm));
    }

    @Operation(summary = "删除角色")
    @DeleteMapping("/{ids}")
    @PreAuthorize("@ss.hasPerm('sys:role:delete')")
    @Log(module = LogModuleEnum.ROLE, value = ActionTypeEnum.DELETE)
    public Result<Void> deleteRoles(@PathVariable String ids) {
        roleService.deleteRoles(ids);
        return Result.success();
    }
}
```

### Controller 注解

| 注解 | 用途 |
|------|------|
| `@Tag(name = "03.角色接口")` | Swagger 分组（带序号） |
| `@Operation(summary = "...")` | 接口摘要 |
| `@PreAuthorize("@ss.hasPerm('xxx')")` | 权限校验 |
| `@RepeatSubmit` | 防重复提交（新增/更新） |
| `@Log(module = ..., value = ...)` | 操作日志（增删改） |
| `@Valid @RequestBody` | 请求体校验 |

## 响应格式与异常处理

### 统一响应 Result

```json
{ "code": "00000", "msg": "成功", "data": { "id": 1, "username": "admin" } }
```

```java
@Data
public class Result<T> implements Serializable {
    private String code;  // String，5 位
    private String msg;
    private T data;

    public static <T> Result<T> success(T data) { ... }
    public static <T> Result<T> failed(String msg) { ... }
    public static <T> Result<T> judge(boolean status) { return status ? success() : failed(); }
}
```

### 分页响应 PageResult

```json
{ "code": "00000", "msg": "成功", "data": { "list": [ ... ], "total": 100 } }
```

分页接口直接返回 `PageResult`，非分页接口返回 `Result`，两者并列。

### 结果码

遵循阿里巴巴错误码：`00000` 成功，`A****` 用户端错误，`B****` 系统端错误，`C****` 第三方服务错误。

```java
public enum ResultCode implements IResultCode {
    SUCCESS("00000", "成功"),
    ACCESS_TOKEN_INVALID("A0230", "访问令牌无效或已过期"),
    REFRESH_TOKEN_INVALID("A0231", "刷新令牌无效或已过期"),
    ACCESS_PERMISSION_EXCEPTION("A0300", "访问权限异常"),
    SYSTEM_ERROR("B0001", "系统执行出错"),
    DATABASE_ACCESS_DENIED("C0351", "演示环境已禁用数据库写入功能");
}
```

### 业务异常

```java
throw new BusinessException(ResultCode.USER_PASSWORD_ERROR);
throw new BusinessException(ResultCode.USER_PASSWORD_ERROR, "密码错误，剩余 2 次");
```

### 全局异常处理

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(BusinessException.class)
    public Result<Void> handleBusinessException(BusinessException e) {
        return Result.failed(e.getCode(), e.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public Result<Void> handleException(Exception e) {
        log.error("系统异常", e);
        return Result.failed(ResultCode.SYSTEM_ERROR);
    }
}
```

业务码使用字符串 `[A-C][0-9]{4}` 而非 HTTP 状态码。前端 axios 拦截器根据 `code` 精确判断（`A0230` 触发 Token 刷新，`A0231` 跳转登录页）。

## 实体规范

```java
@Data
public class BaseEntity {
    @TableId(type = IdType.ASSIGN_ID)
    private Long id;

    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;

    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;

    @TableLogic
    private Integer deleted;
}

@Data
@EqualsAndHashCode(callSuper = true)
@TableName("sys_user")
@Schema(description = "用户实体")
public class SysUser extends BaseEntity {
    @Schema(description = "用户名")
    private String username;

    @Schema(description = "状态(1正常 0禁用)")
    private Integer status;
}
```

## Service 规范

```java
public interface UserService {
    PageResult<UserVO> pageUsers(UserPageQuery query);
    UserForm getUserFormData(Long id);
    Long saveUser(UserForm form);
    void removeUsersByIds(List<Long> ids);
}

@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {

    private final SysUserMapper userMapper;
    private final UserConverter userConverter;

    @Override
    public PageResult<UserVO> getUserPage(UserPageQuery query) {
        Page<SysUser> page = new Page<>(query.getPageNum(), query.getPageSize());
        LambdaQueryWrapper<SysUser> wrapper = Wrappers.lambdaQuery();
        wrapper.like(StrUtil.isNotBlank(query.getKeywords()), SysUser::getUsername, query.getKeywords())
               .eq(query.getStatus() != null, SysUser::getStatus, query.getStatus())
               .orderByDesc(SysUser::getCreateTime);
        Page<SysUser> result = userMapper.selectPage(page, wrapper);
        return PageResult.success(result.convert(userConverter::toVO));
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public Long saveUser(UserForm form) {
        Assert.isTrue(!existsByUsername(form.getUsername()), "用户名已存在");
        SysUser user = userConverter.toEntity(form);
        user.setPassword(BCryptUtils.encode(form.getPassword()));
        userMapper.insert(user);
        return user.getId();
    }
}
```

## 反模式速查

| 反模式 | 正确做法 |
|--------|----------|
| `BeanUtils.copyProperties` | MapStruct |
| `@Transactional` 无 `rollbackFor` | 加 `rollbackFor = Exception.class` |
| `this.method()` 调用事务方法 | 注入自身代理 |
| `log.debug("x" + obj)` | 用占位符 `{}` |
| 公共方法用 `//` 注释 | 用 `/** */` Javadoc |
| `NeedBindMobileException` | `MobileNotBoundException`（名词短语） |
| `MyAuthenticationEntryPoint` | `JsonAuthenticationEntryPoint`（体现特征） |
| 业务认证组件放 `framework/security` | 放 `auth/security` |

## 自查清单

- [ ] 遵循 RESTful API 路径规范（`/api/v1/{资源复数}`）
- [ ] 统一响应格式（`Result` code 为 String 类型）
- [ ] 分页接口返回 `PageResult`
- [ ] 实体继承 `BaseEntity`，使用 `@TableLogic` 逻辑删除
- [ ] Service 接口与实现分离
- [ ] 使用 MapStruct Converter（禁止 `BeanUtils`）
- [ ] 权限注解用 `@PreAuthorize("@ss.hasPerm('xxx')")`
- [ ] 增删改接口添加 `@Log` + `@RepeatSubmit`
- [ ] 写操作加 `@Transactional(rollbackFor = Exception.class)`
- [ ] 只读查询加 `@Transactional(readOnly = true)`
- [ ] 公共类/方法用 Javadoc `/** */`，类有 `@author` + `@since`，首句 summary 用英文句号 `.` 收尾（不能用中文 `。`）
- [ ] 日志用 `@Slf4j` + 占位符 `{}`，敏感信息脱敏

## 参考文档

| 参考文件 | 适用场景 |
|----------|----------|
| [references/authentication.md](references/authentication.md) | 认证架构（Ports & Adapters）、Token 模式、权限校验 |
| [references/transaction.md](references/transaction.md) | 事务规范、失效场景、事务边界 |
| [references/mapstruct.md](references/mapstruct.md) | MapStruct 对象转换、标准模板 |
| [references/comments.md](references/comments.md) | Javadoc 注释规范、标签参考、模板 |
| [references/logging.md](references/logging.md) | 日志级别、打印规范、内容要求 |
| [references/new-module.md](references/new-module.md) | 添加新模块完整教程（Entity → Mapper → DTO → Converter → Service → Controller） |
