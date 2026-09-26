# 注释与 Javadoc

仅在新增公共契约、整理注释或审查文档质量时读取本文件。

## 核心目标

注释应减少理解和维护成本。优先解释代码无法自行表达的信息：业务约束、设计原因、副作用、线程或事务要求、输入格式、失败条件和兼容性决定。

代码变化时同步修改注释。过时或与实现冲突的注释必须修正或删除。

## 规则等级

### 必须

- 公共扩展点、跨模块接口、Port/Adapter 契约和存在非显然约束的方法使用 Javadoc。
- 参数存在格式、范围、单位、空值语义或安全约束时说明这些约束。
- 返回值可能为 `null`、空集合、部分结果或特殊状态时说明其语义。
- 方法会抛出调用方需要处理的异常时使用 `@throws` 说明触发条件。
- 方法内部注释放在相关语句上方，解释“为什么”，不复述下一行代码。
- 删除无用途的注释代码；确需暂留时写明原因和清理条件。

### 建议

- 面向调用方的公共类写一句清晰摘要，必要时补充职责和边界。
- 稳定公共 API 用 `@since` 标明引入版本，值取项目当前版本号，升版本时一起改；普通内部类不强制。
- API 字段使用 `@Schema` 说明协议含义，Javadoc 只补充领域约束，避免重复维护相同文案。
- 枚举值含义不直观时逐项说明；名称已经完整表达语义时无需制造重复注释。

### 可选项目约定

- `@author` 沿用项目既有的作者约定，不要自创值（具体写谁由项目定），也不要求每个类都加。
- 不强制中文 Javadoc 首句以英文句号 `.` 结尾。

## Summary 写法

如果项目确实发布 Javadoc 并依赖摘要索引，优先显式使用 `{@summary}`：

```java
/**
 * {@summary 创建新用户}
 * 校验用户名唯一性，并在同一事务内分配角色。
 */
```

普通中文注释可以使用自然中文标点。英文句号加空格只是传统 Doclet 识别首句的一种方式，不应升级成所有代码必须遵守的社区规范。

## 标签使用

- `@param`：说明约束和语义，不重复参数类型。
- `@return`：说明结果含义和特殊值；`void` 方法不写。
- `@throws`：说明异常类型及触发条件。
- `@since`：标明引入版本，取项目当前版本号（`pom.xml` 的 `version`），不写日期。
- `@deprecated`：与 `@Deprecated` 同时使用，并给出替代方案。
- `{@link}`：正文中的主要关联。
- `@see`：附加参考，不替代正文。

常用块标签顺序保持项目一致，例如 `@param` → `@return` → `@throws` → `@since` → `@see` → `@deprecated`；类级 `@author` 若保留，放在 `@since` 之前。

## 推荐示例

### Controller

Controller 已有 `@Operation` 时，方法 Javadoc 只补充 OpenAPI 注解未表达的重要行为：

```java
/**
 * {@summary 创建用户}
 * 用户名必须唯一；成功后返回新用户 ID。
 *
 * @param form 创建参数
 * @return 新用户 ID
 * @throws BusinessException 用户名已经存在
 */
@PostMapping
public Result<Long> create(@Valid @RequestBody UserCreateForm form) {
    return Result.success(userService.createUser(form));
}
```

如果 `@Operation` 和类型已经完整表达普通 CRUD 行为，可以不再写重复的方法 Javadoc。

### Service 或 Port

```java
/**
 * {@summary 创建用户并分配角色}
 * 实现必须保证用户和角色关系一致提交，密码只保存强哈希。
 *
 * @param form 创建参数，用户名和角色列表必填
 * @return 新用户 ID
 * @throws BusinessException 用户名已存在或角色不可用
 */
Long createUser(UserCreateForm form);
```

### 字段

```java
@Schema(description = "用户状态", allowableValues = {"0", "1"})
private Integer status;

/** 以秒为单位；-1 表示永不过期。 */
private Integer accessTokenTimeToLive;
```

第一项由 `@Schema` 足以表达，不必再复制一份 Javadoc；第二项包含非显然的单位和特殊值，值得保留。

### Port 与 Adapter

```java
/**
 * {@summary 用户认证查询端口}
 * framework 只依赖本接口，不依赖 system 的 Service 或 Entity。
 */
public interface UserAuthenticationPort {
    SecurityUser findByUsername(String username);
}

/** 实现 {@link UserAuthenticationPort}，将认证查询委托给 system 模块。 */
public class UserAuthenticationAdapter implements UserAuthenticationPort {
}
```

## 方法内部注释

```java
// 使用独立事务，避免审计写入失败回滚主业务。
auditService.recordInNewTransaction(event);

/*
 * Flowable 历史查询不保证业务展示顺序，必须在转换后按开始时间排序。
 */
history.sort(comparing(HistoryItem::getStartTime));
```

以下注释没有价值：

```java
// 查询用户
User user = userMapper.selectById(id);

// 返回成功
return Result.success();
```

## TODO 与 FIXME

标记必须包含可以执行的说明；项目需要责任人或期限时再加入，不强制一种固定日期格式：

```java
// TODO: 表单发布后改为通过领域事件刷新缓存。
// FIXME: 并发发布可能重复递增版本号，需要增加乐观锁。
```

能立即修复的问题不要留下 TODO。

## 审查清单

- 注释是否与当前包名、配置键、返回结构和异常类型一致？
- 是否解释了约束和原因，而不是翻译代码？
- `@param`、`@return`、`@throws` 是否只在有信息量时出现？
- `@since` 是否与项目当前版本号一致，`@author` 是否沿用既有约定而不是自创？
- `@Schema` 与 Javadoc 是否重复或冲突？
- 是否意外记录或示例化了真实密码、Token、密钥等敏感信息？
- 注释中提到的类型和链接是否真实存在？
