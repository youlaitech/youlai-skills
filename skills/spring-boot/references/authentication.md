# 认证规范

## 安全架构（Ports & Adapters）

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
system.adapter.security
  └── UserAuthenticationAdapter（实现 Port，委托 system 服务）
```

## Token 模式

```yaml
security:
  session:
    mode: jwt  # 或 redis-token
    jwt:
      secret: SecretKey012345678901234567890123456789
    access-token-ttl: 7200   # 2小时
    refresh-token-ttl: 604800 # 7天
```

## 权限校验

```java
// 按钮权限（推荐用法）
@PreAuthorize("@ss.hasPerm('sys:user:create')")

// 角色权限
@PreAuthorize("hasRole('ADMIN')")
```

使用 `@ss.hasPerm('xxx')` 而非原生 `hasAuthority('xxx')`。`@ss` 是 `PermissionService` 的 Bean 名称（`@Component("ss")`），内部通过 `PermissionPort` 查询权限集合做匹配。

## 权限标识命名

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
