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

- **JDK** - Java 运行环境
- **Spring Boot** - 基础框架
- **MyBatis-Plus** - ORM 框架
- **MySQL** - 数据库
- **Redis** - 缓存
- **JWT** - Token 认证
- **Knife4j** - API 文档

---

## Part 2: 目录结构

```
src/main/java/com/youlai/boot/
├── YouLaiBootApplication.java      # 启动类（@EnableScheduling）
│
├── common/                         # 公共模块
│   ├── annotation/                 # 自定义注解
│   │   ├── DataPermission.java     # 数据权限
│   │   ├── Log.java                # 操作日志
│   │   ├── RepeatSubmit.java       # 防重复提交
│   │   └── ValidField.java         # 字段校验
│   ├── aspect/                     # 切面
│   │   ├── LogAspect.java
│   │   └── RepeatSubmitAspect.java
│   ├── base/                       # 基础类
│   │   ├── BaseEntity.java         # 实体基类（id, createTime, updateTime）
│   │   ├── BaseQuery.java          # 查询基类（pageNum, pageSize, sortBy, order）
│   │   └── IBaseEnum.java          # 枚举基接口
│   ├── constant/                   # 常量
│   │   ├── JwtClaimConstants.java
│   │   ├── RedisConstants.java
│   │   ├── SecurityConstants.java
│   │   └── SystemConstants.java
│   ├── enums/                      # 通用枚举
│   │   ├── ActionTypeEnum.java
│   │   ├── CaptchaTypeEnum.java
│   │   ├── DataScopeEnum.java
│   │   ├── EnvEnum.java
│   │   ├── LogModuleEnum.java
│   │   └── StatusEnum.java
│   ├── exception/                  # 异常
│   │   └── BusinessException.java
│   ├── model/                      # 通用模型
│   │   ├── KeyValue.java
│   │   └── Option.java            # 下拉选项
│   ├── result/                     # 统一响应
│   │   ├── IResultCode.java        # 结果码接口
│   │   ├── ResultCode.java         # 结果码枚举
│   │   ├── Result.java             # 统一响应体（code: String）
│   │   ├── PageResult.java         # 分页结果
│   │   ├── ExcelResult.java        # Excel 导出结果
│   │   └── ResponseWriter.java     # 响应写入工具
│   ├── util/                       # 工具类
│   │   ├── ExcelUtils.java
│   │   └── IPUtils.java
│   └── validator/                  # 自定义校验
│       └── FieldValidator.java     # @ValidField 校验器
│
├── framework/                      # 框架层（技术基础设施）
│   ├── apidoc/                     # API 文档
│   │   ├── OpenApiConfig.java
│   │   └── Knife4jOpenApiCustomizer.java
│   ├── cache/                      # 缓存配置
│   │   ├── RedisConfig.java
│   │   ├── RedisCacheConfig.java
│   │   └── CaffeineConfig.java
│   ├── captcha/                    # 验证码
│   │   ├── config/                 # CaptchaConfig, CaptchaProperties
│   │   ├── service/CaptchaService.java
│   │   ├── model/CaptchaInfo.java
│   │   └── exception/CaptchaException.java
│   ├── integration/                # 第三方集成
│   │   ├── wxma/                   # 微信小程序
│   │   ├── mail/                   # 邮件服务
│   │   └── sms/                    # 短信服务
│   ├── job/                        # 定时任务
│   │   └── XxlJobConfig.java
│   ├── mybatis/                    # MyBatis 配置
│   │   ├── config/MybatisConfig.java
│   │   ├── handler/MyMetaObjectHandler.java
│   │   └── interceptor/MyDataPermissionHandler.java
│   ├── security/                   # 安全框架
│   │   ├── config/                 # SecurityConfig, SecurityProperties, PasswordEncoderConfig
│   │   ├── filter/                 # TokenAuthenticationFilter, CaptchaValidationFilter
│   │   ├── handler/                # MyAccessDeniedHandler, MyAuthenticationEntryPoint
│   │   ├── model/                  # SysUserDetails, AuthenticationToken, RoleDataScope 等
│   │   ├── provider/               # SmsAuthenticationProvider, WxMaAuthenticationProvider
│   │   ├── service/                # PermissionService, SysUserDetailsService
│   │   ├── token/                  # TokenManager, JwtTokenManager, RedisTokenManager
│   │   └── util/SecurityUtils.java
│   └── web/                        # Web 配置
│       ├── advice/GlobalExceptionHandler.java
│       ├── config/                 # CorsConfig, JacksonConfig
│       └── filter/                 # RateLimiterFilter, RequestLogFilter
│
├── system/                         # 系统管理模块
│   ├── controller/                 # 控制器
│   │   ├── UserController.java
│   │   ├── RoleController.java
│   │   ├── MenuController.java
│   │   ├── DeptController.java
│   │   ├── DictController.java
│   │   ├── ConfigController.java
│   │   ├── NoticeController.java
│   │   └── LogController.java
│   ├── converter/                  # MapStruct 对象转换
│   │   └── UserConverter.java 等
│   ├── enums/                      # 模块枚举
│   │   ├── MenuTypeEnum.java
│   │   ├── DictCodeEnum.java
│   │   └── NoticePublishStatusEnum.java 等
│   ├── mapper/                     # MyBatis-Plus Mapper
│   │   └── UserMapper.java 等
│   ├── model/                      # 模型（按类型分子目录）
│   │   ├── entity/                 # 实体（SysUser, SysRole, SysMenu 等）
│   │   ├── form/                   # 表单 DTO（UserForm, RoleForm 等）
│   │   ├── query/                  # 查询 DTO（UserQuery, RoleQuery 等）
│   │   ├── vo/                     # 视图对象（UserPageVO, RolePageVO 等）
│   │   └── dto/                    # 传输对象（RolePermsDTO, VisitCountDTO）
│   ├── service/                    # 服务接口
│   │   ├── UserService.java
│   │   └── impl/UserServiceImpl.java
│   ├── handler/                    # 处理器
│   │   └── XxlJobSampleHandler.java
│   └── listener/                   # 事件监听
│       └── UserImportListener.java
│
├── auth/                           # 认证模块
│   ├── controller/
│   │   ├── AuthController.java
│   │   └── WxMaAuthController.java
│   ├── service/
│   │   ├── AuthService.java
│   │   ├── WxMaAuthService.java
│   │   └── impl/                   # 实现类
│   └── model/
│       ├── LoginReq.java
│       ├── WxMaBindMobileReq.java
│       ├── WxMaLoginResp.java
│       └── WxMaPhoneLoginReq.java
│
├── codegen/                        # 代码生成模块
│   ├── controller/CodegenController.java
│   ├── service/                    # CodegenService, GenTableService
│   ├── converter/CodegenConverter.java
│   ├── mapper/                     # DatabaseMapper, GenTableMapper
│   ├── model/
│   │   ├── entity/                 # GenTable, GenTableColumn
│   │   ├── form/GenConfigForm.java
│   │   ├── query/                  # TablePageQuery, TableQuery
│   │   └── vo/                     # CodegenPreviewVO 等
│   ├── enums/                      # FormTypeEnum, JavaTypeEnum, QueryTypeEnum
│   └── config/CodegenProperties.java
│
├── file/                           # 文件管理模块
│   ├── controller/FileController.java
│   ├── service/
│   │   ├── FileService.java
│   │   └── impl/                   # LocalFileService, AliyunFileService, MinioFileService
│   └── model/FileInfo.java
│
└── message/                        # 消息推送模块
    ├── controller/SseController.java
    ├── service/SseService.java
    ├── registry/SseSessionRegistry.java
    ├── dto/                        # DictChangeEvent, OnlineUserDTO
    ├── job/OnlineUserCountJob.java
    └── topic/SseTopics.java
```

---

## Part 3: 命名规范

### 类命名

| 类型        | 规范                       | 示例                                  |
| ----------- | -------------------------- | ------------------------------------- |
| 实体        | 实体名 + Entity 或前缀 Sys | `User` 或 `SysUser`                   |
| DTO         | 功能 + DTO 类型            | `UserVO`, `UserForm`, `UserPageQuery` |
| Service     | 实体 + Service             | `UserService`                         |
| ServiceImpl | 实体 + ServiceImpl         | `UserServiceImpl`                     |
| Controller  | 实体 + Controller          | `UserController`                      |
| Mapper      | 实体 + Mapper              | `UserMapper`                          |
| Converter   | 实体 + Converter           | `UserConverter`                       |

### 方法命名

| 动作     | 前缀          | 示例                 |
| -------- | ------------- | -------------------- |
| 查询单个 | get           | `getById()`          |
| 查询列表 | list          | `listUsers()`        |
| 分页查询 | page          | `pageUsers()`        |
| 新增     | save/add      | `saveUser()`         |
| 更新     | update        | `updateUser()`       |
| 删除     | remove/delete | `removeById()`       |
| 统计     | count         | `countUsers()`       |
| 判断存在 | exists        | `existsByUsername()` |

### 变量命名

| 类型     | 规范             | 示例                         |
| -------- | ---------------- | ---------------------------- |
| 成员变量 | camelCase        | `userService`                |
| 常量     | UPPER_SNAKE_CASE | `MAX_SIZE`                   |
| 私有变量 | \_ 前缀或无前缀  | `_internalState`             |
| 布尔值   | is/has/can 前缀  | `isDeleted`, `hasPermission` |

---

## Part 4: RESTful API 规范

### 路径命名

- 使用 kebab-case：`/api/v1/user-profile`
- 版本前缀：`/api/v1/`
- 资源命名：复数名词 `/users`, `/roles`

### 标准 CRUD 路径

| 操作     | 方法   | 路径                      |
| -------- | ------ | ------------------------- |
| 分页列表 | GET    | `/api/v1/users/page`      |
| 详情     | GET    | `/api/v1/users/{id}`      |
| 新增     | POST   | `/api/v1/users`           |
| 更新     | PUT    | `/api/v1/users`           |
| 删除     | DELETE | `/api/v1/users/{id}`      |
| 批量删除 | DELETE | `/api/v1/users/batch`     |
| 下拉选项 | GET    | `/api/v1/users/options`   |
| 表单数据 | GET    | `/api/v1/users/{id}/form` |

### Controller 模板

```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
@Tag(name = "用户管理")
public class UserController {

    private final UserService userService;

    @GetMapping("/page")
    @Operation(summary = "用户分页列表")
    public Result<PageResult<UserVO>> page(UserPageQuery query) {
        return Result.success(userService.pageUsers(query));
    }

    @GetMapping("/{id}")
    @Operation(summary = "用户详情")
    public Result<UserVO> getById(@PathVariable Long id) {
        return Result.success(userService.getUserById(id));
    }

    @GetMapping("/{id}/form")
    @Operation(summary = "用户表单数据")
    public Result<UserForm> getFormData(@PathVariable Long id) {
        return Result.success(userService.getUserFormData(id));
    }

    @PostMapping
    @Operation(summary = "新增用户")
    @PreAuthorize("hasAuthority('sys:user:add')")
    public Result<Long> save(@Valid @RequestBody UserForm form) {
        return Result.success(userService.saveUser(form));
    }

    @PutMapping
    @Operation(summary = "更新用户")
    @PreAuthorize("hasAuthority('sys:user:edit')")
    public Result<Void> update(@Valid @RequestBody UserForm form) {
        userService.updateUser(form);
        return Result.success();
    }

    @DeleteMapping("/{id}")
    @Operation(summary = "删除用户")
    @PreAuthorize("hasAuthority('sys:user:delete')")
    public Result<Void> remove(@PathVariable Long id) {
        userService.removeUserById(id);
        return Result.success();
    }

    @DeleteMapping("/batch")
    @Operation(summary = "批量删除用户")
    @PreAuthorize("hasAuthority('sys:user:delete')")
    public Result<Void> batchRemove(@RequestBody List<Long> ids) {
        userService.removeUsersByIds(ids);
        return Result.success();
    }

    @GetMapping("/options")
    @Operation(summary = "用户选项列表")
    public Result<List<OptionType>> options() {
        return Result.success(userService.listUserOptions());
    }
}
```

---

## Part 5: 响应格式

### 统一响应类

```java
@Data
public class Result<T> {
    private Integer code;
    private String msg;
    private T data;

    public static <T> Result<T> success(T data) {
        Result<T> result = new Result<>();
        result.setCode(ResultCode.SUCCESS.getCode());
        result.setMsg(ResultCode.SUCCESS.getMsg());
        result.setData(data);
        return result;
    }

    public static <T> Result<T> failed(ResultCode code) {
        Result<T> result = new Result<>();
        result.setCode(code.getCode());
        result.setMsg(code.getMsg());
        return result;
    }

    public static <T> Result<T> failed(Integer code, String msg) {
        Result<T> result = new Result<>();
        result.setCode(code);
        result.setMsg(msg);
        return result;
    }
}
```

### 分页响应类

```java
@Data
public class PageResult<T> {
    private List<T> list;
    private Long total;

    public static <T> PageResult<T> of(List<T> list, Long total) {
        PageResult<T> result = new PageResult<>();
        result.setList(list);
        result.setTotal(total);
        return result;
    }
}
```

### 响应码枚举

```java
@Getter
@AllArgsConstructor
public enum ResultCode {
    SUCCESS(200, "操作成功"),
    PARAM_ERROR(400, "参数错误"),
    UNAUTHORIZED(401, "未登录或token过期"),
    FORBIDDEN(403, "无权限"),
    NOT_FOUND(404, "资源不存在"),
    SYSTEM_ERROR(500, "系统异常");

    private final Integer code;
    private final String msg;
}
```

---

## Part 6: 实体规范

### 基础实体

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

### 实体示例

```java
@Data
@EqualsAndHashCode(callSuper = true)
@TableName("sys_user")
@Schema(description = "用户实体")
public class SysUser extends BaseEntity {

    @Schema(description = "用户名")
    private String username;

    @Schema(description = "昵称")
    private String nickname;

    @Schema(description = "密码")
    private String password;

    @Schema(description = "手机号")
    private String mobile;

    @Schema(description = "邮箱")
    private String email;

    @Schema(description = "性别(0未知 1男 2女)")
    private Integer gender;

    @Schema(description = "状态(1正常 0禁用)")
    private Integer status;

    @Schema(description = "部门ID")
    private Long deptId;
}
```

---

## Part 7: Service 规范

### Service 接口

```java
public interface UserService {
    PageResult<UserVO> pageUsers(UserPageQuery query);
    UserVO getUserById(Long id);
    UserForm getUserFormData(Long id);
    Long saveUser(UserForm form);
    void updateUser(UserForm form);
    void removeUserById(Long id);
    void removeUsersByIds(List<Long> ids);
    List<OptionType> listUserOptions();
}
```

### Service 实现

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
        List<UserVO> list = userConverter.toVOList(result.getRecords());
        return PageResult.of(list, result.getTotal());
    }

    @Override
    public UserVO getUserById(Long id) {
        SysUser user = userMapper.selectById(id);
        Assert.notNull(user, "用户不存在");
        return userConverter.toVO(user);
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public Long saveUser(UserForm form) {
        // 检查用户名是否存在
        Assert.isTrue(!existsByUsername(form.getUsername()), "用户名已存在");
        // 保存用户
        SysUser user = userConverter.toEntity(form);
        user.setPassword(BCryptUtils.encode(form.getPassword()));
        userMapper.insert(user);
        // 保存用户角色
        userRoleMapper.insertBatch(user.getId(), form.getRoleIds());
        return user.getId();
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void updateUser(UserForm form) {
        SysUser user = userConverter.toEntity(form);
        userMapper.updateById(user);
        // 更新用户角色
        userRoleMapper.deleteByUserId(user.getId());
        userRoleMapper.insertBatch(user.getId(), form.getRoleIds());
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void removeUserById(Long id) {
        userMapper.deleteById(id);
        userRoleMapper.deleteByUserId(id);
    }

    private boolean existsByUsername(String username) {
        return userMapper.exists(Wrappers.lambdaQuery(SysUser.class)
            .eq(SysUser::getUsername, username));
    }
}
```

---

## Part 8: 认证规范

### JWT 配置

```yaml
security:
  session:
    mode: jwt # 或 redis-token
    jwt:
      secret: SecretKey012345678901234567890123456789
      issuer: youlai-boot
    access-token-ttl: 7200 # 2小时
    refresh-token-ttl: 604800 # 7天
```

### Token 生成

```java
@Component
@RequiredArgsConstructor
public class JwtTokenManager {

    @Value("${security.session.jwt.secret}")
    private String secret;

    @Value("${security.session.jwt.issuer}")
    private String issuer;

    public String generateAccessToken(SysUserDetails userDetails) {
        return Jwts.builder()
            .subject(String.valueOf(userDetails.getUserId()))
            .claim("username", userDetails.getUsername())
            .issuer(issuer)
            .issuedAt(new Date())
            .expiration(new Date(System.currentTimeMillis() + accessTokenTtl * 1000))
            .signWith(Keys.hmacShaKeyFor(secret.getBytes()))
            .compact();
    }

    public SysUserDetails parseAccessToken(String token) {
        Claims claims = Jwts.parser()
            .verifyWith(Keys.hmacShaKeyFor(secret.getBytes()))
            .build()
            .parseSignedClaims(token)
            .getPayload();
        return SysUserDetails.builder()
            .userId(Long.parseLong(claims.getSubject()))
            .username(claims.get("username", String.class))
            .build();
    }
}
```

### 权限注解

```java
// 按钮权限
@PreAuthorize("hasAuthority('sys:user:add')")

// 角色权限
@PreAuthorize("hasRole('ADMIN')")

// 数据权限
@DataScope(deptAlias = "d", userAlias = "u")
```

---

## Part 9: 异常处理

### 全局异常处理

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
        return Result.failed(ResultCode.PARAM_ERROR.getCode(), message);
    }

    @ExceptionHandler(Exception.class)
    public Result<Void> handleException(Exception e) {
        log.error("系统异常", e);
        return Result.failed(ResultCode.SYSTEM_ERROR);
    }
}
```

### 业务异常

```java
@Getter
public class BusinessException extends RuntimeException {
    private final Integer code;

    public BusinessException(String message) {
        super(message);
        this.code = ResultCode.PARAM_ERROR.getCode();
    }

    public BusinessException(Integer code, String message) {
        super(message);
        this.code = code;
    }

    public BusinessException(ResultCode code) {
        super(code.getMsg());
        this.code = code.getCode();
    }
}
```

---

## Part 10: 代码质量检查清单

- [ ] 遵循 RESTful API 路径规范
- [ ] 使用统一响应格式
- [ ] 实现全局异常处理
- [ ] 添加权限注解
- [ ] 分页接口使用 PageResult
- [ ] 参数校验使用 @Valid
- [ ] 重要操作记录日志
- [ ] 使用 @Transactional 处理事务
- [ ] 实体继承 BaseEntity
- [ ] 使用 MyBatis-Plus 逻辑删除
- [ ] API 添加 @Operation 注解
- [ ] Service 接口与实现分离
