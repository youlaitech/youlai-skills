# 注释规范

来源：阿里巴巴 Java 开发手册（八）注释规约 + Oracle Javadoc 官方规范。

## 总则

- Javadoc 优先：公共类/属性/方法用 `/** */`，不用 `//`
- 注释写约束和踩坑点，不写代码已经能看出来的
- 中文优先：专有名词保留英文（`TCP`、`@Transactional`），解释用中文
- 代码改了注释必须同步改，过时注释比没注释更坑
- 避免冗余：类名后缀（Controller/Service/Mapper）和注解（`@RestController`）已表达的信息，不在首句重复

## 强制规则

- 公共类/属性/方法必须用 `/** */`，不得用 `// xxx`
- 所有类必须注明 `@author` + `@since`（`@since` 为起始版本号）
- 公共方法必须注明 `@param` + `@return`，含约束、格式、特殊值（`null`/空集合）
- 抛异常的方法必须注明 `@throws`，注明触发条件
- `@param`/`@return` 描述以小写字母开头、短语形式、不加句号；不重复参数名/类型，不以 `is` 开头，不加破折号前缀
- 抽象方法/接口方法必须说明「做什么、实现什么功能」，对子类的实现要求一并写明
- 枚举字段每个值必须用 `/** */` 注释说明用途
- 方法内部注释：单行在被注释语句上方另起一行用 `//`，多行用 `/* */`，与代码对齐
- 注释掉的代码必须在上方说明保留原因，无用则直接删除

## 首句 summary 必须用英文句号

Javadoc 工具用「dot-space 算法」截断首句作为 API 文档索引摘要：扫描注释，遇到**英文句号 `.` + 空白字符**即认为 summary 结束。中文句号 `。` 不被识别。

- ✅ `/** 创建新用户. */` —— summary 正确截断为「创建新用户」
- ❌ `/** 创建新用户。 */` —— 中文句号不被识别，整段描述都被当作 summary，索引混乱
- ❌ `/** 创建新用户 */` —— 无句号，javadoc 找不到边界，可能告警

详细描述部分可以用中文句号，但首句必须用英文句号收尾。

### 首句含英文缩写的坑

首句若含 `e.g.`、`Prof.`、`i.e.` 等含英文句号的缩写，会误触发截断。在句号后加 `&nbsp;` 可规避：

```java
/** 关闭 Prof.&nbsp; Knuth 提到的 MIX 计算机. */
```

### JDK 10+ 的彻底方案

用 `{@summary}` 标签显式指定摘要，完全规避句号问题：

```java
/** {@summary 创建新用户} 支持唯一性校验。详细描述可自由使用中文句号。 */
```

## @author + @since

类注释用 `@author` + `@since 版本号`，不用 `@date`。

- `@author` 写代码作者名（个人名或组织标识均可）
- 新增类默认沿用项目现有作者（如 `Ray.Hao`），也可使用开源项目组织标识 `youlai.tech`
- 历史代码的 `@author` 不批量改动

```java
/**
 * @author <author>
 * @since <version>
 */
```

## Javadoc 标签参考

| 标签 | 作用对象 | 用法 | 必填 |
|------|----------|------|:---:|
| `@author` | 类 | 作者名 | 是 |
| `@since` | 类、方法 | 起始版本号 | 是 |
| `@param` | 方法 | 参数名 + 说明（小写开头、不加句号） | 是 |
| `@return` | 方法 | 返回值含义（含特殊值） | 是 |
| `@throws` | 方法 | 异常类 + 抛出条件 | 有异常时必填 |
| `@deprecated` | 方法 | 废弃说明 + 替代方案 | 有 `@Deprecated` 时必填 |
| `{@code}` | 任意 | 内联代码片段 | — |
| `{@link}` | 任意 | 行内可点击引用，嵌在描述里 | 跨层接口互相定位 |
| `@see` | 类、方法 | 「另请参阅」独立块标签，用于关联但非主线的引用 | 按需 |

### 块标签顺序

按 `@param` → `@return` → `@throws` → `@since` → `@see`/`@deprecated` 排列，Checkstyle 的 `AtclauseOrder` 默认即此顺序。

### `@see` 与 `{@link}` 的区别

二者不是简单替代关系：

- `@see` 是 **block tag**，独立成段，用于「另请参阅」列表
- `{@link}` 是 **inline tag**，嵌在描述文本里，生成可点击超链接

```java
/**
 * 用户认证端口.
 * <p>由 {@link com.youlai.boot.system.adapter.security.UserAuthenticationAdapter} 实现</p>
 *
 * @see PermissionPort 权限校验端口
 * @author <author>
 * @since 4.7.0
 */
public interface UserAuthenticationPort { ... }
```

## 注释模板

### Controller

```java
/**
 * 用户管理控制器.
 * <p>
 * 基础路径：{@code /api/v1/users}，除下拉选项外均需按钮权限。
 *
 * @author <author>
 * @since 3.0.0
 */
@RestController
@RequestMapping("/api/v1/users")
public class UserController {

    /**
     * 创建新用户.
     *
     * @param form 用户表单（username 必填且唯一）
     * @return 新用户 ID
     * @throws BusinessException 如果用户名已存在
     */
    @PostMapping
    public Result<Long> save(@Valid @RequestBody UserForm form) { ... }
}
```

### Service 接口（抽象方法必须说明「做什么 + 实现要求」）

```java
/**
 * 用户管理服务.
 * <p>
 * 职责：用户 CRUD、密码管理、角色分配。
 *
 * @author <author>
 * @since 3.0.0
 */
public interface UserService {

    /**
     * 保存用户并分配角色.
     * <p>
     * 实现要求：必须校验用户名唯一性，密码使用 BCrypt 加密，角色分配需事务一致。
     *
     * @param form 用户表单（username 必填且唯一）
     * @return 新用户 ID
     * @throws BusinessException 如果用户名已存在
     */
    Long saveUser(UserForm form);
}
```

### 类属性/字段（不限于 Entity）

```java
/** 用户名（唯一，大小写敏感） */
private String username;

/**
 * 用户状态.
 * <p>
 * 取值：{@code 1} 启用，{@code 0} 禁用。禁用的用户无法登录。
 */
private Integer status;
```

### 跨层接口（Port / Adapter）

用 `{@link}` 嵌在描述里互相导航，便于阅读时直接点击跳转（`@see` 留给「关联但非主线」的引用）：

```java
/**
 * 操作日志落库端口.
 * <p>由 {@link com.youlai.boot.system.adapter.log.OperationLogAdapter} 实现</p>
 *
 * @author <author>
 * @since 4.7.0
 */
public interface OperationLogPort { ... }

/**
 * 操作日志适配器.
 * <p>实现 {@link OperationLogPort}，落库 sys_log</p>
 *
 * @author <author>
 * @since 4.7.0
 */
public class OperationLogAdapter implements OperationLogPort { ... }
```

## 方法内部注释

- 单行：在被注释语句**上方另起一行**，用 `//`
- 多行：用 `/* */`，与代码对齐
- 写「为什么」和「踩坑点」，不写代码已表达的内容

```java
public void transfer(Long from, Long to, BigDecimal amount) {
    // 踩坑：此处不能用 this.debit()，会绕过事务代理
    self.debit(from, amount);

    /*
     * 跨行注释示例：
     * 大额转账需异步通知风控，避免阻塞主流程
     */
    if (amount.compareTo(THRESHOLD) > 0) {
        asyncRiskNotify(from, to, amount);
    }
}
```

## TODO / FIXME 标记

| 标记 | 格式 | 示例 |
|------|------|------|
| `TODO` | `TODO(标记人, 日期, [说明])` | `TODO(zhangsan, 2025/06/01, 扩展邮箱验证码)` |
| `FIXME` | `FIXME(标记人, 日期, [说明])` | `FIXME(lisi, 2025/06/15, 并发偶发死锁)` |

线上故障常来源于此类标记处的代码，需定期扫描清理。

## 常见反模式

| 反模式 | 正确做法 |
|--------|----------|
| 公共方法用 `//` 注释 | 用 `/** */` 格式 |
| 首句用中文句号 `。` 或无句号 | 首句用英文句号 `.` 收尾 |
| `@param user 参数`（废话） | `@param user 用户实体（必填，密码需加密）` |
| `@return result`（废话） | `@return 登录结果（含 accessToken）` |
| Service 接口只写方法名不写实现要求 | 写明「做什么 + 对子类的实现要求」 |
| 接口方法注释里漏 `@since` | 类和接口方法都补 `@since` |
| 注释掉的代码无说明 | 上方加 `// 保留原因：xxx` 或直接删除 |
