---
name: spring-boot
description: Spring Boot backend development standards. Use this skill when developing Spring Boot projects, implementing REST APIs, MyBatis-Plus data access, authentication, or JWT/Redis token management.
---

# Spring Boot 后端开发规范

## 触发条件

- Develop Spring Boot projects
- Implement REST APIs
- Use MyBatis-Plus for data access
- Implement JWT/Redis Token authentication
- Implement permission control

---

## Part 1: 技术栈

| 层 | 选型 | 说明 |
|----|------|------|
| JDK | **Java 17+** | 运行环境 |
| 基础框架 | **Spring Boot 3.x** | 自动配置、内嵌容器 |
| ORM | **MyBatis-Plus** | 增强 CRUD、代码生成 |
| 数据库 | **MySQL 8.x** | InnoDB 引擎 |
| 缓存 | **Redis 7.x** | 分布式缓存、Token 存储 |
| 认证 | **Spring Security** + **JWT** / **Redis Token** | 双模式可切换 |
| API 文档 | **Knife4j** (Swagger) | OpenAPI 3.0 |
| 对象映射 | **MapStruct** | 编译期生成，禁止 BeanUtils |
| 工具库 | **Hutool** / **Lombok** | 减少样板代码 |

---

## Part 2: 目录结构

```
src/main/java/com/youlai/boot/
├── YouLaiBootApplication.java      # 启动类（@EnableScheduling）
│
├── common/                         # 公共模块（被所有层共享）
│   ├── annotation/                 # 自定义注解（DataPermission, Log, RepeatSubmit, ValidField）
│   ├── aspect/                     # 切面（LogAspect, RepeatSubmitAspect）
│   ├── base/                       # 基类（BaseEntity, BaseQuery, IBaseEnum）
│   ├── constant/                   # 常量（JwtClaimConstants, RedisConstants, SecurityConstants, SystemConstants）
│   ├── enums/                      # 通用枚举（ActionTypeEnum, DataScopeEnum, LogModuleEnum, StatusEnum）
│   ├── exception/                  # BusinessException
│   ├── model/                      # 通用模型（KeyValue, Option）
│   ├── result/                     # Result, PageResult, ResultCode, ResponseWriter
│   ├── util/                       # ExcelUtils, IPUtils
│   └── validator/                  # @ValidField 校验器
│
├── framework/                      # 框架层（技术基础设施，不依赖业务模块）
│   ├── apidoc/                     # OpenApiConfig, Knife4jOpenApiCustomizer
│   ├── cache/                      # RedisConfig, CaffeineConfig
│   ├── captcha/                    # 验证码（config, service, model, exception）
│   ├── integration/                # 第三方集成（wxma, mail, sms）
│   ├── job/                        # XxlJobConfig
│   ├── mybatis/                    # MybatisConfig, MyMetaObjectHandler, MyDataPermissionHandler
│   ├── security/                   # 安全内核（通用，不依赖业务）
│   │   ├── config/                 # SecurityProperties
│   │   ├── filter/                 # TokenAuthenticationFilter
│   │   ├── model/                  # SecurityUser, SecurityUserDetails, RoleDataScope, AuthenticationToken
│   │   ├── port/                   # UserAuthenticationPort, PermissionPort（端口接口）
│   │   ├── service/                # SecurityUserDetailsService, PermissionService
│   │   ├── token/                  # TokenManager, JwtTokenManager, RedisTokenManager
│   │   └── util/                   # SecurityUtils
│   └── web/                        # GlobalExceptionHandler, CorsConfig, JacksonConfig, RateLimiterFilter
│
├── auth/                           # 认证模块（登录、Token 发放）
│   ├── controller/                 # AuthController, WxMaAuthController
│   ├── service/                    # AuthService, WxMaAuthService
│   ├── model/                      # LoginReq, WxMaBindMobileReq, WxMaLoginResp
│   └── security/                   # 认证流程实现（归使用方）
│       ├── config/                 # SecurityConfig（Provider/Filter/Handler 总装）
│       ├── provider/               # SmsAuthenticationProvider, WxMaAuthenticationProvider
│       ├── model/                  # SmsAuthenticationToken, WxMaAuthenticationToken
│       ├── filter/                 # CaptchaValidationFilter（登录前置校验）
│       ├── handler/                # JsonAuthenticationEntryPoint, JsonAccessDeniedHandler
│       └── exception/              # MobileNotBoundException, SmsCaptchaException
│
├── system/                         # 系统管理模块（用户/角色/菜单/部门/字典 CRUD）
│   ├── controller/                 # UserController, RoleController, MenuController, DeptController 等
│   ├── converter/                  # UserConverter 等（MapStruct）
│   ├── enums/                      # MenuTypeEnum, DictCodeEnum 等
│   ├── mapper/                     # UserMapper 等
│   ├── model/                      # entity/ form/ query/ vo/ dto/
│   ├── service/                    # UserService 等 + impl/
│   └── security/
│       └── adapter/                # UserAuthenticationAdapter（实现 framework 端口，委托 system 服务）
│
├── codegen/                        # 代码生成模块
├── file/                           # 文件管理模块
└── message/                        # 消息推送模块（SSE）
```

**设计原则**：
- `common/` 是公共常量/枚举/工具，被所有层共享，不依赖任何层
- `framework/` 是框架层，不依赖 `auth/` 和 `system/`，通过 Port 接口解耦
- `auth/` 是认证模块，依赖 `framework/`，含认证流程实现（Provider/Filter/Handler）
- `system/` 是系统管理，依赖 `framework/`，通过 Adapter 实现 Port
- `framework.security` 只含通用安全组件；业务特定认证（短信/微信）归 `auth.security`

---

## Part 3: 命名规范

### 3.1 文件命名

| 类型 | 规范 | 示例 |
|------|------|------|
| Java 文件 | PascalCase，类名与文件名一致 | `UserController.java` |
| 配置文件 | 小写 + `-` 分隔 | `application-dev.yml` |
| XML 映射 | 实体 + Mapper.xml | `UserMapper.xml` |
| SQL 脚本 | 小写 + `_` 分隔 | `schema_init.sql` |

### 3.2 类命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 实体 | `Sys` 前缀 + PascalCase | `SysUser`, `SysRole`, `SysMenu` |
| DTO | 功能 + 类型后缀 | `UserVO`, `UserForm`, `UserPageQuery` |
| Service | 实体 + Service | `UserService` |
| ServiceImpl | 实体 + ServiceImpl | `UserServiceImpl` |
| Controller | 实体 + Controller | `UserController` |
| Mapper | 实体 + Mapper | `UserMapper` |
| Converter | 实体 + Converter | `UserConverter` |
| Config | 功能 + Config | `RedisConfig` |
| Enum | 功能 + Enum | `DataScopeEnum` |
| 异常 | 名词短语描述异常状态 + Exception | `MobileNotBoundException` |

**DTO 后缀规范**：

| 后缀 | 用途 | 示例 |
|------|------|------|
| `VO` | 视图对象（返回前端） | `UserVO`, `RolePageVO` |
| `Form` | 表单对象（新增/更新） | `UserForm`, `RoleForm` |
| `Query` | 查询参数 | `UserQuery`, `RolePageQuery` |
| `DTO` | 传输对象（内部/服务间） | `RolePermsDTO` |

> ⚠️ **异常命名**：用名词短语描述异常状态，禁止中文直译动词句式。
> - ❌ `NeedBindMobileException`（"需要绑定手机号"直译）
> - ✅ `MobileNotBoundException`（描述"手机号未绑定"状态）

### 3.3 方法命名

| 动作 | 前缀 | 示例 |
|------|------|------|
| 查询单个 | get | `getById()` |
| 查询列表 | list | `listUsers()` |
| 分页查询 | page / getPxxPage | `getUserPage()` |
| 新增 | save / add | `saveUser()` |
| 更新 | update | `updateUser()` |
| 删除 | remove / delete | `removeById()` |
| 统计 | count | `countUsers()` |
| 判断存在 | exists | `existsByUsername()` |
| 下拉选项 | listXxxOptions | `listRoleOptions()` |

### 3.4 变量命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 成员变量 | camelCase | `userService` |
| 常量 | UPPER_SNAKE_CASE | `MAX_PAGE_SIZE` |
| 布尔值 | is/has/can 前缀 | `isDeleted`, `hasPermission` |
| 私有方法 | 无特殊前缀 | `existsByUsername()` |

### 3.5 import 排序

```java
// 1. java.* / jakarta.* 标准库
import java.util.List;
import jakarta.validation.Valid;

// 2. org.springframework.* Spring 框架
import org.springframework.web.bind.annotation.*;

// 3. 第三方库（cn.hutool, com.baomidou, io.swagger 等）
import cn.hutool.core.util.StrUtil;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;

// 4. 项目内部（com.youlai.boot.*，按 common → framework → 业务模块排序）
import com.youlai.boot.common.result.Result;
import com.youlai.boot.framework.security.util.SecurityUtils;
import com.youlai.boot.system.model.entity.SysUser;
import com.youlai.boot.system.service.UserService;
```

---

## Part 4: RESTful API 规范

### 4.1 路径命名

- 资源路径：`/api/v1/{资源复数}`，如 `/api/v1/users`、`/api/v1/roles`
- 版本前缀：`/api/v1/`
- 路径参数：`/{id}`、`/{ids}`（批量删除逗号分隔）
- 子资源：`/{id}/menus`、`/{id}/form`、`/{id}/status`

### 4.2 标准 CRUD 路径

| 操作 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 分页列表 | `GET` | `/api/v1/roles` | Query 参数传递查询条件 |
| 详情/表单 | `GET` | `/api/v1/roles/{id}/form` | 表单回显数据 |
| 新增 | `POST` | `/api/v1/roles` | Body: `RoleForm` |
| 更新 | `PUT` | `/api/v1/roles/{id}` | Body: `RoleForm` |
| 删除（批量） | `DELETE` | `/api/v1/roles/{ids}` | 逗号分隔：`1,2,3` |
| 修改状态 | `PUT` | `/api/v1/roles/{id}/status` | Query 参数 `status` |
| 下拉选项 | `GET` | `/api/v1/roles/options` | 返回 `List<Option<Long>>` |
| 分配菜单 | `PUT` | `/api/v1/roles/{id}/menus` | Body: `List<Long>` |

### 4.3 Controller 模板

```java
/**
 * 角色控制层。
 *
 * @author Ray.Hao
 * @since 2022/10/16
 */
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
        boolean result = roleService.saveRole(roleForm);
        return Result.judge(result);
    }

    @Operation(summary = "获取角色表单数据")
    @GetMapping("/{roleId}/form")
    @PreAuthorize("@ss.hasPerm('sys:role:update')")
    public Result<RoleForm> getRoleForm(@PathVariable Long roleId) {
        return Result.success(roleService.getRoleForm(roleId));
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

### 4.4 Controller 注解规范

| 注解 | 用途 | 示例 |
|------|------|------|
| `@Tag(name = "03.角色接口")` | Swagger 分组（带序号） | 类级别 |
| `@Operation(summary = "...")` | 接口摘要 | 方法级别 |
| `@PreAuthorize("@ss.hasPerm('sys:role:create')")` | 权限校验 | 使用 `@ss.hasPerm` |
| `@RepeatSubmit` | 防重复提交 | 新增/更新接口 |
| `@Log(module = ..., value = ...)` | 操作日志 | 增删改接口 |
| `@Valid @RequestBody` | 请求体校验 | POST/PUT Body |

---

## Part 5: 响应格式与异常处理

### 5.1 统一响应 `Result<T>`

```json
{ "code": "00000", "msg": "成功", "data": { "id": 1, "username": "admin" } }
```

```java
@Data
public class Result<T> implements Serializable {
    /** 业务状态码（String，5 位：00000 成功 / A0230 令牌失效 / B0001 系统错误） */
    private String code;
    private String msg;
    private T data;

    public static <T> Result<T> success() { return success(null); }
    public static <T> Result<T> success(T data) { ... }
    public static <T> Result<T> failed() { ... }
    public static <T> Result<T> failed(String msg) { ... }
    public static <T> Result<T> failed(IResultCode resultCode) { ... }
    public static <T> Result<T> judge(boolean status) { return status ? success() : failed(); }
}
```

### 5.2 分页响应 `PageResult<T>`

```json
{ "code": "00000", "msg": "成功", "data": { "list": [ ... ], "total": 100 } }
```

```java
@Data
public class PageResult<T> implements Serializable {
    private String code;
    private String msg;
    private PageData<T> data;

    public static <T> PageResult<T> success(IPage<T> page) { ... }
    public static <T> PageResult<T> success(List<T> list) { ... }

    @Data
    public static class PageData<T> {
        private List<T> list;
        private long total;
    }
}
```

> **关键**：分页接口直接返回 `PageResult`，非分页接口返回 `Result`，两者并列。

### 5.3 结果码 `ResultCode`

遵循阿里巴巴错误码建议：`00000` 成功，`A****` 用户端错误，`B****` 系统端错误，`C****` 第三方服务错误。

```java
public enum ResultCode implements IResultCode {
    SUCCESS("00000", "成功"),

    // A 类：用户端错误
    USER_LOGIN_EXCEPTION("A0200", "用户登录异常"),
    USER_PASSWORD_ERROR("A0210", "用户名或密码错误"),
    ACCESS_TOKEN_INVALID("A0230", "访问令牌无效或已过期"),
    REFRESH_TOKEN_INVALID("A0231", "刷新令牌无效或已过期"),
    USER_VERIFICATION_CODE_ERROR("A0240", "验证码错误"),
    ACCESS_PERMISSION_EXCEPTION("A0300", "访问权限异常"),
    USER_REQUEST_PARAMETER_ERROR("A0400", "用户请求参数错误"),
    DUPLICATE_SUBMISSION("A0506", "请勿重复提交"),

    // B 类：系统端错误
    SYSTEM_ERROR("B0001", "系统执行出错"),

    // C 类：第三方服务错误
    DATABASE_SERVICE_ERROR("C0300", "数据库服务出错"),
    DATABASE_ACCESS_DENIED("C0351", "演示环境已禁用数据库写入功能");

    private final String code;
    private final String msg;
}
```

### 5.4 业务异常 `BusinessException`

```java
@Getter
public class BusinessException extends RuntimeException {
    private final String code;  // String 类型

    public BusinessException(String message) { ... }
    public BusinessException(String code, String message) { ... }
    public BusinessException(IResultCode resultCode) { ... }
}
```

```java
// 使用示例
throw new BusinessException(ResultCode.ACCOUNT_NOT_FOUND);
throw new BusinessException(ResultCode.USER_PASSWORD_ERROR, "密码错误，剩余 2 次");
return Result.judge(userService.saveRole(form));
```

### 5.5 全局异常处理

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(BusinessException.class)
    public Result<Void> handleBusinessException(BusinessException e) {
        return Result.failed(e.getCode(), e.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Result<Void> handleValidationException(MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldErrors().stream()
            .map(FieldError::getDefaultMessage)
            .collect(Collectors.joining(", "));
        return Result.failed(ResultCode.USER_REQUEST_PARAMETER_ERROR.getCode(), message);
    }

    @ExceptionHandler(Exception.class)
    public Result<Void> handleException(Exception e) {
        log.error("系统异常", e);
        return Result.failed(ResultCode.SYSTEM_ERROR);
    }
}
```

> **设计原则**：业务码使用字符串 `[A-C][0-9]{4}` 而非 HTTP 状态码数字。前端 axios 拦截器根据 `code` 精确判断业务逻辑（`A0230` 触发 Token 刷新，`A0231` 跳转登录页）。

---

## Part 6: 实体规范

### 6.1 基础实体

```java
@Data
public class BaseEntity {
    @TableId(type = IdType.ASSIGN_ID)
    private Long id;

    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;

    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;

    @TableField(fill = FieldFill.INSERT)
    private Long createBy;

    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Long updateBy;

    @TableLogic
    private Integer deleted;
}
```

### 6.2 实体示例

```java
@Data
@EqualsAndHashCode(callSuper = true)
@TableName("sys_user")
@Schema(description = "用户实体")
public class SysUser extends BaseEntity {

    @Schema(description = "用户名")
    private String username;

    @Schema(description = "状态(1正常 0禁用)")
    private Integer status;

    @Schema(description = "部门ID")
    private Long deptId;
}
```

---

## Part 7: Service 规范

### 7.1 Service 接口

```java
public interface UserService {
    PageResult<UserVO> pageUsers(UserPageQuery query);
    UserVO getUserById(Long id);
    UserForm getUserFormData(Long id);
    Long saveUser(UserForm form);
    void updateUser(UserForm form);
    void removeUsersByIds(List<Long> ids);
    List<OptionType> listUserOptions();
}
```

### 7.2 Service 实现

```java
@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {

    private final SysUserMapper userMapper;
    private final UserConverter userConverter;

    @Override
    public PageResult<UserVO> pageUsers(UserPageQuery query) {
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
        userRoleMapper.insertBatch(user.getId(), form.getRoleIds());
        return user.getId();
    }

    private boolean existsByUsername(String username) {
        return userMapper.exists(Wrappers.lambdaQuery(SysUser.class)
            .eq(SysUser::getUsername, username));
    }
}
```

---

## Part 8: 认证规范

### 8.1 安全架构（Ports & Adapters）

```
framework.security (Starter 内核，通用)
  ├── port/          UserAuthenticationPort, PermissionPort（端口接口）
  ├── model/         SecurityUser, SecurityUserDetails
  ├── service/       SecurityUserDetailsService, PermissionService
  └── token/         TokenManager, JwtTokenManager, RedisTokenManager
        ↑ 端口调用
        |
auth.security (认证流程实现，归使用方)
  ├── config/        SecurityConfig（Provider/Filter/Handler 总装）
  ├── provider/      SmsAuthenticationProvider, WxMaAuthenticationProvider
  ├── filter/        CaptchaValidationFilter
  ├── handler/       JsonAuthenticationEntryPoint, JsonAccessDeniedHandler
  └── exception/     MobileNotBoundException, SmsCaptchaException
        |
        ↓ 数据查询委托
system.security.adapter
  └── UserAuthenticationAdapter（实现 Port，委托 system 服务）
```

### 8.2 Token 模式

```yaml
security:
  session:
    mode: jwt  # 或 redis-token
    jwt:
      secret: SecretKey012345678901234567890123456789
    access-token-ttl: 7200   # 2小时
    refresh-token-ttl: 604800 # 7天
```

### 8.3 权限校验

```java
// 按钮权限（推荐用法）
@PreAuthorize("@ss.hasPerm('sys:user:create')")

// 角色权限
@PreAuthorize("hasRole('ADMIN')")
```

> **关键**：使用 `@ss.hasPerm('xxx')` 而非原生 `hasAuthority('xxx')`。`@ss` 是 `PermissionService` 的 Bean 名称（`@Component("ss")`），内部通过 `PermissionPort` 查询权限集合做匹配。

### 8.4 权限标识命名

```
模块:实体:操作
sys:user:create
sys:user:edit
sys:role:assign
```

| 操作 | 说明 |
|------|------|
| `list` / `detail` | 列表 / 详情 |
| `create` / `edit` / `delete` | 增改删 |
| `import` / `export` | 导入导出 |
| `assign` | 分配权限 |

---

## Part 9: 事务规范

### 9.1 基本原则

```java
// ✅ 必须指定 rollbackFor = Exception.class
@Transactional(rollbackFor = Exception.class)

// ✅ 只读查询加 readOnly = true
@Transactional(readOnly = true)

// ❌ 错误：未指定 rollbackFor，受检异常不回滚
@Transactional
```

### 9.2 事务失效场景（高频踩坑）

| 场景 | 原因 | 解决方案 |
|------|------|----------|
| 类内部方法调用 | AOP 代理失效，`this.method()` 不走代理 | 注入自身代理：`self.method()` |
| 方法非 public | Spring AOP 只代理 public 方法 | 改为 public |
| 异常被 catch 吞掉 | 未抛出 | catch 块中 `throw new RuntimeException(e)` |
| rollbackFor 未指定 | 受检异常不回滚 | 指定 `rollbackFor = Exception.class` |

### 9.3 事务边界

- **Service 层加事务**，Controller 和 Mapper 层不加
- **只读查询**标记 `readOnly = true`
- **批量操作**分批提交（每 500 条），避免长事务锁表
- **远程调用（RPC/HTTP）不在事务内**，避免事务过长
- **异步操作**用 `@TransactionalEventListener` 事务提交后触发

---

## Part 10: MapStruct 对象转换规范

### 10.1 命名与目录

| 规范 | 说明 |
|------|------|
| 接口名 | `{Entity}Converter`，如 `UserConverter` |
| 存放位置 | `{module}/converter/` |
| 注解 | `@Mapper(componentModel = "spring")` |

### 10.2 标准模板

```java
@Mapper(componentModel = "spring")
public interface UserConverter {

    // Entity → VO
    UserVO toVO(SysUser entity);
    List<UserVO> toVOList(List<SysUser> entities);

    // Form → Entity
    SysUser toEntity(UserForm form);

    // Entity → Form（编辑回显）
    UserForm toForm(SysUser entity);

    // 字段名不同时用 @Mapping
    @Mapping(source = "dept.name", target = "deptName")
    UserVO toVOWithDept(SysUser entity);
}
```

### 10.3 MapStruct vs BeanUtils

| 方案 | 性能 | 类型安全 | 编译期检查 | 推荐度 |
|------|------|----------|-----------|:---:|
| **MapStruct** | 极高（编译期生成） | ✅ | ✅ | ⭐⭐⭐⭐⭐ |
| 手写 getter/setter | 极高 | ✅ | ✅ | ⭐⭐⭐ |
| BeanUtils.copyProperties | 低（反射） | ❌ | ❌ | ⭐ |

> **强制**：使用 MapStruct，禁止 `BeanUtils.copyProperties` 和 JSON 中转。

---

## Part 11: 注释规范

> **来源**：阿里巴巴 Java 开发手册 + Javadoc 最佳实践。

### 11.1 总则

- **Javadoc 优先**：公共类/方法必须用 `/** */` 格式，不用 `//`
- **自解释代码**：注释描述"为什么"和"是什么"，而非"如何做"
- **中文优先**：专有名词保留英文（`TCP`、`@Transactional`），解释用中文
- **同步更新**：代码修改时注释必须同步修改

### 11.2 强制规则

| 规则 | 说明 |
|------|------|
| 公共类/方法用 `/** */` | IDE 可识别为 Javadoc，生成 API 文档 |
| 类必须注明 `@author` + `@date` | 格式 `yyyy/MM/dd` |
| 公共方法注明 `@param` + `@return` | 含约束、格式、特殊值（null/空集合） |
| 抛异常的方法注明 `@throws` | 注明触发条件 |
| 枚举字段用 `/** */` 注释 | 每个枚举值说明含义 |

### 11.3 Javadoc 标签参考

| 标签 | 作用于 | 用法 | 必填 |
|------|--------|------|:---:|
| `@author` | 类 | 作者名 | ✅ |
| `@date` | 类 | 创建日期 `yyyy/MM/dd` | ✅ |
| `@since` | 类、方法 | 起始版本号 | 推荐 |
| `@param` | 方法 | 参数名 + 说明 | ✅ |
| `@return` | 方法 | 返回值含义 | ✅ |
| `@throws` | 方法 | 异常类 + 抛出条件 | 有异常时必填 |
| `@see` | 类、方法 | 关联引用 | 推荐 |
| `@deprecated` | 方法 | 废弃说明 + 替代方案 | 有 `@Deprecated` 时必填 |
| `{@code}` | 任意 | 内联代码片段 | — |
| `{@link}` | 任意 | 可点击跳转引用 | — |

### 11.4 注释模板

**Controller**：

```java
/**
 * 用户管理 REST 控制器。
 * <p>
 * 基础路径：{@code /api/v1/users}，除下拉选项外均需按钮权限。
 *
 * @author Ray.Hao
 * @date 2025/01/15
 * @since 3.0.0
 */
@RestController
@RequestMapping("/api/v1/users")
public class UserController {

    /**
     * 创建新用户。
     *
     * @param form 用户表单（username 必填且唯一）
     * @return 新用户 ID
     * @throws BusinessException 如果用户名已存在
     */
    @PostMapping
    public Result<Long> save(@Valid @RequestBody UserForm form) { ... }
}
```

**Service**：

```java
/**
 * 用户管理服务。
 * <p>
 * 职责：用户 CRUD、密码管理、角色分配。
 *
 * @author Ray.Hao
 * @see UserMapper
 * @see UserConverter
 */
public interface UserService { ... }
```

**Entity 字段**：

```java
/** 用户名（唯一，大小写敏感） */
private String username;

/**
 * 用户状态。
 * <p>
 * 取值：{@code 1} 启用，{@code 0} 禁用。禁用的用户无法登录。
 */
private Integer status;
```

### 11.5 TODO / FIXME 标记

| 标记 | 格式 | 示例 |
|------|------|------|
| `TODO` | `TODO(标记人, 日期, [说明])` | `TODO(zhangsan, 2025/06/01, 扩展邮箱验证码)` |
| `FIXME` | `FIXME(标记人, 日期, [说明])` | `FIXME(lisi, 2025/06/15, 并发偶发死锁)` |

### 11.6 常见反模式

| 反模式 | 正确做法 |
|--------|----------|
| 公共方法用 `//` 注释 | 用 `/** */` 格式 |
| `@param user 参数`（废话） | `@param user 用户实体（必填，密码需加密）` |
| `@return result`（废话） | `@return 登录结果（含 accessToken）` |
| 注释掉的代码无说明 | 加 `// 保留原因：xxx` 或直接删除 |

---

## Part 12: 日志规范

### 12.1 日志框架

- **门面**：SLF4J（注解 `@Slf4j` 替代 `LoggerFactory.getLogger()`）
- **实现**：Logback（Spring Boot 默认）

### 12.2 日志级别

| 级别 | 使用场景 | 示例 |
|------|----------|------|
| `ERROR` | 影响运行的错误，需人工介入 | 数据库连接失败、第三方 API 异常 |
| `WARN` | 非预期但可降级 | 缓存未命中降级、限流触发、登录失败 |
| `INFO` | 关键业务节点 | 用户登录、订单创建、定时任务开始/结束 |
| `DEBUG` | 调试信息（生产关闭） | SQL 语句、请求参数 |

### 12.3 打印规范

```java
// ❌ 错误：字符串拼接（即使日志级别关闭也会执行）
log.debug("用户信息: " + user.toString());

// ❌ 错误：用 System.out
System.out.println("用户创建成功");

// ✅ 正确：占位符 {}
log.info("用户创建成功: username={}, userId={}", form.getUsername(), user.getId());

// ✅ 正确：异常日志含堆栈（e 作为最后参数）
try {
    userMapper.insert(user);
} catch (Exception e) {
    log.error("保存用户失败: username={}", form.getUsername(), e);
    throw new BusinessException(ResultCode.SYSTEM_ERROR);
}

// ✅ 正确：敏感信息脱敏
log.info("用户登录: username={}, mobile={}", username, maskMobile(mobile));
// ❌ 禁止：日志输出密码、Token、身份证号明文
```

### 12.4 内容要求

| 要求 | 说明 |
|------|------|
| 携带业务标识 | `userId={}` `orderNo={}`，方便检索 |
| 异常含堆栈 | `log.error(msg, e)`，异常对象作为最后参数 |
| 禁止敏感信息 | 密码、Token、身份证号必须脱敏 |
| 条件日志 | 复杂计算用 `if (log.isDebugEnabled())` 包裹 |

---

## Part 13: 添加新模块 — 完整教程

以添加 **"通知公告"** 模块为例。

### Step 1: 创建实体

```java
// system/model/entity/SysNotice.java
@Data
@EqualsAndHashCode(callSuper = true)
@TableName("sys_notice")
public class SysNotice extends BaseEntity {
    private String title;
    private String content;
    private Integer type;    // 1:通知 2:公告
    private Integer status;  // 1:已发布 0:未发布
}
```

### Step 2: 创建 Mapper

```java
// system/mapper/SysNoticeMapper.java
@Mapper
public interface SysNoticeMapper extends BaseMapper<SysNotice> {
}
```

### Step 3: 创建 DTO

```java
// system/model/form/NoticeForm.java
@Data
public class NoticeForm {
    @NotBlank
    private String title;
    private String content;
    private Integer type;
}

// system/model/vo/NoticePageVO.java
@Data
public class NoticePageVO {
    private Long id;
    private String title;
    private Integer type;
    private Integer status;
    private LocalDateTime createTime;
}

// system/model/query/NoticePageQuery.java
@Data
@EqualsAndHashCode(callSuper = true)
public class NoticePageQuery extends BaseQuery {
    private String keywords;
    private Integer type;
}
```

### Step 4: 创建 Converter

```java
// system/converter/NoticeConverter.java
@Mapper(componentModel = "spring")
public interface NoticeConverter {
    NoticePageVO toVO(SysNotice entity);
    List<NoticePageVO> toVOList(List<SysNotice> entities);
    SysNotice toEntity(NoticeForm form);
}
```

### Step 5: 创建 Service

```java
// system/service/NoticeService.java
public interface NoticeService {
    PageResult<NoticePageVO> getNoticePage(NoticePageQuery query);
    boolean saveNotice(NoticeForm form);
    void deleteNotices(String ids);
}

// system/service/impl/NoticeServiceImpl.java
@Service
@RequiredArgsConstructor
public class NoticeServiceImpl implements NoticeService {
    private final SysNoticeMapper noticeMapper;
    private final NoticeConverter noticeConverter;

    @Override
    public PageResult<NoticePageVO> getNoticePage(NoticePageQuery query) {
        Page<SysNotice> page = new Page<>(query.getPageNum(), query.getPageSize());
        LambdaQueryWrapper<SysNotice> wrapper = Wrappers.lambdaQuery();
        wrapper.like(StrUtil.isNotBlank(query.getKeywords()), SysNotice::getTitle, query.getKeywords())
               .eq(query.getType() != null, SysNotice::getType, query.getType())
               .orderByDesc(SysNotice::getCreateTime);
        Page<SysNotice> result = noticeMapper.selectPage(page, wrapper);
        return PageResult.success(result.convert(noticeConverter::toVO));
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public boolean saveNotice(NoticeForm form) {
        SysNotice notice = noticeConverter.toEntity(form);
        return noticeMapper.insertOrUpdate(notice);
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void deleteNotices(String ids) {
        List<Long> idList = Arrays.stream(ids.split(","))
            .map(Long::parseLong).collect(Collectors.toList());
        noticeMapper.deleteBatchIds(idList);
    }
}
```

### Step 6: 创建 Controller

```java
// system/controller/NoticeController.java
@Tag(name = "08.通知公告")
@RestController
@RequestMapping("/api/v1/notices")
@RequiredArgsConstructor
public class NoticeController {

    private final NoticeService noticeService;

    @Operation(summary = "通知分页列表")
    @GetMapping
    @Log(module = LogModuleEnum.NOTICE, value = ActionTypeEnum.LIST)
    public PageResult<NoticePageVO> getNoticePage(NoticePageQuery query) {
        return noticeService.getNoticePage(query);
    }

    @Operation(summary = "新增通知")
    @PostMapping
    @PreAuthorize("@ss.hasPerm('sys:notice:create')")
    @RepeatSubmit
    @Log(module = LogModuleEnum.NOTICE, value = ActionTypeEnum.INSERT)
    public Result<?> addNotice(@Valid @RequestBody NoticeForm form) {
        return Result.judge(noticeService.saveNotice(form));
    }

    @Operation(summary = "删除通知")
    @DeleteMapping("/{ids}")
    @PreAuthorize("@ss.hasPerm('sys:notice:delete')")
    @Log(module = LogModuleEnum.NOTICE, value = ActionTypeEnum.DELETE)
    public Result<Void> deleteNotices(@PathVariable String ids) {
        noticeService.deleteNotices(ids);
        return Result.success();
    }
}
```

### 模块文件清单

```
system/
├── controller/NoticeController.java
├── converter/NoticeConverter.java
├── mapper/SysNoticeMapper.java
├── model/
│   ├── entity/SysNotice.java
│   ├── form/NoticeForm.java
│   ├── query/NoticePageQuery.java
│   └── vo/NoticePageVO.java
└── service/
    ├── NoticeService.java
    └── impl/NoticeServiceImpl.java
```

---

## Part 14: 代码质量检查清单

### 14.1 通用检查

- [ ] 遵循 RESTful API 路径规范（`/api/v1/{资源复数}`）
- [ ] 统一响应格式（`Result` code 为 String 类型）
- [ ] 分页接口返回 `PageResult.success(page)` 而非 `Result.success(...)`
- [ ] 实现全局异常处理（`@RestControllerAdvice`）
- [ ] 参数校验使用 `@Valid`
- [ ] 实体继承 `BaseEntity`
- [ ] 使用 MyBatis-Plus 逻辑删除（`@TableLogic`）
- [ ] Service 接口与实现分离
- [ ] 使用 MapStruct Converter（禁止 `BeanUtils.copyProperties`）
- [ ] API 添加 `@Operation` 注解

### 14.2 认证与权限

- [ ] 权限注解用 `@PreAuthorize("@ss.hasPerm('xxx')")` 而非 `hasAuthority`
- [ ] 增删改接口添加 `@Log` 操作日志
- [ ] 新增接口添加 `@RepeatSubmit` 防重复提交
- [ ] 业务特定认证组件（Provider/Filter/Handler）放 `auth/security/`，不放 `framework/security/`

### 14.3 事务

- [ ] 写操作加 `@Transactional(rollbackFor = Exception.class)`
- [ ] 只读查询加 `@Transactional(readOnly = true)`
- [ ] 类内部调用事务方法用注入自身代理
- [ ] 事务方法为 public
- [ ] 不在事务内做 RPC/HTTP 远程调用

### 14.4 注释

- [ ] 公共类/方法用 `/** */` Javadoc 格式
- [ ] 类有 `@author` 和 `@date`
- [ ] 公共方法有 `@param` + `@return` + `@throws`
- [ ] 枚举字段有 `/** */` 注释
- [ ] TODO/FIXME 含标记人、时间、说明

### 14.5 日志

- [ ] 使用 `@Slf4j` 替代 `LoggerFactory.getLogger()`
- [ ] 使用占位符 `{}` 而非字符串拼接
- [ ] 异常日志包含堆栈（`log.error(msg, e)`）
- [ ] 敏感信息已脱敏
- [ ] 关键业务节点打印 INFO 日志

### 14.6 常见反模式

| 反模式 | 问题 | 正确做法 |
|--------|------|----------|
| `BeanUtils.copyProperties` | 反射性能差、无编译期检查 | 用 MapStruct |
| `@Transactional` 无 `rollbackFor` | 受检异常不回滚 | 加 `rollbackFor = Exception.class` |
| `this.method()` 调用事务方法 | AOP 代理失效 | 注入自身代理 `self.method()` |
| `log.debug("x" + obj)` | 即使日志关闭也执行拼接 | 用占位符 `{}` |
| `System.out.println()` | 无日志级别、无时间戳 | 用 `log.info()` |
| 公共方法用 `//` 注释 | IDE 无法识别为 Javadoc | 用 `/** */` 格式 |
| `@param user 参数` | 废话 | `@param user 用户实体（必填）` |
| `NeedBindMobileException` | 中文直译动词句式 | `MobileNotBoundException`（名词短语描述状态） |
| `MyAuthenticationEntryPoint` | `My*` 无效前缀 | `JsonAuthenticationEntryPoint`（体现特征） |
| 业务认证组件放 `framework/security` | framework 不应含业务特定代码 | 放 `auth/security` |
