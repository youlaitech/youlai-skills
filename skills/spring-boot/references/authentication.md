# 认证与权限

仅在修改 Spring Security、登录、Token、白名单或权限时读取本文件。

## 依赖方向

```text
framework.security
  ├── port/       UserAuthenticationPort, PermissionPort
  ├── model/      SecurityUser, SecurityUserDetails
  ├── service/    SecurityUserDetailsService, PermissionService
  └── token/      TokenManager, JwtTokenManager, RedisTokenManager
        ↑
        │ Port 调用
        │
system.adapter.security
  └── UserAuthenticationAdapter, PermissionAdapter

auth
  ├── controller/
  ├── service/
  ├── security/filter/
  ├── security/provider/
  ├── security/handler/
  └── exception/

config/SecurityConfig   # 组合根，装配上述组件
```

- `framework.security` 不依赖 `auth` 或 `system` 的实现类。
- `system.adapter.security` 实现 framework 定义的 Port。
- `SecurityConfig` 负责装配，可以依赖各模块 Bean，但不承载登录业务规则。
- 认证请求与响应遵循项目统一模型语言：输入使用 `Form`，输出使用 `VO`；内部会话状态放 `context` 或 `session`。

## Token 配置

属性名以 `SecurityProperties` 为准：

```yaml
security:
  session:
    type: jwt # jwt 或 redis-token
    access-token-time-to-live: 7200
    refresh-token-time-to-live: 604800
    jwt:
      secret-key: ${JWT_SECRET_KEY}
    redis-token:
      allow-multi-login: true
```

- 不在仓库中提交真实 JWT 密钥、数据库密码、Redis 密码或对象存储密钥。
- 示例配置只放环境变量占位符；已经提交过的真实密钥必须轮换，删除文件并不能使历史密钥恢复安全。
- 修改 TTL 时同时检查刷新、注销、多端登录和存量 Token 的兼容行为。

## 白名单语义

`ignoreUrls` 与 `unsecuredUrls` 不能混用：

| 配置 | 语义 |
|------|------|
| `ignore-urls` | `permitAll`，请求仍经过 Spring Security 过滤器链 |
| `unsecured-urls` | `web.ignoring()`，完全绕过 Spring Security 过滤器链 |

业务匿名接口通常使用 `ignore-urls`，这样仍可保留 CORS、CSRF 策略及其他安全过滤器。只把静态文档等无需安全链处理的端点放入 `unsecured-urls`。

## 权限校验

```java
@PreAuthorize("@ss.hasPerm('sys:user:create')")
@PreAuthorize("@ss.hasPerm('sys:user:update')")
@PreAuthorize("@ss.hasPerm('sys:user:delete')")
```

权限标识统一为：

```text
模块:资源:操作
sys:user:list
sys:user:detail
sys:user:create
sys:user:update
sys:user:delete
sys:user:import
sys:user:export
sys:role:assign
```

- 同一项目不要混用 `edit` 与 `update`；youlai-boot 使用 `update`。
- Controller 做访问授权，Service 仍需校验资源所有权、数据范围和状态转换等业务授权。
- 新增匿名端点时同时检查 URL 匹配范围、限流、防重复提交、敏感返回字段和审计日志。

## 认证数据安全

- 密码只以强哈希保存，不写入日志、不进入 VO。
- Token、验证码、openid、手机号和邮箱按敏感等级脱敏或避免记录。
- 登录失败不要泄露“用户存在但密码错误”等可用于枚举账户的信息，除非产品明确接受该风险。
- 认证异常使用 401，权限不足使用 403，限流使用 429；响应体可继续携带项目业务码。
